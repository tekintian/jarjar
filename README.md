# jarjar工具

jarjar工具也是一个很方便的打包java代码的工具，可以使用ant任务的方式，也可以单独在命令行下使用。 项目主页在：https://github.com/tekintian/jarjar

为什么要使用jarjar
------

- 有没有碰到这么一种情况，在开发项目的时候，为了避免“JAR hell”，为了更好的管理项目依赖，需要将别的java项目的jar包重新打包到我们自己现有的项目中来，比如把包org.apache.commons.logging打包成repackaged.org.apache.commons，你会怎么办？手工改显然费时费力，不仅要改变每个类的package声明，还要改变自己项目中所有使用了commons-logging类的代码。那么jarjar就是为了干这件事而专门定做的工具。

- 代码重新打包工具jarjar可以帮助你将其它用到的java库打包并嵌入到你自己的项目jar包中。这样做的原因有:

- 当你发布项目的时候，把用到的库打包进现有项目jar包，可以让发布的这个jar包不比依赖于其它项目的jar包;
- 当你所用到的java库升级了以后，它所新发布的jar包可能和你现存的项目不匹配，为了保持项目的代码稳定性，你可以把编写代码时所用到的依赖jar包，全部打包进现在的项目jar包，以避免出现这个问题。

	jarjar可以通过Ant任务的方式使用，也可以单独地在命令行下使用。打包代码时，如果你要重命名某些依赖包的名字的时候，jarjar会调用字节码转换(通过ASM)来更新代码，并自动做好其他工作。


以Ant任务的形式使用jarjar
------

我们现存的Ant任务里可以用jar任务来打包代码，比如:
```xml
<target name="jar" depends="compile">
	<jar jarfile="dist/example.jar">
		<fileset dir="build/main"/>
	</jar>
</target>
```

为了使用jarjar工具，我们创建一个叫jarjar的任务，由于JarJarTask是Ant标准任务Jar的子类，所以如果你不需要使用jarjar的特有功能的话，完全可以像这样调用jarjar工具:
```xml
<target name="jar" depends="compile">
	<taskdef name="jarjar" classname="com.tonicsystems.jarjar.JarJarTask" classpath="lib/jarjar.jar"/>
	<jarjar jarfile="dist/example.jar">
		<fileset dir="build/main"/>
	</jarjar>
</target>
```
就像标准的”jar”任务一样，可以通过”zipfileset”元素来包含其它jar包。但是仅仅包含其它jar包并不能让你远离“jar包陷阱”,因为你所依赖的jar包中的类名还是没有改变，仍然有可能和其它版本的jar包里的类名相同，产生冲突。
为了重命名类名，JarJarTask引入了一个新元素”rule”。”rule”包含了”pattern”属性，你可以通过这个属性，使用通配符来选择哪些类需要重命名，通过”result”属性可以设置如何给选中的类重命名。

在本例中我们希望引入一个叫jaxen.jar的库。并将所有以”org.jaxen”开头的类重命名以”org.example.jaxen”开头:
```xml
<target name="jar" depends="compile">
<taskdef name="jarjar" classname="com.tonicsystems.jarjar.JarJarTask" classpath="lib/jarjar.jar"/>
<jarjar jarfile="dist/example.jar">
<fileset dir="build/main"/>
<zipfileset src="lib/jaxen.jar"/>
<rule pattern="org.jaxen.**" result="org.example.@1"/>
</jarjar>
</target>
```
通配符**表示匹配循环所有的子包，如果你只希望匹配一个子包的话，可以使用*。
@1表示第一个**所匹配到的内容，一次类推，@2表示从左到右第二个所匹配到的*或**。@0是特殊的标志，它代表整个匹配到的类的全名。




命令行下单独使用jarjar
-----

java -jar jarjar.jar [help]

打印帮助信息。

java -jar jarjar.jar strings 
打印类路径classpath下的字符串信息，如果类中有debug信息的话，会打印出所在行的行号。
比如java -jar jarjar.jar strings servlet-api.jar会打印:

```text
javax.servlet.http.HttpServletRequest "BASIC" "FORM" "CLIENT_CERT" "DIGEST" 
javax.servlet.http.HttpUtils "javax.servlet.http.LocalStrings" 88: "javax.servlet.http.LocalStrings" 339: "://" 341: "http" 341: "https" 145: "&" 238: "err.io.short_read" 254: "8859_1"
```



java -jar jarjar.jar find []

打印出类路径下java类对类路径下类的依赖，如果省略了，那么用代替。只能取class或者jar，前者代表打印各个类之间的依赖情况，后者会打印包对包之间的依赖。

java -jar jarjar.jar process 

将按照文件所指定的方法转换到里，中原有的类将被删除。


- 类路径Classpath的格式
类路径classpath是用逗号或分号(具体是那种分隔符依赖操作系统)隔开的一组目录，jar包或者zip包。

- Rules规则文件格式
Rules规则文件是实际上一种文本文件，每一行代表一条规则Rule，行首和行末的空格会被忽略掉，有三种不同样式的Rule写法:
```xml
rule <pattern> <result>

zap <pattern>

keep <pattern>

```
第一个是用来设置jarjar如何重命名类文件的。所有类，只要它引用到了需要改变名字的类，其相关内容就会被自动同步改变，保证不会出现引用错误。如果一个类匹配了不同的rule，只有第一个匹配的rule会生效。 和的设定同上面讲过的Ant中一样。
zap规则中 所匹配的类将会不加入生成的新jar包。


English description
-------

Jar Jar Links is a utility that makes it easy to repackage Java
libraries and embed them into your own distribution. This is useful
for two reasons:

You can easily ship a single jar file with no external dependencies.

You can avoid problems where your library depends on a specific
version of a library, which may conflict with the dependencies of
another library.

Respository
https://mvnrepository.com/artifact/org.anarres.jarjar/jarjar-gradle


How does it work?

Jar Jar Links includes an Ant task that extends the built-in jar
task. The normal zipfileset element is used to embed jar files. A
new rule element is added which uses wildcards patterns to rename
the embedded class files. Bytecode transformation (via ASM) is used
to change references to the renamed classes, and special handling is
provided for moving resource files and transforming string literals.

Using with ant
--------------

In our imaginary project, the Ant "jar" target looks like:

```
<target name="jar" depends="compile">
    <jar jarfile="dist/example.jar">
        <fileset dir="build/main"/>
    </jar>
</target>
```

To use Jar Jar Links, we define a new task named "jarjar", and
substitute it wherever we used the jar task. Because the JarJarTask
class extends the normal Ant Jar task, you can use jarjar without
any of its additional features, if you want:

```
<target name="jar" depends="compile">
    <taskdef name="jarjar" classname="com.tonicsystems.jarjar.JarJarTask"
        classpath="lib/jarjar.jar"/>
    <jarjar jarfile="dist/example.jar">
        <fileset dir="build/main"/>
    </jarjar>
</target>
```

Just like with the "jar" task, we can include the contents of another
jar file using the "zipfileset" element. But simply including another
projects classes is not good enough to avoid jar hell, since the class
names remain unchanged and can still conflict with other versions.

To rename the classes, JarJarTask adds a new "rule" element. The
rule takes a "pattern" attribute, which uses wildcards to match
against class names, and a "result" attribute, which describes how
to transform the matched names.

In this example we include classes from jaxen.jar and add a rule
that changes any class name starting with "org.jaxen" to start with
"org.example.jaxen" instead (in our imaginary world we control the
example.org domain):

```
<target name="jar" depends="compile">
    <taskdef name="jarjar" classname="com.tonicsystems.jarjar.JarJarTask"
        classpath="lib/jarjar.jar"/>
    <jarjar jarfile="dist/example.jar">
        <fileset dir="build/main"/>
        <zipfileset src="lib/jaxen.jar"/>
        <rule pattern="org.jaxen.**" result="org.example.@1"/>
    </jarjar>
</target>
```

The ** in the pattern means to match against any valid package
substring. To match against a single package component (by excluding
dots (.) from the match), a single * may be used instead.

The @1 in the result is a reference to the ** portion of the rule. For
every * or ** in the rule, a numbered reference is available for use
in the result. References are numbered from left to right, starting
with @1, then @2, and so on.

The special @0 reference refers to the entire class name.

Using with gradle
-----------------

```
	dependencies {
		// Use jarjar.repackage in place of a dependency notation.
		compile jarjar.repackage {
			from 'com.google.guava:guava:18.0'

			classDelete "com.google.common.base.**"

			classRename "com.google.**" "org.private.google.@1"
		}
	}
```

See (jarjar-gradle/example/build.gradle) for some complete examples.

Using from the command line
---------------------------

From the command-line

```
java -jar jarjar.jar [help]
```

Prints this help message.

```
java -jar jarjar.jar strings <cp>
```

Dumps all string literals in classpath `<cp>`. Line numbers will be
included if the classes have debug information.

```
java -jar jarjar.jar find <level> <cp1> [<cp2>]
```

Prints dependencies on classpath `<cp2>` in classpath `<cp1>`. If `<cp2>`
is omitted, `<cp1>` is used for both arguments.

The level argument must be class or jar. The former prints dependencies
between individual classes, while the latter only prints jar->jar
dependencies. A "jar" in this context is actually any classpath
component, which can be a jar file, a zip file, or a parent directory
(see below).

```
java -jar jarjar.jar process <rulesFile> <inJar> <outJar>
```

Transform the `<inJar>` jar file, writing a new jar file to `<outJar>`. Any
existing file named by `<outJar>` will be deleted.

The transformation is defined by a set of rules in the file specified
by the rules argument (see below).  Classpath format

The classpath argument is a colon or semi-colon delimited
set (depending on platform) of directories, jar files,
or zip files. See the following page for more details:
http://java.sun.com/j2se/1.5.0/docs/tooldocs/solaris/classpath.html

Mustang-style wildcards are also supported:
http://bugs.sun.com/bugdatabase/view_bug.do?bug_id=6268383 Rules
file format

The rules file is a text file, one rule per line. Leading and trailing
whitespace is ignored. There are three types of rules:

```
rule <pattern> <result>
zap <pattern>
keep <pattern>
```

The standard rule (rule) is used to rename classes. All references to
the renamed classes will also be updated. If a class name is matched
by more than one rule, only the first one will apply.

`<pattern>` is a class name with optional wildcards. `**` will match
against any valid class name substring. To match a single package
component (by excluding . from the match), a single `*` may be used
instead.

`<result>` is a class name which can optionally reference the substrings
matched by the wildcards. A numbered reference is available for every
`*` or `**` in the `<pattern>`, starting from left to right: @1, @2, etc. A
special @0 reference contains the entire matched class name.

The zap rule causes any matched class to be removed from the resulting
jar file. All zap rules are processed before renaming rules.

The keep rule marks all matched classes as "roots". If any keep rules
are defined all classes which are not reachable from the roots via
dependency analysis are discarded when writing the output jar. This
is the last step in the process, after renaming and zapping.


Other resources
------
## application_plugin for gradle:
http://www.gradle.org/docs/current/userguide/application_plugin.html
just add this lines to build.gradle:

apply plugin: ‘application’
mainClassName = “com.mkyong.DateUtils”

and then run gradle task:

$ gradle distZip

It’s better, because:
1. This is plugin and your build.gradle much cleaner.
2. This distribute only runtime deps, not compile and test.
3. This will be maintained by plugin contributers.

