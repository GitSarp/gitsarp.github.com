#### 构建工具
- make
- Ant
- maven
### quickstart
- 创建工程：mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=my-app -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
- 构建mvn package
- 测试java -cp target/my-app-1.0-SNAPSHOT.jar com.mycompany.app.App
### Build Lifecycle构建生命周期
构建生命周期是一组阶段的序列（sequence of phases），每个阶段定义了目标被执行的顺序。
典型：
| 阶段              | 处理     | 描述                                                 |
| ----------------- | -------- | ---------------------------------------------------- |
| prepare-resources | 资源拷贝 | 本阶段可以自定义需要拷贝的资源                       |
| compile           | 编译     | 本阶段完成源代码编译                                 |
| package           | 打包     | 本阶段根据 pom.xml 中描述的打包配置创建 JAR / WAR 包 |
| install           | 安装     | 本阶段在本地 / 远程仓库中安装工程包                  |
#### 标准生命周期
- clean
- default(or build)
- site
#### clean生命周期

- pre-clean
- clean
- post-clean
将 maven-antrun-plugin:run 目标添加到 pre-clean、clean 和 post-clean 阶段中,这样可以在 clean 生命周期的各个阶段显示文本信息:
```
 <plugin>
   <groupId>org.apache.maven.plugins</groupId>
   <artifactId>maven-antrun-plugin</artifactId>
   <version>1.1</version>
   <executions>
      <execution>
         <id>id.pre-clean</id>
         <phase>pre-clean</phase>
         <goals>
            <goal>run</goal>
         </goals>
         <configuration>
            <tasks>
               <echo>pre-clean phase</echo>
            </tasks>
         </configuration>
      </execution>
      <execution>
         <id>id.clean</id>
         <phase>clean</phase>
         <goals>
          <goal>run</goal>
         </goals>
         <configuration>
            <tasks>
               <echo>clean phase</echo>
            </tasks>
         </configuration>
      </execution>
      <execution>
         <id>id.post-clean</id>
         <phase>post-clean</phase>
         <goals>
            <goal>run</goal>
         </goals>
         <configuration>
            <tasks>
               <echo>post-clean phase</echo>
            </tasks>
         </configuration>
      </execution>
   </executions>
   </plugin>
```
#### Default (or Build) 生命周期
当一个阶段通过 Maven 命令调用时，例如 mvn compile，该阶段之前以及包括该阶段在内的所有阶段会被执行。

| 生命周期阶段          | 描述                                                         |
| --------------------- | ------------------------------------------------------------ |
| **validate**              | 检查工程配置是否正确，完成构建过程的所有必要信息是否能够获取到。 |
| initialize            | 初始化构建状态，例如设置属性。                               |
| generate-sources      | 生成编译阶段需要包含的任何源码文件。                         |
| process-sources       | 处理源代码，例如，过滤任何值（filter any value）。           |
| generate-resources    | 生成工程包中需要包含的资源文件。                             |
| process-resources     | 拷贝和处理资源文件到目的目录中，为打包阶段做准备。           |
| **compile**               | 编译工程源码。                                               |
| process-classes       | 处理编译生成的文件，例如 Java Class 字节码的加强和优化。     |
| generate-test-sources | 生成编译阶段需要包含的任何测试源代码。                       |
| process-test-sources  | 处理测试源代码，例如，过滤任何值（filter any values)。       |
| test-compile          | 编译测试源代码到测试目的目录。                               |
| process-test-classes  | 处理测试代码文件编译后生成的文件。                           |
| **test**                  | 使用适当的单元测试框架（例如JUnit）运行测试。                |
| prepare-package       | 在真正打包之前，为准备打包执行任何必要的操作。               |
| **package**               | 获取编译后的代码，并按照可发布的格式进行打包，例如 JAR、WAR 或者 EAR 文件。 |
| pre-integration-test  | 在集成测试执行之前，执行所需的操作。例如，设置所需的环境变量。 |
| integration-test      | 处理和部署必须的工程包到集成测试能够运行的环境中。           |
| post-integration-test | 在集成测试被执行后执行必要的操作。例如，清理环境。           |
| **verify**                | 运行检查操作来验证工程包是有效的，并满足质量要求。           |
| **install**               | 安装工程包到本地仓库中，该仓库可以作为本地其他工程的依赖。   |
| **deploy**                | 拷贝最终的工程包到远程仓库中，以共享给其他开发人员和工程。   |

### Site生命周期

- pre-site
- site
- post-site
- site-deploy

### maven核心插件

- clean  构建完之后的清理（Clean up after the build.）
- compiler  编译源码（Compiles Java sources.）
	 deploy 部署编译后的产品到远程仓库（ 	Deploy the built artifact to the remote repository.）
- failsafe在隔离的classloader上允许junit集成测试（Run the JUnit integration tests in an isolated classloader）
	 install 	安装编译后的产品到本地仓库Install the built artifact into the local repository. 
	 resources 		Copy the resources to the output directory for including in the JAR. 	
	 site 		Generate a site for the current project. 	
	 surefire 		Run the JUnit unit tests in an isolated classloader. 	
- verifier：Useful for integration tests - verifies the existence of certain conditions.

```
<build>
		<pluginManagement>
			<plugins>
				<!-- 以防编译出现中文乱码导致页面乱码 -->
				<plugin>
					<groupId>org.apache.maven.plugins</groupId>
					<artifactId>maven-compiler-plugin</artifactId>
					<version>2.3.2</version>
					<configuration>
						<source>1.7</source>
						<target>1.7</target>
						<encoding>UTF-8</encoding>
					</configuration>
				</plugin>
		</pluginManagement>
</build>
```
```
mvn [plugin-name]:[goal-name]
eg.mvn compiler:compile
```

### 常用命令

`mvn archetype：create` 创建Maven项目

 `mvn compile` 编译源代码

 `mvn deploy` 发布项目

 `mvn test-compile` 编译测试源代码

 `mvn test` 运行应用程序中的单元测试

 `mvn site` 生成项目相关信息的网站

 `mvn clean` 清除项目目录中的生成结果

 `mvn package` 根据项目生成的jar

 `mvn install` 在本地Repository中安装jar

 `mvn eclipse:eclipse` 生成eclipse项目文件

 `mvnjetty:run` 启动jetty服务

 `mvntomcat:run` 启动tomcat服务

 `mvn clean package -Dmaven.test.skip=true` 清除以前的包后重新打包，跳过测试类
