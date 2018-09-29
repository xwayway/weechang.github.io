---
title: 手把手教学，教你把你的个人项目推送到maven中央仓库
date: 2018-07-20 16:43:08
categories: 开源之路
tags: [手把手教学, maven中央仓库]
---

# 前言
最近闲下来了，想要自己造点小轮子用，方便自己在不同项目使用，同时想偷懒，不想每次都在项目里面copy代码。于是想到了重要仓库这个东西，把自己的代码托管到github（虽然现在的github已经不再单纯），然后发布jar包到maven中央仓库，以后每个项目都只用引入maven依赖就可以了。同时还能方便好基友们使用，万一哪一天好基友高兴就献身了呢，想想就有点小激动呢。好了废话不多说，下面正式开始手把手教学活动。
<!-- more -->
<b><a href="https://github.com/weechang/JUtil">附上小轮子传送门</a>给感兴趣的小伙伴</b>
将项目发布到maven仓库需要以下几步:
1.在sonatype中创建issue 
2.使用gpg生成秘钥 
3.配置Maven进行deploy和release

# 1 在sonatype中创建issue
首先我们需要在<a href="https://issues.sonatype.org/secure/Dashboard.jspa" targe="_blank">https://issues.sonatype.org/secure/Dashboard.jspa</a>中新建一个issues。如果没有sonatype账号的话，分分钟注册一个
{% asset_img 创建项目.png 手把手教学，教你把你的个人项目推送到maven中央仓库 %}
创建完成后就是如下图示
{% asset_img 创建完成.png 手把手教学，教你把你的个人项目推送到maven中央仓库 %}
这个时候只需要耐心等待工作人员审核就行了，因为我的domain问题，经历了二次审核，但是都很快，一般提交后两个小时内就会有结果。当issues状态变为Resolved就可以继续下一步操作了。

# 2 使用gpg生成秘钥
## 2.1 安装
选择对应的OS版本进行下载安装即可，下载地址传送门：<a href="https://www.gnupg.org/download/index.html" targe="_blank">https://www.gnupg.org/download/index.html</a>

## 2.2 生成key
首先查看安装成功没有 ```gpg --version``` （MAC 和 Linux系统需要 gpg2 --version）
通过 ```gpg --gen-key``` 生成key，也可以通过UI界面生成和管理key
{% asset_img 生成key.png 手把手教学，教你把你的个人项目推送到maven中央仓库 %}
运行后gpg要求你选择加密算法，过期时间等等，这些直接选择默认值即可。通过对比发现，gpg 2.0以上的版本运行gpg --gen-key命令 会跳过这些步骤。
之后gpg要求你输入姓名，邮箱以及关键的Passphrase，依次输入即可。然后gpg为你生成了一对秘钥。
通过```gpg --list-keys```查看生成的key列表
{% asset_img 秘钥列表.png 手把手教学，教你把你的个人项目推送到maven中央仓库 %}
这里可以看到我的公钥是：34754DFE562C10E1A09907B7F4797C9A95E36DB6，记住这个key，下面我们需要用到。

## 2.3 上传公钥
生成秘钥后，我们需要把公钥上传到服务器上。运行以下命令：```gpg2 --keyserver hkp://pool.sks-keyservers.net --send-keys 34754DFE562C10E1A09907B7F4797C9A95E36DB6（刚才生成的公钥）```

# 3 配置Maven
## 3.1 配置maven setting.xml
需要在本地的maven配置server 和 profile 两个地方，啰嗦的话就不多少了，直接上配置：
```
    <server>
        <id>ossrh</id>
        <username>第一步注册的用户名</username>
        <password>第一步注册的密码</password>
    </server>
```
```
     <profile>
         <id>ossrh</id>
         <activation>
             <activeByDefault>true</activeByDefault>
         </activation>
         <properties>
             <gpg.executable>gpg(MAC 和 Linux用户使用gpg2)</gpg.executable>
             <gpg.passphrash>生成密钥时输入的密码</gpg.passphrash>
         </properties>
    </profile>
```

## 3.2 配置项目的pom.xml
具体的配置可以查看小轮子里面的pom设置 <a href="https://github.com/weechang/JUtil/blob/master/pom.xml" target="_blank">https://github.com/weechang/JUtil/blob/master/pom.xml</a>
首先需要添加指向sonatype仓库的<distributionManagement>
```
    <distributionManagement>
        <snapshotRepository>
            <id>ossrh</id>
            <url>https://oss.sonatype.org/content/repositories/snapshots</url>
        </snapshotRepository>
        <repository>
            <id>ossrh</id>
            <name>Maven Central Staging Repository</name>
            <url>https://oss.sonatype.org/service/local/staging/deploy/maven2/</url>
        </repository>
    </distributionManagement>
```
继续配置pfofiles,添加各种推送、DOC、加密的插件
```
    <profiles>
        <profile>
            <id>release</id>
            <build>
                <plugins>
                    <plugin>
                        <groupId>org.sonatype.plugins</groupId>
                        <artifactId>nexus-staging-maven-plugin</artifactId>
                        <version>1.6.8</version>
                        <extensions>true</extensions>
                        <configuration>
                            <serverId>ossrh</serverId>
                            <nexusUrl>https://oss.sonatype.org/</nexusUrl>
                            <autoReleaseAfterClose>true</autoReleaseAfterClose>
                        </configuration>
                    </plugin>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-javadoc-plugin</artifactId>
                        <version>3.0.1</version>
                        <executions>
                            <execution>
                                <id>attach-javadocs</id>
                                <goals>
                                    <goal>jar</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-source-plugin</artifactId>
                        <executions>
                            <execution>
                                <id>attach-sources</id>
                                <goals>
                                    <goal>jar-no-fork</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-gpg-plugin</artifactId>
                        <version>1.6</version>
                        <executions>
                            <execution>
                                <id>sign-artifacts</id>
                                <phase>verify</phase>
                                <goals>
                                    <goal>sign</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>
                </plugins>
            </build>
        </profile>
    </profiles>
```
然后还可以添加一些开发者信息和license信息，具体的就不多说了。具体可以参考小轮子的配置

## 3.3 部署到中央仓库
运行以下代码进行deploy
```mvn clean deploy -P release```
如下图表示成功部署到中央仓库
{% asset_img 部署成功.png 手把手教学，教你把你的个人项目推送到maven中央仓库 %}
因为我们在pom中添加了自动发布插件，所以可以不用管理，直接到中央仓库去查看就能看到你发布的项目了
<b>PS:在部署过成功可能会遇到401的问题，具体请参考传送门 <a href="https://stackoverflow.com/questions/24830610/why-am-i-getting-a-401-unauthorized-error-in-maven" target="_blank">https://stackoverflow.com/questions/24830610/why-am-i-getting-a-401-unauthorized-error-in-maven</a></b>