### 基于github搭建的个人项目仓库

#### I. 使用手册

**添加仓库**

##### 1、存在snapshot和release分支

如果要区分snapshot和release的话，如下配置

```xml
<repositories>
    <repository>
        <id>yihui-maven-repo-snap</id>
        <url>https://raw.githubusercontent.com/liuyueyi/maven-repository/snapshot/repository</url>
    </repository>
    <repository>
        <id>yihui-maven-repo-release</id>
        <url>https://raw.githubusercontent.com/liuyueyi/maven-repository/release/repository</url>
    </repository>
</repositories>
```

如果不care的话，直接添加下面的即可

```xml
<repositories>
    <repository>
        <id>yihui-maven-repo</id>
        <url>https://raw.githubusercontent.com/liuyueyi/maven-repository/master/repository</url>
    </repository>
</repositories>
```

##### 2、只有一个分支时

```xml
<repositories>
	<repository>
		<!-- A unique identifier for a repository. -->
		<id>ijiangtao-net-repo</id>
		<!--Human readable name of the repository. -->
		<name>ijiangtao.net repository</name>
		<url>https://raw.githubusercontent.com/ijiangtao/ijiangtao-net-repo/master/repository</url>
		<releases>
			<enabled>true</enabled>
			<checksumPolicy>warn</checksumPolicy>
		</releases>
		<snapshots>
			<enabled>true</enabled>
			<updatePolicy>always</updatePolicy>
			<checksumPolicy>warn</checksumPolicy>
		</snapshots>
	</repository>
</repositories>
```



**引入依赖**

按需引入即可



### II. 从零开始搭建过程

> 下面是使用两个分支的创建过程

### 1. github仓库建立

新建一个repository的前提是有github帐号，默认看到本文的是有帐号的

首先是在github上新建一个仓库，命令随意，如我新建项目为

- https://github.com/liuyueyi/maven-repository

### 2. 配置本地仓库

本地指定一个目录，新建文件夹 `maven-repository`, 如我的本地配置如下

```
## 进入目录
cd /Users/yihui/GitHub

## 新建目录
mkdir maven-repository; cd maven-repository

## 新建repository目录
# 这个目录下面就是存放我们deploy的项目相关信息
# 也就是说我们项目deploy指定的目录，就是这里
mkdir repository

## 新增一个readme文档
# 保持良好的习惯，每个项目都有一个说明文档
touch README.md
```

**这个目录结构为什么是这样的？**

我们直接看maven配置中默认的目录结构，同样拷贝一份出来而已

### 3. 仓库关联

将本地的仓库和远程的github仓库关联起来，执行的命令也比较简单了

```
git add .
git commit -m 'first comit'
git remote add origin https://github.com/liuyueyi/maven-repository.git
git push -u origin master
```

接着就是进行分支管理了

- 约定将项目中的snapshot版，deploy到仓库的 snapshot分支上
- 约定将项目中的release版，deploy到仓库的 release分支上
- master分支管理所有的版本

所以需要新创建两个分支

```
## 创建snapshot分支
git checkout -b snapshot 
git push origin snapshot
# 也可以使用 git branch snapshot , 我通常用上面哪个，创建并切换分支

## 创建release分支
git checkout -b release
git push origin release
```

### 4. 本地配置

maven工具的settings.xml文件,找到servers标签，添加一个server

```xml
<server>
    <!-- id，这只是一个标识名，根据它找到用户名和密码 -->
    <id>github</id>
    <username>guihub登录的用户名</username>
    <password>guihub登录的用户密码</password>
</server>
```

在maven项目的pom.xml中添加入下代码

```xml
<properties>
    <github.global.server>github</github.global.server>
</properties>

<plugins>
    <!--1.作用：将jar deploy(发布)到本地储存库位置(altDeploymentRepository)-->
    <plugin>
        <artifactId>maven-deploy-plugin</artifactId>
        <configuration>
            <!--设置部署目录-->
            <altDeploymentRepository>
                <!--repository 本地生成的文件夹名称-->
                internal.repo::default::file://${project.build.directory}/repository
            </altDeploymentRepository>
        </configuration>
    </plugin>
    <!--2.作用：将本地存储库位置的jar文件发布到github上-->
    <plugin>
        <groupId>com.github.github</groupId>
        <artifactId>site-maven-plugin</artifactId>
        <!--
                  这里需要使用 0.12, 0.9 部署时会出错，具体查看
                  https://github.com/github/maven-plugins/issues/105
                -->
        <version>0.12</version>
        <configuration>
            <!--git 提交的消息-->
            <message>Maven artifacts for ${project.version}</message>
            <!--禁用网页处理-->
            <noJekyll>true</noJekyll>
            <!--部署的目录，这里是和上面的 maven-deploy-plugin 的 configuration.altDeploymentRepository 对应-->
            <outputDirectory>${project.build.directory}/repository
            </outputDirectory> <!-- matches distribution management repository url above -->
            <!-- refs/heads/远程分支名,多个分支时，这里经常需要进行修改-->
            <branch>refs/heads/mvn-repo</branch>
            <includes>
                <include>**/*</include>
            </includes>
            <!--github 仓库的名字-->
            <repositoryName>mvn-repo</repositoryName>
            <!--github 用户名-->
            <repositoryOwner>liuwen</repositoryOwner>
        </configuration>
        <executions>
            <execution>
                <goals>
                    <!--suppress MybatisMapperXmlInspection -->
                    <goal>site</goal>
                </goals>
                <phase>deploy</phase>
            </execution>
        </executions>
    </plugin>
</plugins>
```



执行命令

```bash
mvn clean deploy
```

当两个maven插件结合起来使用时，执行上面命令，就不是发布到本地储存库位置，而是直接发布到github上了



此时打开github查看并刷新仓库mvn-repo，已经存在上传的jar。

上传后效果图：

![img](https://raw.githubusercontent.com/wefashe/git-images/master/images/20200710134216)