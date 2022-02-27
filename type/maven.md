### 1. **依赖**

#### 	1.1**依赖范围**

1. compile：编译依赖，默认的依赖范围，对于编译、测试、正式运行三中都有效。

2. test：测试依赖，只对测试代码有效，在编译主代码或者正式运行时无法使用此类依赖。典型的就是JUnit,只有在编译测试代码及运行测试代码时才生效。

3. provided：已提供依赖范围，在编译测试时生效，但在正式运行时无效，典型例子是servlet-api,运行时WEB容器赢提供，无需重复引入。

4. runtime：运行时依赖，在测试和正式运行时生效，编译时无效。

5. import：

   | 依赖范围 | 对于编译<br/>classpath有效 | 对于测试<br/>classpath有效 | 对于运行时<br/>classpath有效 |             例子              |
   | :------: | :------------------------: | :------------------------: | :--------------------------: | :---------------------------: |
   | compile  |             Y              |             Y              |              Y               |          spring-core          |
   |   test   |             --             |             Y              |              --              |             JUnit             |
   | provided |             Y              |             Y              |              --              |      容器中的servlet-api      |
   | runtime  |             --             |             Y              |              Y               |         JDBC驱动实现          |
   |  system  |             Y              |             Y              |              --              | 本地的，maven仓库之外的类文件 |

#### 	1.2 **依赖传递**

|          | compile  | test | provided | runtime  |
| :------: | :------: | :--: | :------: | :------: |
| compile  | compile  |  --  |    --    | runtime  |
|   test   |   test   |  --  |    --    |   test   |
| provided | provided |  --  | provided | provided |
| runtime  | runtime  |  --  |    --    | runtime  |

#### 	1.3 **依赖调解**

​			当发生依赖重复冲突时，maven根据第一原则进行处理，如果不满足第一原则，则按照第二原则处理。

1. 第一原则：最优路径优先，即最短路径。
2. 第二原则：第一声明优先，即按照声明顺序选择。

#### 	1.4 **排除依赖**

​			通过exclusions及子元素exclusion来排除依赖，声明exclousion时只需要groupId和artifactId，不需要指定version,因为只需要groupId和artifactId就能唯一定位依赖。

------



###  2. **仓库**

#### 	2.1 **远程仓库的配置**			

```xml
<project>
	<repositories>
  	<repository>
    	<id>jboss<id>
      <name>JBoss私有仓库</name>
      <url>http://xxx/</url>  
      <releases>
        <enabled>true</enabled>
        <updatePolicy>daily</updatePolicy>
        <checksumPolicy>warn</checksumPolicy>
      </releases>
      <snapshot>
        <enabled>false</enabled>
        <updatePolicy>interval:30</updatePolicy>
        <checksumPolicy>ignore</<checksumPolicy>>
      </snapshot>
    </repository>
  </repositories>
</project>
```

1. 在repositories元素下，可以使用repository子元素声明一个或多个远程仓库，其中id为远程仓库的唯一标识且必须是唯一的。Maven中央仓库使用的id是central，如果其他仓库的id也设置为central，则会覆盖中央仓库。
2. repository中的release和snapshots元素比较重要，是用于控制Maven对发布版构件和快照版构件的下载，通过子元素enable的值来控制，当为true时，标识开启远程仓库对应版本的下载，设置为false，标识关闭远程仓库的对应版本的下载。
3. release和snapshots的子元素updatePolicy用来配置从远程仓库检查更新的频率，默认值是daily，标识每天检查一次，值为never时表示冲不检查更新，always表示每一次构件都进行检查，interval:x 表示每隔x(正整数)分钟检查一次更新。
4. release和snapshots的子元素checksumPolicy用来配置检查检验和文件的策略，因为当构件被部署到仓库中，同同时部署的还包括校验和文件，后续在下载构件时，maven会验证校验和文件。当checksumPolicy为warn时，会在执行构建时输出警告信息，值为fail时maven会让构建失败，为ignore时会在构建项目完全忽略校验和文件。

#### **2.2 仓库认证**

有些私服仓库访问时需要认证，比如一般企业内部的maven仓库，认证信息配置在maven的setting.xml文件中。

```xml
<settings>
  ……
	<servers>
  	<server>
   		<id>远程仓库的标识ID</id>
      <username>用户名</username>
      <password>密码</password>
    </server>
  </servers> 
  ……
</settings>
```

#### **2.3 部署至远程仓库**

​	在项目pom.xml中配置distributionManagement元素，如下：

```xml
<project>
	<distributionManagement>
  	<repository>
    	<id>releases</id>
      <name>名称描述</name>
      <url>http://xxx/content/repositories/release</url>
    </repository>
    <snapshotRepository>
    	<id>releases</id>
      <name>名称描述</name>
      <url>http://xxx/content/repositories/release</url>
    </snapshotRepository>
  <distributionManagement>
</project>
```

- ​	子元素repository表示正式发布版本构件的仓库，snapshotRepository表示要发布快照版本的仓库。其中两个子元素都包括了id（远程仓库的唯一表示），name（名称方便阅读理解）、url（仓库的具体地址）。

#### **2.4 镜像仓库**

​	仓库X可以提供仓库Y存储的所有内容，那仓库X就是Y的镜像，镜像仓库在setting.xml文件中配置，如下：

```xml
<settings>
	……
  <mirrors>
    <mirror>
    	<id>releases</id>
      <name>名称描述</name>
      <url>http://xxx/content/repositories/release</url>
      <mirrorOf>central</mirrorOf><!--取值也可以是 * 号 -->
  	</mirror>
  </mirrors>
  ……
</settings>
```

​	  其中mirrorOf子元素指定该镜像是哪个仓库的镜像，如central，表示是中央仓库的镜像。任何对于中央仓库的请求都会转至该镜像仓库。

```xml
<!--当mirrorOf取值*号时，表示是所有maven仓库的镜像，任何对于远程仓库的请求都会被转至该镜像仓库。-->
<mirrorOf>central</mirrorOf>

<!--匹配多个仓库是，使用逗号分隔。-->
<mirrorOf>repo1,repo2</mirrorOf>

<!--匹配所有远程仓库，repo1仓库除外。-->
<mirrorOf>*,!repo1</mirrorOf>
```

> 镜像仓库会完全屏蔽被镜像的仓库，所以当镜像仓库不稳定或者停止服务时，maven将无法访问仓库并下载构件。

### 3. **生命周期与插件**

#### 	**3.1 三套生命周期**

1. clean 的生命周期目的是清理项目，又包含以下三个阶段。

   1.1 pre-clean：执行清理前需要完成的工作。

   1.2 clean：清理上一次构建生成的文件。

   1.3 post-clean：执行一些清理后需要完成的工作。

2. default生命周期定义了真正构建时需要执行的所有步骤，是核心部分，以下列举default中主要的步骤。

   2.1 process-sources：处理项目主资源文件。一般来说，是对src/main/resources目录的 内容进行变量替换等工	  作后，复制到项目输出的主classpath目录中。

   2.2 process-test-sources：处理项目测试资源文件。一般来说，是对src/test/resources目 录的内容进行变量替换	  等工作后，复制到项目输出的测试classpath目录中。

   2.3 compile：编译项目的主源码。一般来说，是编译src/main/java目录下的Java文件至 项目输出的主classpath	  目录中。

   2.4 test-compile：编译项目的测试代码。一般来说，是编译src/test/java目录下的Java 文件至项目输出的测试		classpath目录中。

   2.5 test：使用单元测试框架运行测试，测试代码不会被打包或部署。

   2.6 package：接受编译好的代码，打包成可发布的格式，如JAR。

   2.7 install：将包安装到Maven本地仓库，供本地其他Maven项目使用。 

   2.8 deploy：将最终的包复制到远程仓库，供其他开发人员和Maven项目使用。

3. site生命周期的目的是建立和发布项目站点，Maven能够基于POM所包含的信 息，自动生成一个友好的站点，方便团队交流和发布项目信息。该生命周期包含如 下阶段： 

   3.1 pre-site：执行一些在生成项目站点之前需要完成的工作。 

   3.2 site：生成项目站点文档。 

   3.3 post-site：执行一些在生成项目站点之后需要完成的工作。 ·site-deploy将生成的项目站点发布到服务器上。

#### 	**3.2 命令行与生命周期**

| 命令              | 说明                                                         |
| ----------------- | :----------------------------------------------------------- |
| mvn clean         | 该命令调用clean生命周期的clean阶段。实际执行的阶段为clean生 命周期的pre-clean和clean阶段。 |
| mvn test          | 该命令调用default生命周期的test阶段。实际执行的阶段为default生 命周期的validate、initialize等，直到test的所有阶段。这也解释了为什么在执行测试 的时候，项目的代码能够自动得以编译。 |
| mvn clean install | 该命令调用clean生命周期的clean阶段和default生命周期的install阶段。实际执行的阶段为clean生命周期的pre-clean、clean阶段，以及default生 命周期的从validate至install的所有阶段。该命令结合了两个生命周期，在执行真正的 项目构建之前清理项目是一个很好的实践。 |
| mvn clean deploy  | 该命令调用clean生命周期的clean阶段、default生命周期的deploy阶段，以及site生命周期的site-deploy阶段。实际执行的阶段为clean 生命周期的pre-clean、clean阶段，default生命周期的所有阶段 |

#### 	3.3 插件目标

Maven的核心仅仅定义了抽象的生命周期，具体的任务是交由插件完成。插件以独立的构件形式存在，因此，Maven核心的分发包只有不到 3MB的大小，Maven会在需要的时候下载并使用插件。对于插件本身，为了能够复用代码，它往往能够完成多个任务。

> 例如 mavendependency-plugin，它能够基于项目依赖做很多事情。它能够分析项目依赖，帮助找 出潜在的无用依赖；它能够列出项目的依赖树，帮助分析依赖来源；它能够列出项 目所有已解析的依赖，等等。为每个这样的功能编写一个独立的插件显然是不可取 的，因为这些任务背后有很多可以复用的代码，因此，这些功能聚集在一个插件 里，每个功能就是一个插件目标。maven-dependency-plugin有十多个目标，每个目标对应了一个功能，上述提到的 几个功能分别对应的插件目标为dependency：analyze、dependency：tree和 dependency：list。这是一种通用的写法，冒号前面是插件前缀，冒号后面是该插件 的目标。类似地，还可以写出compiler：compile（这是maven-compiler-plugin的 compile目标）和surefire：test（这是maven-surefire-plugin的test目标）。

#### **3.4 内置绑定**

以下是default生命周期的内置插件丙丁关系及具体任务，以下之列出了default生命周期中拥有插件绑定关系的阶段。

| 生命周期              | 插件目标                             | 执行任务                       |
| --------------------- | ------------------------------------ | ------------------------------ |
| process-resources     | maven-resources-plugin:resources     | 复制资源文件至主输出目录       |
| compile               | maven-compile-plugin:compile         | 编译主代码至主输出目录         |
| process-test-resource | maven-resources-plugin:testResources | 复制测试资源文件至测试输出目录 |
| test-compile          | maven-compile-plugin:testCompile     | 编译测试代码至测试输出目录     |
| test                  | maven-surefire-plugin:test           | 执行项目测试用例               |
| package               | maven-jar-plugin:jar                 | 构件打包                       |
| install               | maven-install-plugin:install         | 构件安装到本地仓库             |
| deploy                | maven-deploy-plugin:deploy           | 构件安装到远程仓库             |



​		
