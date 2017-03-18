---
title: Android快速发布项目到jcenter
date: 2017-03-18 13:36:01
tags: Android Jcenter
---
1.新建[Bintray帐号](https://bintray.com/signup/oss)
  注意:邮箱不能使用qq邮箱和163邮箱

2.新建Repositories, name:maven type:Maven
<!-- more -->
![Markdown](http://p1.bqimg.com/587822/5dd605e1162a4043s.png)

3.local.properties中添加
```
bintray.user=andwong001 //bintray用户名
bintray.apikey=xxxxxxxxxxxxxxxx //apikey
```
apikey的获取 : edit profile -> api key
![Markdown](http://p1.bpimg.com/587822/3b7057888acb55ffs.png)

4.项目根目录build.gradle中添加:
```
classpath 'com.github.dcendents:android-maven-gradle-plugin:1.5'
classpath 'com.jfrog.bintray.gradle:gradle-bintray-plugin:1.2'
```
完整代码如下:
```
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.2.2'
        //添加,后期版本号可能会过期,注意修改
        classpath 'com.github.dcendents:android-maven-gradle-plugin:1.5'
        classpath 'com.jfrog.bintray.gradle:gradle-bintray-plugin:1.2'
        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}
allprojects {
    repositories {
        jcenter()
    }
}
task clean(type: Delete) {
    delete rootProject.buildDir
}
```
5.moudle目录下build.gradle中添加 :
```
apply plugin: 'com.android.library'
android {
    compileSdkVersion 23
    buildToolsVersion "23.0.2"
    resourcePrefix "badger"	//这个随便填
    defaultConfig {
        minSdkVersion 14
        targetSdkVersion 23
        versionCode 1
        versionName "1.0"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.android.support:appcompat-v7:23.2.0'
}
//添加如下代码
apply plugin: 'com.github.dcendents.android-maven'
apply plugin: 'com.jfrog.bintray'
version = "1.0.0" //版本号
def siteUrl = 'https://github.com/AndWong/BadgerHelper'      // 项目的主页
def gitUrl = 'https://github.com/AndWong/BadgerHelper.git'   // Git仓库的url
group = "com.tool.badger"  //一般填你唯一的包名
install {
    repositories.mavenInstaller {
        // This generates POM.xml with proper parameters
        pom {
            project {
                packaging 'aar'
                // Add your description here
                name 'Android Util For Badger' 	//项目描述
                url siteUrl
                // Set your license
                licenses {
                    license {
                        name 'The Apache Software License, Version 2.0'
                        url 'http://www.apache.org/licenses/LICENSE-2.0.txt'
                    }
                }
                developers {
                    developer {
                        id 'wong' //填写的一些基本信息
                        name 'andwong001'
                        email 'wonglife@yahoo.com'
                    }
                }
                scm {
                    connection gitUrl
                    developerConnection gitUrl
                    url siteUrl
                }
            }
        }
    }
}
task sourcesJar(type: Jar) {
    from android.sourceSets.main.java.srcDirs
    classifier = 'sources'
}
task javadoc(type: Javadoc) {
    source = android.sourceSets.main.java.srcDirs
    classpath += project.files(android.getBootClasspath().join(File.pathSeparator))
}
task javadocJar(type: Jar, dependsOn: javadoc) {
    classifier = 'javadoc'
    from javadoc.destinationDir
}
//这个很重要
artifacts {
    archives javadocJar
    archives sourcesJar
}
Properties properties = new Properties()
properties.load(project.rootProject.file('local.properties').newDataInputStream())
bintray {
    user = properties.getProperty("bintray.user")
    key = properties.getProperty("bintray.apikey")
    configurations = ['archives']
    pkg {
        repo = "maven"  //
        name = "Badger"	//发布到JCenter上的项目名字
        websiteUrl = siteUrl
        vcsUrl = gitUrl
        licenses = ["Apache-2.0"]
        publish = true
    }
}
```
6.Rebuild项目
并在项目根目录下执行:
```
$./gradlew install
$./gradlew bintrayUpload
```

7.完成后到Bintray网站的该项目点击 add to jCenter,等待审核
选择上传的项目
![Markdown](http://p1.bpimg.com/587822/197c85eeaf18de6ds.png)
打开后,会有一个 [Add to Jcenter]按钮,点击提交等待审核(一般1~2天就行)
下图是审核过的项目
![Markdown](http://p1.bpimg.com/587822/8988ce7808992257s.png)

项目地址 : https://github.com/AndWong/BadgerHelper
请多多指教,多多start!
