# Maven依赖管理、项目构建工具

## 一、Maven私服

* 私服简介
  * Maven私服是一种特殊的Maven远程仓库，它是架设在局域网内（也可以部署在公网）的仓库服务，用来代理位于外部的远程仓库（中央仓库、其他远程公共仓库）
  * 建立了Maven私服后，当局域网内的用户需要某个构件时，会按照如下顺序进行请求和下载
    * 请求本地仓库，若本地仓库不存在所需构件，则跳转到第2步；
    * 请求Maven私服，将所需构件下载并缓存到Maven私服，若外部远程仓库不存在所需构件，则Maven直接报错
  * 此外一些无法从外部仓库下载到的构件，也能从本地上传到私服供其他人使用
    ![Maven基础03_私服0](image/Maven%E5%9F%BA%E7%A1%8003_%E7%A7%81%E6%9C%8D/1709967298017.png)  

* 私服优势
  * 节省外网带宽，下载速度更快
    * 消除对外部远程仓库的大量重复请求（会消耗很大量的带宽），降低外网带宽压力
    * Maven私服位于局域网内，从私服下载构建更快更稳定
  * 便于部署第三方构件
    * 有些构件无法从任何一个远程仓库获得（如：公司或者组织内部的私有构件、Oracle的JDBC驱动等），建立私服后，可以将这些构件部署在私服中，供内部Maven项目使用
  * 提高项目的稳定性，增强对项目的控制
    * 私服软件（如：Nexus）提供了很多控制功能，如权限管理、版本控制等，可以对仓库进行一些更加高级的控制
  * 降低中央仓库的负荷压力
    * 私服会缓存中央仓库的构件，避免了对中央仓库的重复下载

* 常见的Maven私服产品
  * Apache的Archiva
  * JFrog的Artifactory
  * Sonatype的Nexus（目前最流行、使用最广泛）

### （一）Nexus下载安装

下载地址：<https://help.sonatype.com/en/download.html>

#### 初始设置

* 打开浏览器，访问localhost:8081或者127.0.0.1:8081
* 点击右上角sign in登录，默认账户为admin，默认密码在对话框提示路径的文件中
  ![Maven基础03_私服3](image/Maven%E5%9F%BA%E7%A1%8003_%E7%A7%81%E6%9C%8D/1709969484305.png)  

* 重新设置密码
* 设置为禁止匿名访问（一般企业需求）

### （二）Nexus上的各种仓库

![Maven基础03_私服4](image/Maven%E5%9F%BA%E7%A1%8003_%E7%A7%81%E6%9C%8D/1709970116024.png)  

|仓库类型|说明|
|-|-|
|proxy|某个远程仓库的代理|
|group|存放：通过Nexus获取的第三方jar包|
|hosted|存放：本团队其他开发人员部署到Nexus的jar包|

|仓库名称|说明|
|-|-|
|maven-central|Nexus对Maven仓库的代理|
|maven-public|Maven默认创建，供开发人员下载使用的组仓库|
|maven-releases|Maven默认创建，供开发人员部署自己的jar包的宿主仓库（要求releases版本）|
|maven-snapshots|Maven默认创建，供开发人员部署自己的jar包的宿主仓库（要求snapshots版本）|

### （三）通过Nexus下载jar包

* 修改本地maven的核心配置文件settings.xml，设置新的本地仓库地址

```xml
<!--配置一个新的Maven本地仓库-->
<localRepository>D:\Environment\maven-repository-new</localRepository>
```

* 修改之前配置的阿里云仓库标签如下

```xml
<mirror>
  <id>nexus-mine</id>
  <mirrorOf>central</mirrorOf>
  <name>Nexus mine</name>
  <url>http://127.0.0.1:8081/repository/maven-central/</url>
</mirror>
```

* 若允许匿名访问，则上一步已配置完成，否则，settings.xml还需以下配置（注意：server标签内的id标签必须和mirror标签中的id一样）

```xml
<server>
  <id>nexus-mine</id>
  <username>admin</username>
  <password>重新设置的密码</password>
</server>
```

* 最后，在使用到框架的Maven工程，执行命令：

```terminal
mvn clean compile
```

* Tip：如果默认远程仓库下载速度太慢，可在nexus代理仓库设置中配置阿里云镜像仓库：<http://maven.aliyun.com/nexus/content/groups/public/>
  ![Maven基础03_私服5](image/Maven%E5%9F%BA%E7%A1%8003_%E7%A7%81%E6%9C%8D/1709979471753.png)  

### （四）将打包的jar包部署到Nexus中

* 需要在项目pom.xml中配置以下参数

  ```xml
  <distributionManagement>
    <repository>
      <id>nexus-releases</id>
      <name>Nexus Releases</name>
      <url>http://127.0.0.1:8081/repository/maven-releases/</url>
    </repository>
    <snapshotRepository>
      <!--id与maven settings中配置的server id保持一致，用于找到maven私服的用户名与密码，获取私服访问权限-->
      <id>nexus-snapshot</id>
      <!--name随意设置-->
      <name>Nexus Snapshot</name>
      <!--sanpshot仓库地址-->
      <url>http://127.0.0.1:8081/repository/maven-snapshots/</url>
    </snapshotRepository>
  </distributionManagement>
  ```

* 在maven setting.xml中添加配置

  ```xml
  <servers>
    <!-- 发布Releases版的账号，ID要与distributionManagement中的Releases ID一致 -->
    <server>
      <id>nexus-releases</id>
      <username>admin</username>
      <password>******</password>
    </server>
    <!-- 发布snapshot版的账号，ID要与distributionManagement中的snapshot ID一致 -->
    <server>
      <id>nexus-snapshot</id>
      <username>admin</username>
      <password>******</password>
    </server>
  </servers>
  ```

### （五）引用别人部署的jar包

* maven工程中的配置

  ```xml
   <repositories>
    <repository>
      <!--id与maven settings中配置的server id保持一致，用于找到maven私服的用户名与密码，获取私服访问权限-->
      <id>nexus-mine</id>
      <name>nexus-mine</name>
      <!--获取jar包的仓库-->
      <url>http://127.0.0.1:8081/repository/maven-releases/</url>
      <snapshots>
        <enabled>true</enabled>
      </snapshots>
      <releases>
        <enabled>true</enabled>
      </releases>
    </repository>
  </repositories>
  ```

## 二、Maven综合案例

参考链接：<https://www.bilibili.com/video/BV1JN411G7gX> P55、P56
