# maven
## 自动化构建工具
Maven这个单词的本意是：专家，内行。读音是['meɪv(ə)n]或['mevn]，不要读作“妈文”。
Maven是一款自动化构建工具，专注服务于Java平台的项目构建和依赖管理。在JavaEE开发的历史上构建工具的发展也经历了一系列的演化和变迁：
Make→Ant→Maven→Gradle→其他……

## Maven核心概念
Maven之所以能够实现自动化的构建，和它的设计是紧密相关的。我们对Maven的学习就围绕它的九个核心概念展开：
1. POM
2. 约定的目录结构
3. 坐标
4. 依赖管理
5. 仓库管理
6. 生命周期
7. 插件和目标
8. 继承
9. 聚合

## How
1. 检查JAVA_HOME环境变量。Maven是使用Java开发的，所以必须知道当前系统环境中JDK的安装目录。
````
C:\Windows\System32>echo %JAVA_HOME%
C:\Java\jdk1.8.0_45
````
2. 解压Maven的核心程序。将apache-maven-3.5.0-bin.zip解压到一个非中文无空格的目录下。
3. 配置环境变量。
4. 查看Maven版本信息验证安装是否正确
````
Apache Maven 3.5.0 (ff8f5e7444045639af65f6095c62210b5713f426; 2017-04-04T03:39:06+08:00)
Maven home: D:\Server\apache-maven-3.5.0\bin\..
Java version: 11.0.1, vendor: Oracle Corporation
Java home: C:\Program Files\Java\jdk-11.0.1
Default locale: zh_CN, platform encoding: GBK
OS name: "windows 10", version: "10.0", arch: "amd64", family: "windows"
````
5. 配置本地仓库
     1. Maven默认的本地仓库：~\.m2\repository目录。Tips：~表示当前用户的家目录。
     2. Maven的核心程序并不包含具体功能，仅负责宏观调度。具体功能由插件来完成。Maven核心程序会到本地仓库中查找插件。如果本地仓库中没有就会从远程中央仓库下载。此时如果不能上网则无法执行Maven的具体功能。为了解决这个问题，我们可以将Maven的本地仓库指向一个在联网情况下下载好的目录。
     3. Maven的核心配置文件位置：解压目录\ D:\Server\ apache-maven-3.5.0\conf\settings.xml
     4. 设置方式  \<localRepository>以及准备好的仓库位置\</localRepository><localRepository>D:/RepMaven</localRepository>


## POM
**Project Object Model：项目对象模型。将Java工程的相关信息封装为对象作为便于操作和管理的模型。Maven工程的核心配置。可以说学习Maven就是学习pom.xml文件中的配置。**

## Maven的坐标
使用如下三个向量在Maven的仓库中唯一的确定一个Maven工程：
1. groupId：公司或组织的域名倒序+当前项目名称
2. artifactId：当前项目的模块名称
4. version：当前模块的版本
````xml
<groupId>com.atguigu.maven</groupId>
<artifactId>Hello</artifactId>
<version>0.0.1-SNAPSHOT</version>
````

## 依赖管理
### 基本概念
当A jar包需要用到B jar包中的类时，我们就说A对B有依赖。例如：commons-fileupload-1.3.jar依赖于commons-io-2.0.1.jar。
通过第二个Maven工程我们已经看到，当前工程会到本地仓库中根据坐标查找它所依赖的jar包。
配置的基本形式是使用dependency标签指定目标jar包的坐标。例如：
````xml
	<dependencies>
		<dependency>
			<!—坐标 -->
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.10</version>
			<!-- 依赖的范围 -->
			<scope>test</scope>
		</dependency>
	</dependencies>
````
### 直接依赖和间接依赖
如果A依赖B，B依赖C，那么A→B和B→C都是直接依赖，而A→C是间接依赖。

### 依赖的范围
当一个Maven工程添加了对某个jar包的依赖后，这个被依赖的jar包可以对应下面几个可选的范围：
1. compile
     1. main目录下的Java代码可以访问这个范围的依赖
     2. test目录下的Java代码可以访问这个范围的依赖
     3. 部署到Tomcat服务器上运行时要放在WEB-INF的lib目录下
2. test
     1. main目录下的Java代码不能访问这个范围的依赖
     2. test目录下的Java代码可以访问这个范围的依赖
     3. 部署到Tomcat服务器上运行时不会放在WEB-INF的lib目录下
3. provided
     1. main目录下的Java代码可以访问这个范围的依赖
     2. test目录下的Java代码可以访问这个范围的依赖
     3. 部署到Tomcat服务器上运行时不会放在WEB-INF的lib目录下
4. runtime
     1. main目录下的Java代码不能访问这个范围的依赖
     2. test目录下的Java代码可以访问这个范围的依赖
     3. 部署到Tomcat服务器上运行时会放在WEB-INF的lib目录下
5. 其他：import、system等。

### 依赖的传递性
当存在间接依赖的情况时，主工程对间接依赖的jar可以访问吗？这要看间接依赖的jar包引入时的依赖范围——只有依赖范围为compile时可以访问。

### 依赖的原则：解决jar包冲突
1. 路径最短者优先
2. 路径相同时先声明者优先
**这里“声明”的先后顺序指的是dependency标签配置的先后顺序。**

### 依赖的排除
有的时候为了确保程序正确可以将有可能重复的间接依赖排除。请看如下的例子：
1. 假设当前工程为survey_public，直接依赖survey_environment。
2. survey_environment依赖commons-logging的1.1.1对于survey_public来说是间接依赖。
3. 当前工程survey_public直接依赖commons-logging的1.1.2
4. 加入exclusions配置后可以在依赖survey_environment的时候排除版本为1.1.1的commons-logging的间接依赖
````xml
<dependency>
	<groupId>com.atguigu.maven</groupId>
	<artifactId>Survey160225_4_Environment</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<!-- 依赖排除 -->
	<exclusions>
		<exclusion>
			<groupId>commons-logging</groupId>
			<artifactId>commons-logging</artifactId>
		</exclusion>
	</exclusions>
</dependency>
<dependency>
	<groupId>commons-logging</groupId>
	<artifactId>commons-logging</artifactId>
	<version>1.1.2</version>
</dependency>
````


## 仓库
### 分类
1. 本地仓库：为当前本机电脑上的所有Maven工程服务。
2. 远程仓库
     1. 私服：架设在当前局域网环境下，为当前局域网范围内的所有Maven工程服务。
     2. 中央仓库：架设在Internet上，为全世界所有Maven工程服务。
     3. 中央仓库的镜像：架设在各个大洲，为中央仓库分担流量。减轻中央仓库的压力，同时更快的响应用户请求。

## 生命周期
### Maven的生命周期
●Maven生命周期定义了各个构建环节的执行顺序，有了这个清单，Maven就可以自动化的执行构建命令了。  
●Maven有三套相互独立的生命周期，分别是： 
1. Clean Lifecycle在进行真正的构建之前进行一些清理工作。
2. Default Lifecycle构建的核心部分，编译，测试，打包，安装，部署等等。
3. Site Lifecycle生成项目报告，站点，发布站点。

### clean生命周期
Clean生命周期一共包含了三个阶段：
1. pre-clean 执行一些需要在clean之前完成的工作 
2. clean 移除所有上一次构建生成的文件 
3. post-clean 执行一些需要在clean之后立刻完成的工作 

###  Site生命周期
1. pre-site 执行一些需要在生成站点文档之前完成的工作
2. site 生成项目的站点文档
3. post-site 执行一些需要在生成站点文档之后完成的工作，并且为部署做准备
4. site-deploy 将生成的站点文档部署到特定的服务器上

## 继承
### 创建父工程、
创建父工程和创建一般的Java工程操作一致，唯一需要注意的是：打包方式处要设置为pom。
### 在子工程中引用父工程
````xml
<parent>
	<!-- 父工程坐标 -->
	<groupId>...</groupId>
	<artifactId>...</artifactId>
	<version>...</version>
	<relativePath>从当前目录到父项目的pom.xml文件的相对路径</relativePath>
</parent>

<parent>
	<!-- 父工程坐标 -->
	<groupId>...</groupId>
	<artifactId>...</artifactId>
	<version>...</version>
	<relativePath>从当前目录到父项目的pom.xml文件的相对路径</relativePath>
</parent>

````
### 在父工程中管理依赖
1. 将Parent项目中的dependencies标签，用dependencyManagement标签括起来
````xml
<dependencyManagement>
	<dependencies>
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.9</version>
			<scope>test</scope>
		</dependency>
	</dependencies>
</dependencyManagement>
````

2. 在子项目中重新指定需要的依赖，删除范围和版本号
````xml
<dependencies>
	<dependency>
		<groupId>junit</groupId>
		<artifactId>junit</artifactId>
	</dependency>
</dependencies>
````

# Maven酷站
我们可以到http://mvnrepository.com/搜索需要的jar包的依赖信息。
http://search.maven.org/



