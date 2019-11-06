  **2019.11.6**
  * [Linux部署上的那些坑](#linux部署上的那些坑)
      * [版本环境的一致性](#版本环境的一致性)
      * [数据库部署](#数据库部署)
      * [SpringBoot项目部署心得](#springboot项目部署心得)
      * [Linux上的排查手段和检查日志](#linux上的排查手段和检查日志)
      
# Linux部署上的那些坑
由于以前开发web项目都是一些小的课程项目，所以仅仅是在本地上跑一跑，能跑的通就OK了，但真实的生产环境往往项目是部署在多台服务器上，一些数据库和缓存可能还要有分布式和集群等等，所以在远程服务器上部署项目上线是十分重要的事。我在部署到linux中，遇到了很多很多的天坑，走了不少弯路，现在记录下来，以后学习中肯定用得到。

## 版本环境的一致性
因为我是一直做的javaWeb的开发，在linux中肯定也要安装JDK，Tomcat, MySQL这样的环境和应用服务器来部署项目，为了部署成功，一定要保证linux上的tomcat的版本，JDK版本和开发环境保持一致，尤其是Tomcat，在8.5以下很多都事不支持SpringBoot的，这点一定要注意。像TOMCAT这样的，建议通过SFTP,直接将开发环境中的Webapps文件夹替换掉linux中的，避免出现路径问题。

## 数据库部署
在部署数据库的时候，流程是一定的，但还有些坑要注意，以下步骤来避坑吧！

**yum上mysql的资源有问题，所以不能仅仅之用yum。在使用yum之前还需要用其他命令获取mysql社区版**

```
cd /tmp
wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm  
rpm -ivh mysql-community-release-el7-5.noarch.rpm  
```
然后通过yum进行安装

```
yum install mysql mysql-server mysql-devel -y
```
启动检查mysql服务

```
systemctl start mysql.service
```

```
netstat -anp|grep 3306
```

设置密码：

```
mysqladmin -u root password admin
```

```
mysql -uroot -padmin
```
导入sql文件

```
mysql -u root -padmin --default-character-set=utf8 数据库名< /路径/XXX.sql
```

另一个坑是**Linux MySQL默认是大小写敏感的**，而windows下则是不敏感的，为了不出现各种恶心的错误，在mysql设置表名的时候，一定要全部小写。

## SpringBoot项目部署心得
因为SpringBoot类似一个插槽，需要什么部件都是从pom.xml里面引用就好了，像tomcat这样的应用服务器也都是由容器给我们提供好了直接用。但在linux部署中， tomcat的部署是单独的，所以把项目打包成一个war包发布到服务器上需要对项目做一些改动，我的总结如下：
**1:修改主入口函数，继承一个初始化类，并实现其要求的方法**

```
@SpringBootApplication
public class QuestioncommunityApplication extends SpringBootServletInitializer {

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return builder.sources(QuestioncommunityApplication.class);
    }
    public static void main(String[] args) {
        SpringApplication.run(QuestioncommunityApplication.class, args);
    }

}
```
**2：将原来带有的tomcat去掉，或者改成provide,有外部提供**

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-tomcat</artifactId>
        <scope>provided</scope>
    </dependency>

**3：检查pom.xml中的其他配置项，需要的也要改变其作用范围，常见的有servlet等，jpa 持久化的方式也要慎重**

    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>3.1.0</version>
        <scope>provided</scope>
    </dependency>
    
**4：注意打包的格式和版本等信息**

```
<artifactId>questioncommunity</artifactId>
<version>1.0</version>
<packaging>war</packaging>
<name>questioncommunity</name>
```

## Linux上的排查手段和检查日志
虽然已经避开了一些坑，但是部署到linux上发布还是会可能出现错误，这时需要在linux上来检查错误出现在哪里，常见命令参考如下：

**1：启动mysql和tomcat等服务，提示没有权限：**

```
chmod u+x *.sh  给所有.sh的文件格式所有权限或者chmod 777
```

**2：启动项目失败，查看tomcat的启动日志：**
进入这个目录：以。/catalina.sh -run 的形式查看启动日志，或者进入logs文件中查看
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191106182240212.png)

**3：查看主机上所有的Tomcat进程，防止出现未关闭等内存端口故障**

```
ps -ef |grep tomcat
```
**4：查看当前可用内存，（因为我买的服务器内存太小了，很多次起不来都是这个问题，要手动删掉一些）**

```
free -h
```
**5：根据 pid删去无用的进程，kill**
