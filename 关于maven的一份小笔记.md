# 简介

项目里一直用的 maven，几乎天天和这个“熟知”的工具打交道，但是，最近我发觉自己对 maven 了解的还不够，例如，什么是 goal？什么是 phase？等等。趁着最近有时间，把[官网文档](https://maven.apache.org/)大致看了一遍，并且做做笔记，也就形成了这篇博客。

本文主要讲解以下内容：

1. 什么是 maven？maven有什么用？
2. 安装和使用 maven
3. maven 的构建生命周期
4. 配置 maven
5. 常见问题（持续更新）

# 什么是maven？maven 有什么用？

这两个问题，很多文章都有说到，但是，大部分都是翻译了官网的这句笼统的话，看了和没看一样。

```
Apache Maven is a software project management and comprehension tool. Based on the concept of a project object model (POM), Maven can manage a project's build, reporting and documentation from a central piece of information.
```

以下是我的个人总结，可能稍微好理解一点。

首先，maven 是一个工具，用来帮助我们**简化**、**标准化**项目的构建，主要分成四点：

1. **如何描述一个项目**。我们可以简单地用一个坐标（groupId、artifactId、version）来描述一个项目。
2. **将项目的构建分为哪些阶段**。maven 将项目的构建过程标准化，划分为多个有序的阶段，例如，默认生命周期大致包括：编译、测试、打包、安装、部署等。
3. **如何发布和共享项目**。maven 项目的发布和共享基于**仓库**和**坐标**两个基础，我们可将项目发布到仓库，其他人可以通过项目的坐标从仓库中获取这个项目。
4. **如何处理项目间的关系**。我们可以在 pom.xml 配置对应的坐标来依赖其他的项目，而不需要手动地将众多的 jar 包添加到 classpath 中。


# 下载、安装

## 项目环境

maven：3.6.3

操作系统：win10

JDK：8u231

## 下载、安装

进入[官网下载地址](https://maven.apache.org/download.cgi)，根据自己的操作系统和 JDK 选择合适的 maven 版本，这里我们也可以选择下载二进制安装包或者源码包。这里我选择版本 3.6.3 的二进制安装包。

<img src="https://img2020.cnblogs.com/blog/1731892/202007/1731892-20200722133541794-1800207055.png" alt="img_maven_download" style="zoom:80%;" />

将下载的 .zip 文件解压，可以看到以下的目录结构：

<img src="https://img2020.cnblogs.com/blog/1731892/202007/1731892-20200722133612955-379423914.png" style="zoom:67%;" />

进行到这一步可以说 maven 已经安装好了，只是我们还需要进行简单的配置。

## 环境配置

首先，因为 maven 是由 Java 编写，需要 JDK 才能运行，所以，我们必须保证配置好了 JAVA_HOME 的环境变量。这个我就不展开了。

然后，将解压文件中的 bin 目录添加到 Path 的环境变量中（我的电脑(右键属性)->高级系统设置->环境变量），如下所示：

![img_maven_environment_variable](https://img2020.cnblogs.com/blog/1731892/202007/1731892-20200722133637978-188291846.png)


## 测试

在任一位置打开命令行，输入：`mvn -v`或`mvn --version`，显示以下内容，说明安装完成。

![img_maven_mvn_v](https://img2020.cnblogs.com/blog/1731892/202007/1731892-20200722133701551-947328959.png)


# 简单使用maven

安装完成后，下面通过一个简单的例子来模拟构建 maven 项目的过程。

## 生成maven项目 

cmd 进入到你想要存放项目的文件夹，输入以下命令：

```shell
mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=my-app -DarchetypeArtifactId=maven-archetype-quickstart -DarchetypeVersion=1.4 -DinteractiveMode=false
```

在这个命令中，`archetype:generate`为一个`goal`（后面展开分析），而后面那些都是执行这个`goal`所需的参数。

如果是你的 maven 是刚安装的，这个命令可以会执行比较久，因为 maven 需要将所需的软件包或其他文件下载到你的本地仓库（默认在 ${user.home}/.m2/repository 目录下）。如果出现连接超时等情况，可以尝试多执行几次（可以将`settings.xml`的仓库镜像配置为其他地址来提高下载速度）。

执行完这个命令，可以看到指定文件夹下生成了一个 maven 项目，`cd my-app`，它的文件结构如下：

```shell
my-app
|-- pom.xml
`-- src
    |-- main
    |   `-- java
    |       `-- com
    |           `-- mycompany
    |               `-- app
    |                   `-- App.java
    `-- test
        `-- java
            `-- com
                `-- mycompany
                    `-- app
                        `-- AppTest.java
```

其中， 

`src/main/java`目录用来放项目的代码，这些代码将会被编译并打包。

 `src/test/java`目录用来放项目的测试代码，这些代码仅进行编译运行，不打包。

如果我们想要添加一些资源文件，可以在`src/main`目录下创建一个`resource`目录，这些资源最终也会被打包到项目根目录下。

```shell
my-app
|-- pom.xml
`-- src
    |-- main
    |   |-- java
    |   |   `-- com
    |   |       `-- mycompany
    |   |           `-- app
    |   |               `-- App.java
    |   `-- resources
    |       `-- META-INF
    |           `-- application.properties
    `-- test
        `-- java
            `-- com
                `-- mycompany
                    `-- app
                        `-- AppTest.java
```

 `pom.xml`则包含了构建这个项目的配置信息，后面我们将重点介绍这个文件。

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.mycompany.app</groupId>
  <artifactId>my-app</artifactId>
  <version>1.0-SNAPSHOT</version>

  <name>my-app</name>
  <!-- FIXME change it to the project's website -->
  <url>http://www.example.com</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.7</maven.compiler.source>
    <maven.compiler.target>1.7</maven.compiler.target>
  </properties>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
</project>
```

上面生成的项目中包含了两个类，其中，App.java 有一个打印 Hello World! 的 main 方法。接下来我们尝试将项目打包并运行。

## 构建项目

maven 将构建和部署项目的过程定义成了很多个有序的`phase`（可以理解为步骤），我们可以执行以下命令来打包项目。

```shell
mvn package
```

这个命令将执行`package`之前的`phase`，如`validate`、`compile`、`test`等，以及执行`package`本身。严格上来讲，实际上执行的不是`phase`，而是绑定在这些`phase`上的`goal`，后面会展开讲解。

执行成功后，项目根目录生成了一个 target 文件夹，里面就有打包好的 my-app-1.0-SNAPSHOT.jar。

```shell
my-app\target
│  my-app-1.0-SNAPSHOT.jar
│
├─classes
│  └─com
│      └─mycompany
│          └─app
│                  App.class
│
├─generated-sources
│  └─annotations
├─generated-test-sources
│  └─test-annotations
├─maven-archiver
│      pom.properties
│
├─maven-status
│  └─maven-compiler-plugin
│      ├─compile
│      │  └─default-compile
│      │          createdFiles.lst
│      │          inputFiles.lst
│      │
│      └─testCompile
│          └─default-testCompile
│                  createdFiles.lst
│                  inputFiles.lst
│
├─surefire-reports
│      com.mycompany.app.AppTest.txt
│      TEST-com.mycompany.app.AppTest.xml
│
└─test-classes
    └─com
        └─mycompany
            └─app
                    AppTest.class
```

## 运行项目

接下来就是运行 jar 包了，在命令行输入

```shell
java -cp target/my-app-1.0-SNAPSHOT.jar com.mycompany.app.App
```

执行完成，可以看到打印：

```shell
Hello World!
```

# maven 的构建生命周期

构建生命周期（  build lifecycle ）是 maven 的**核心**理论基础之一，它将项目的构建过程标准化。

maven 有三个**独立**的生命周期：

1. 默认生命周期。用来定义项目构建的过程。
2. 清理生命周期。用来定义项目清理的过程。
3. 站点生命周期。用来定义项目站点发布的过程。

## phases

一个生命周期包括了许多具体的`phase`（可以理解为步骤），如下：

### 默认生命周期

一般我们接触比较多的是默认生命周期，它**主要**包括以下过程：

- `validate` - 校验项目是一个正确的 maven 项目
- `compile` - 编译代码
- `test` - 测试`src/test/java`中的方法，`src/test/java`的内容仅作为测试使用，不会进行打包或部署
- `package` - 将项目打包为可执行的 jar、war 等二进制软件包。
- `install` - 将软件包安装到本地仓库
- `deploy` - 将软件包部署到远程仓库

除了这几个常用的`phase`，还有` initialize `、` generate-sources `、` process-sources `等等，需要注意一点，当我们执行某个阶段的命令时，类似 `pre-*`, `post-*`, or `process-*` 的阶段一般只是产生中间结果，并不会对最终构建结果产生影响。

### 清理生命周期

`pre-clean	` - 在清理项目前执行一些东西

` clean `- 清理项目，例如删除 target 包

`post-clean`- 在清理项目后执行一些东西

### 站点生命周期

`pre-site` - 在生成站点文档前执行一些东西

`site` - 生成站点文档

`post-site` - 在生成站点文档后、部署站点文档前执行一些东西

` site-deploy ` -  部署站点文档

## goals

在下面这个 maven 命令中，`archetype:generate`称之为一个`goal`，后面那些都是执行这个`goal`所需的参数。

```shell
mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=my-app -DarchetypeArtifactId=maven-archetype-quickstart -DarchetypeVersion=1.4 -DinteractiveMode=false
```

至于`goal`，对应的是某个插件的某个方法，执行这个命令，我们可以在界面中看到执行的是`maven-archetype-plugin`插件的`generate`方法

<img src="https://img2020.cnblogs.com/blog/1731892/202007/1731892-20200722133822081-787297944.png" alt="img_maven_goal01" style="zoom:80%;" />

## bindings

和上面这个命令不同，下面的这个命令我们并没有传入`goal`，传入的是`phase`。

```java
mvn clean
```

执行这个命令，可以看到，这个命令也是执行了插件的方法，换句话来讲，就是执行了`goal`。

<img src="https://img2020.cnblogs.com/blog/1731892/202007/1731892-20200722133846484-563007827.png" alt="img_maven_phase01" style="zoom: 67%;" />


这里就涉及到一个很重要的概念：**当我们在命令中指定了`phase`，执行的并不是`phase`本身，而是绑定在`phase`上面的`goal`，绑定的`goal`数量可以是一个也可以是多个**。

下面是官方给的部分`binding`，`phase`和`goal`的绑定关系主要和项目的`packaging`配置有关。

| Phase                    | plugin:goal                                                  |
| :----------------------- | :----------------------------------------------------------- |
| `process-resources`      | `resources:resources`                                        |
| `compile`                | `compiler:compile`                                           |
| `process-test-resources` | `resources:testResources`                                    |
| `test-compile`           | `compiler:testCompile`                                       |
| `test`                   | `surefire:test`                                              |
| `package`                | `ejb:ejb` *or* `ejb3:ejb3` *or* `jar:jar` *or* `par:par` *or* `rar:rar` *or* `war:war` |
| `install`                | `install:install`                                            |
| `deploy`                 | `deploy:deploy`                                              |

这些绑定关系，在`${MAVEN_HOME}\lib\maven-core-3.6.3\META-INF\plexus\ default-bindings.xml`中定义。

## mvn [phase]命令的运行

当我们执行`phase`命令时，在执行指定`phase`之前，会先**有序**地执行指定`phase`之前的`phase`以及它本身。例如，我执行`mvn package`，会出现下面的信息：

<img src="https://img2020.cnblogs.com/blog/1731892/202007/1731892-20200722133943607-479903791.png" alt="img_maven_phase02" style="zoom: 67%;" />


在`package`之前的`phase`，包括`compile`、`test`等都会被执行，而且是有序的。

# settings.xml

与`pom.xml`不同，`settings.xml`用于**全局**地配置 maven，而不是配置具体的项目。我们可以在`${maven.home}/conf/`目录下找到这个文件。

`settings.xml`文件主要包含以下节点：

```xml
    <settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                          https://maven.apache.org/xsd/settings-1.0.0.xsd">
      <localRepository/>
      <interactiveMode/>
      <offline/>
      <pluginGroups/>
      <servers/>
      <mirrors/>
      <proxies/>
      <profiles/>
      <activeProfiles/>
    </settings>
```

这个文件的配置可以参考[Settings Reference]( http://maven.apache.org/settings.html )。这里我补充下`servers`、`mirrors`、`profiles`这三个节点的内容。

## servers--配置仓库认证授权信息

`servers`用于配置仓库（包括下载项目和部署项目的仓库）的认证授权信息，例如，用户密码等。

在具体项目中，我们可以在`pom.xml`中的`repositories`、`pluginRepositories`和`distributionManagement`节点配置用于下载项目和部署项目的仓库，但是我们不能把认证授权的信息放在`pom.xml`文件中，于是`servers`就发挥了作用。

```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                      https://maven.apache.org/xsd/settings-1.0.0.xsd">
  ...
  <servers>
    <server>
      <id>server001</id>
      <username>my_login</username>
      <password>my_password</password>
      <privateKey>${user.home}/.ssh/id_dsa</privateKey>
      <passphrase>some_passphrase</passphrase>
      <filePermissions>664</filePermissions>
      <directoryPermissions>775</directoryPermissions>
      <configuration></configuration>
    </server>
  </servers>
  ...
</settings>
```

## Mirrors--配置仓库的镜像

`Mirrors`用于配置下载项目的仓库镜像。前面说过，国内使用 maven 的中央仓库下载项目比较慢，甚至会出现超时失败的情况，这时，我们就可以通过配置镜像来提高传输速度。在此之前，我们需要区分镜像和仓库两个概念，以下这篇文章作出了很好的解释。[Maven：mirror和repository 区别](https://www.cnblogs.com/bollen/p/7143551.html)

在下面这个例子中，我们使用阿里云的镜像来请求 maven 的中央仓库，注意，`mirror`的`mirrorOf`节点必须指定仓库的 id，当然，这里还支持多种形式。例如，`<mirrorOf>*</mirrorOf>`表示匹配所有远程仓库；`<mirrorOf>repo1,repo2</mirrorOf>`表示匹配仓库 repo1 和 repo2，使用逗号分隔多个远程仓库；`<mirrorOf>*,!repo1</miiroOf>` 匹配所有远程仓库，repo1 除外，使用感叹号将仓库从匹配中排除。 

```xml
<!-- settings.xml的配置 -->
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                      https://maven.apache.org/xsd/settings-1.0.0.xsd">
  ...
  <mirrors>
  		<mirror>
			<id>alimaven</id>
			<name>aliyun maven</name>
			<url>http://maven.aliyun.com/nexus/content/groups/public/</url>
			<mirrorOf>central</mirrorOf>        
		</mirror>
  </mirrors
  ...
</settings>

<!-- pom.xml的配置 --> 
<project>
  ...
  <repositories>
    <repository>
      <id>central</id>
      <name>Central Repository</name>
      <url>https://repo.maven.apache.org/maven2</url>
      <layout>default</layout>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
    </repository>
  </repositories>
  ...
</project>
```

补充下，`mirror`节点对`repositories`、`pluginRepositories`和`distributionManagement`均生效。

## profiles

`profiles`：提供了一组可选的配置，我们可以根据不同的环境选择激活哪一套配置，它包括:`activation`、 `repositories`、`pluginRepositories` 和`properties`四个节点。其中，`activation`节点用于配置该`profile`在什么环境下才能激活。

```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                      https://maven.apache.org/xsd/settings-1.0.0.xsd">
  ...
  <profiles>
    <profile>
      <id>test</id>
      <activation>
        <activeByDefault>false</activeByDefault>
        <jdk>1.5</jdk>
        <os>
          <name>Windows XP</name>
          <family>Windows</family>
          <arch>x86</arch>
          <version>5.1.2600</version>
        </os>
        <property>
          <name>mavenVersion</name>
          <value>2.0.3</value>
        </property>
        <file>
          <exists>${basedir}/file2.properties</exists>
          <missing>${basedir}/file1.properties</missing>
        </file>
      </activation>
        
	  <properties>
        <user.install>${user.home}/our-project</user.install>
      </properties>
        
      <repositories>
        <repository>
          <id>codehausSnapshots</id>
          <name>Codehaus Snapshots</name>
          <releases>
            <enabled>false</enabled>
            <updatePolicy>always</updatePolicy>
            <checksumPolicy>warn</checksumPolicy>
          </releases>
          <snapshots>
            <enabled>true</enabled>
            <updatePolicy>never</updatePolicy>
            <checksumPolicy>fail</checksumPolicy>
          </snapshots>
          <url>http://snapshots.maven.codehaus.org/maven2</url>
          <layout>default</layout>
        </repository>
      </repositories>
        
      <pluginRepositories>
        ...
      </pluginRepositories>
        
    </profile>
    ...
  </profiles>
  ...
</settings>
```

**注意，如果`settings.xml`的某个`profile`被激活，那么，它的配置将覆盖`pom.xml`中相同 id 的仓库以及相同名称的 property**。

# pom.xml

`pom.xml`几乎包含了对 maven 项目的所有描述信息，包括项目的坐标、依赖关系、构建配置等等。这个文件非常重要，官网有这么一句话，在 maven 的世界里，一个完整的项目可以不包含任何的代码，而只需要一个`pom.xml`。

`pom.xml`的节点如下：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
 
  <!-- The Basics -->
  <groupId>...</groupId>
  <artifactId>...</artifactId>
  <version>...</version>
  <packaging>...</packaging>
  <dependencies>...</dependencies>
  <parent>...</parent>
  <dependencyManagement>...</dependencyManagement>
  <modules>...</modules>
  <properties>...</properties>
 
  <!-- Build Settings -->
  <build>...</build>
  <reporting>...</reporting>
 
  <!-- More Project Information -->
  <name>...</name>
  <description>...</description>
  <url>...</url>
  <inceptionYear>...</inceptionYear>
  <licenses>...</licenses>
  <organization>...</organization>
  <developers>...</developers>
  <contributors>...</contributors>
 
  <!-- Environment Settings -->
  <issueManagement>...</issueManagement>
  <ciManagement>...</ciManagement>
  <mailingLists>...</mailingLists>
  <scm>...</scm>
  <prerequisites>...</prerequisites>
  <repositories>...</repositories>
  <pluginRepositories>...</pluginRepositories>
  <distributionManagement>...</distributionManagement>
  <profiles>...</profiles>
</project>
```

关于`pom.xml`文件的内容，就不详细展开了，可参考[POM Reference]( http://maven.apache.org/pom.html#pom-reference )。这里介绍下两个比较重要的概念。

## scope

`scope`是`dependency`的子节点，用于设置以下两个内容：

1. 依赖是否在（测试）编译、（测试）运行等时机加入 classpath。
2. 限制依赖的传递性。（假设当前项目为 A，它依赖了 B，如果 C 依赖了 A，则 C 也会依赖 B。可以看出，A 将自己对 B 的依赖传递给了 C）

maven 提供了五种`scope`给我们选择，如下。

|    scope     | 编译期 | 运行期 | 测试编译期 | 测试运行期 | 依赖传递 |
| :----------: | :----: | :----: | :--------: | :--------: | :------: |
| **compile**  |   √    |   √    |     √      |     √      |    √     |
| **provided** |   √    |   √    |     √      |     √      |    ×     |
| **runtime**  |   ×    |   √    |     ×      |     √      |    √     |
|   **test**   |   ×    |   ×    |     √      |     √      |    ×     |
|  **system**  |   √    |   √    |     √      |     √      |    √     |

通过`dependency`的子节点` optional`可以改变传递性。**system对应的依赖不会从仓库获取，而是从` systemPath `指定的路径中获取**。


## super pom

pom 文件可以通过`<parent>`节点来继承其他项目的配置信息，而且，和 Java 的对象默认继承 Object 一样，pom 文件默认会去继承 super pom，该 pom 文件的内容见： [Super POM for Maven 3.6.3](http://maven.apache.org/ref/3.6.3/maven-model-builder/super-pom.html)

# 常见问题

## 中央仓库没有的依赖，怎么获取

当我们的项目需要依赖某个在中央仓库中不存在的依赖，例如，`oracle`的驱动包，我们可以采用三种解决方案：

1. 将依赖的项目安装到本地仓库。命令如下：

```shell
mvn install:install-file -Dfile=non-maven-proj.jar -DgroupId=some.group -DartifactId=non-maven-proj -Dversion=1 -Dpackaging=jar
```

2. 将依赖的项目安装到私服。命令如下：

```shell
deploy:deploy-file -Dfile=non-maven-proj.jar -DgroupId=some.group -DartifactId=non-maven-proj -Dversion=1 -Dpackaging=jar
```

3. 使用`system`作用域指定包的路径。

```xml
<dependency>
  <groupId>some.group</groupId>
  <artifactId>non-maven-proj</artifactId>
  <version>1.0</version>
  <scope>system</scope>
  <systemPath>${java.home}/lib/non-maven-proj.jar</systemPath>
</dependency>
```


# 参考资料

[Apache Maven官网](https://maven.apache.org/)

[Maven：mirror和repository 区别](https://www.cnblogs.com/bollen/p/7143551.html)


> 相关源码请移步：[https://github.com/ZhangZiSheng001/maven-demo](https://github.com/ZhangZiSheng001/maven-demo)

>本文为原创文章，转载请附上原文出处链接：[https://www.cnblogs.com/ZhangZiSheng001/p/13360234.html](https://www.cnblogs.com/ZhangZiSheng001/p/13360234.html) 