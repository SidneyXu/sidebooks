---
date: 2015-09-17
draft: false
#slug: "slug"
tags: [maven, gradle]
categories: [Tools]
title: "如何上传 Library 到 Maven 仓库"
weight: 10
---

## 注册账号

登录 [issues.sonatype.org](https://issues.sonatype.org/secure/Dashboard.jspa) 注册账号

创建一个 Issue，申请发布权限
<!--more-->

![](http://7xlqqp.com1.z0.glb.clouddn.com/create_issue.png)

![](http://7xlqqp.com1.z0.glb.clouddn.com/create_issue2.png)

创建完 issue 后，等候 1-2 天就会接收到 sonatype 发来的邮件表示审核通过

![](http://7xlqqp.com1.z0.glb.clouddn.com/mail.png)

## 编写 Gradle 发布脚本

在项目根目录创建 `mvn_publish.gradle` 文件，内容如下

``` gradle
apply plugin: 'maven'
apply plugin: 'signing'

def isReleaseBuild = !VERSION_NAME.endsWith('-SNAPSHOT')

def getReleaseRepositoryUrl() {
    return hasProperty('RELEASE_REPOSITORY_URL') ? RELEASE_REPOSITORY_URL
            : "https://oss.sonatype.org/service/local/staging/deploy/maven2/"
}

def getSnapshotRepositoryUrl() {
    return hasProperty('SNAPSHOT_REPOSITORY_URL') ? SNAPSHOT_REPOSITORY_URL
            : "https://oss.sonatype.org/content/repositories/snapshots/"
}

def getRepositoryUsername() {
    return hasProperty('NEXUS_USERNAME') ? NEXUS_USERNAME : ""
}

def getRepositoryPassword() {
    return hasProperty('NEXUS_PASSWORD') ? NEXUS_PASSWORD : ""
}

afterEvaluate { project ->
    uploadArchives {

        repositories {
//            flatDir {
//                dirs "file://${localReleaseDest}"
//            }

            mavenDeployer {
                beforeDeployment {
                    MavenDeployment deployment -> signing.signPom(deployment)
                }

//            repository(url: "file://${localReleaseDest}")

                repository(url: getReleaseRepositoryUrl()) {
                    authentication(userName: getRepositoryUsername(), password: getRepositoryPassword())
                }
                snapshotRepository(url: getSnapshotRepositoryUrl()) {
                    authentication(userName: getRepositoryUsername(), password: getRepositoryPassword())
                }

                pom.groupId = POM_GROUP_ID
                pom.artifactId = POM_ARTIFACT_ID
                pom.version = VERSION_NAME

                pom.project {
                    name POM_NAME
                    packaging POM_PACKAGING
                    description POM_DESCRIPTION
                    url POM_URL
                    inceptionYear POM_INCEPTION_YEAR

                    scm {
                        url POM_SCM_URL
                        connection POM_SCM_CONNECTION
                        developerConnection POM_SCM_DEV_CONNECTION
                    }

                    licenses {
                        license {
                            name POM_LICENCE_NAME
                            url POM_LICENCE_URL
                            distribution POM_LICENCE_DIST
                        }
                    }

                    developers {
                        developer {
                            id POM_DEVELOPER_ID
                            name POM_DEVELOPER_NAME
                        }
                    }
                }

            }
        }
    }

    signing {
        required { isReleaseBuild && gradle.taskGraph.hasTask("uploadArchives") }
        sign configurations.archives
    }

}
```

接着在工程根目录创建 `gradle.properties`，填写如下内容，各属性需要修改成自己的，其中 VERSION_NAME 以 “-SNAPSHOT” 结尾的话就会被发布到 “SNAPSHOT” 仓库，否则的话会发布到 “RELEASE” 仓库。

``` 
POM_GROUP_ID=yourGroupId
VERSION_NAME=0.0.1-SNAPSHOT
VERSION_CODE=1

POM_DEVELOPER_ID=sidneyxu
POM_DEVELOPER_NAME=Sidney Xu
POM_INCEPTION_YEAR=2015

POM_LICENCE_NAME=The Apache Software License, Version 2.0
POM_LICENCE_URL=http://www.apache.org/licenses/LICENSE-2.0.txt
POM_LICENCE_DIST=repo

SNAPSHOT_REPOSITORY_URL=https://oss.sonatype.org/content/repositories/snapshots
RELEASE_REPOSITORY_URL=https://oss.sonatype.org/service/local/staging/deploy/maven2
```

## 编写模块的发布脚本

接着在需要发布的模块下创建 `gradle.properties` 文件，填入以下内容，各属性需要修改成自己的。

``` 
POM_NAME=SauceCoreLibrary
POM_ARTIFACT_ID=sauce-core
POM_PACKAGING=jar   # or aar

POM_DESCRIPTION=SauceCoreLibrary is a core lib of sauce. Other libraries must base on it.
POM_URL=https://github.com/SidneyXu/sauce
POM_SCM_URL=https://github.com/SidneyXu/sauce
POM_SCM_CONNECTION=scm:git@github.com:SidneyXu/sauce.git
POM_SCM_DEV_CONNECTION=scm:git@github.com:SidneyXu/sauce.git
```

如果需要发布的为 Java Library，则修改该模块下的 `build.gradle` 文件，添加如下代码

``` gradle
// 发布到本地目录，非必须
ext.localReleaseDest = "${buildDir}/release/${VERSION_NAME}"

task javadocs(type: Javadoc) {
    source = sourceSets.main.allJava
    options.encoding = "UTF-8"
}

task javadocsJar(type: Jar, dependsOn: javadoc) {
    classifier = 'javadoc'
    from javadoc.destinationDir
}

task sourcesJar(type: Jar) {
    classifier = 'sources'
    from sourceSets.main.allJava
}

artifacts {
    archives javadocsJar
    archives sourcesJar
}

apply from: '../mvn_publish.gradle'
```

如果需要发布的为 Android Library，则修改为如下内容

``` gradle
// 发布到本地目录，非必须
ext.localReleaseDest = "${buildDir}/release/${VERSION_NAME}"

task androidJavadocs(type: Javadoc) {
    source = android.sourceSets.main.java.srcDirs
    classpath += project.files(android.getBootClasspath().join(File.pathSeparator))
}

task androidJavadocsJar(type: Jar, dependsOn: androidJavadocs) {
    classifier = 'javadoc'
    from androidJavadocs.destinationDir
}

task androidSourcesJar(type: Jar) {
    classifier = 'sources'
    from android.sourceSets.main.java.sourceFiles
}

artifacts {
    archives androidSourcesJar
    archives androidJavadocsJar
}
```

## 下载 GPGTools，创建 Signing Key

Windows 下载地址为

[GPG](https://www.gnupg.org/download/index.html)

[GPG for Windows](http://gpg4win.org/)

Mac 版本下载地址为

[GPG Tools](https://gpgtools.org/)

安装完后可以执行以下命令验证是否安装成功

``` bash
gpg --version
```

之后可以使用对应平台的 GUI 工具创建 Signing Key，也可以通过命令行创建。GUI 创建异常简单，所以以下只介绍命令行方式。

创建 Key

``` bash
gpg --gen-key
```

执行完成后根据提示一步步创建 Key

``` bash
Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
Your selection? 1
RSA keys may be between 1024 and 8192 bits long.
What keysize do you want? (2048)
Requested keysize is 2048 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 0
Key does not expire at all
Is this correct? (y/N) y

GnuPG needs to construct a user ID to identify your key.

Real name: Sidney
Email address: sidney@test.com
Comment: This is just a testing key.
You selected this USER-ID:
    "Sidney (This is just a testing key.) <sidney@test.com>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O
You need a Passphrase to protect your secret key.
```

创建完的 Signing Key 分为 public key 和 secret key

查看创建的公钥 public key

``` bash
gpg --list-keys
```

输出内容中的 "pub   xxxx/yyyyyyyy" 即为创建的公钥记录，其中的 "yyyyyyyy" 8位16进制数为对应的 KEY ID，之后需要配置在 `~/.gradle/gradle.properties` 中。

查看创建的密钥 serect key

``` bash
gpg --list-secret-keys
```

输出内容的第一行 ".../gnupg/secring.gpg" 为密钥路径，之后需要填写到 `~/.gradle/gradle.properties`文件中。

首先需要上传 public key 到 keyservers.net

``` bash
gpg --keyserver hkp://pool.sks-keyservers.net --send-keys <YOUR KEY ID>
```

然后将 Signing Key 的信息填写到 `~/.gradle/gradle.properties` 中

``` 
signing.keyId=<Your Key Id>
signing.password=<Your Key Password>
signing.secretKeyRingFile=<Your Secret GPG File Location>
NEXUS_USERNAME=<Sonatype Username>
NEXUS_PASSWORD=<Sonatype Password>
```

## 执行发布脚本

定位到需要发布的模块目录下执行以下语句就可以完成发布

``` bash
../gradlew upload
```

如果发布的是 SNAPSHOT 版本，执行到这一步就可以了，之后可以到 [snaphost](https://oss.sonatype.org/content/repositories/snapshots) 查看是否上传成功。

## 发布 Release 版本

SNAPSHOT 版本只要执行以上操作就行了，但如果发布的是 RLEASE 版本的话则还需要进行以下操作。

首先需要登陆 [oss.sonatype.org](https://oss.sonatype.org)，选择 `Staging Repositories`。

接着在搜索框输入你的 "group id" ，之前上传成功的话可以看到 `repository` 一栏显示的是你 `groupid-1000`，之后你每上传一次该数字都会递增。该 `repository` 为临时仓库，并不是 Maven 的中央仓库，所以还需要进行同步操作。

先确认该仓库的 `Status` 为 `open` 状态，接着选择 `Content` 选项卡确认上传的文件是否如预想一样。都确认好后可以点击 `Close` 按钮。

![](http://7xlqqp.com1.z0.glb.clouddn.com/release.png)

然后回到最开始创建的 Issue 处，添加一条评论，申请开启到 Maven 仓库的同步服务。

![](http://7xlqqp.com1.z0.glb.clouddn.com/add_comment.png)

等待几个小时后会收到工作人员的邮件，然后会发现原来的 `Release` 按钮变成了可用状态，点击后就完成了发布操作。接着在回到 Issue 处添加以下评论等待几个小时就可以在 Maven 中央仓库看到你的劳动成果了。



## 参考资料

- [OSSRH Guide](http://central.sonatype.org/pages/ossrh-guide.html)
- [Gradle](http://central.sonatype.org/pages/gradle.html)
