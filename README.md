# Maven多模块version管理

> 对于使用maven构建的java项目，通常聚合多个子模块项目。在版本迭代的过程中经常变更版本号，更新parent版本号，发现子模块版本号没有更新，需要一个个的手动去更新，太麻烦，且容易出错。
> 在版本更新之后，内部模块之间的依赖也需要变更，往往存在遗漏。


### MAVEN多个子模块项目
![项目结构](https://img-blog.csdnimg.cn/20190116184713355.png)

以上是一个基本的

*  **主项目parent包**
`maven-multily-module/pom.xml`
   1. 指定整个应用的dependencyManagement
   2. 定义项目的发布的仓库地址distributionManagement
   3. 所有第三方依赖的版本号全部定义在properties下
   4. 所有内部模块依赖版本号统一使用**${project.version}**
   5. 指定所有的子模块modules

* **项目子模块pom.xml**
   `app-api/pom.xml;app-dao/pom.xml;util/pom.xml;trade-core/pom.xlm;user-core/pom.xml`
   1. 明确定义parent模块的`artifactId，groupId，version`
   2. 不要定义子模块的version（同parent保持一致）
   3. 子模块无需定义groupId
   5. 子模块所有的依赖包版本全部集成parent模块，即：子模块不得定义依赖包版本号
   5. 子模块需定义是否需要deploy到私服`<maven.deploy.skip>true</maven.deploy.skip>`
   6. 对于需要depoly的子模块【对外发布的，比如dubbo提供的api包】不应该依赖重量级jar包(比如：spring,mybatis等)

* **子模块packaging为pom**
`app-core/pom.xml`
   1. 指定所有的子模块modules
   2. 无需定义groupId
   3. 明确定义parent模块的`artifactId，groupId，version`

以上定义规则保证了项目内部模块之间的依赖版本统一，第三方依赖包版本不冲突

### MAVEN聚合多个子模块项目版本号修改

虽然按照以上的规则定义模块及依赖，但是在版本迭代过程中需要修改对应的版本号，还是需要顶级pom的版本号，及每个子模块内部parent的版本号。

```
   <parent>
        <groupId>com.maven.multily.module</groupId>
        <artifactId>parent</artifactId>
        <!--版本升级需要修改每个子模块 parent.version的值-->
        <version>1.1.0-SNAPSHOT</version>
    </parent>
```
这种手工修改方式极容易遗漏，导致项目内部模块版本依赖存在问题。

我们可以通过maven的插件方式来升级整个项目的版本号。方案如下：

* 在项目顶层pom中添加version插件

```
 <plugin>
      <groupId>org.codehaus.mojo</groupId>
      <artifactId>versions-maven-plugin</artifactId>
      <version>2.3</version>
      <configuration>
          <generateBackupPoms>false</generateBackupPoms>
      </configuration>
  </plugin>
```

* 在项目根目录下执行以下命令修改版本号

```
// 设置新的版本号未1.2.0-SNAPSHOT
mvn versions:set -DnewVersion=1.2.0-SNAPSHOT
```
以上命令会将maven-multily-module/pom.xml版本修改为1.2.0-SNAPSHOT，且会修改所有子模块内 parent的version为1.2.0-SNAPSHOT。所以建议子模块不设置version，自动从parent继承version即可

> 参考 [versions-maven-plugin](https://www.mojohaus.org/versions-maven-plugin/usage.html)
> 项目demo代码  [maven-mutily-module](https://github.com/caojingui/maven-multily-module)




