---
layout: post
title: Gradle 相关
category: [Java,Maven]
comments: false
---

* content
{:toc}

### eclipse中gradle多模块配置父build.gradle

```
group 'my.test.group'
version '0.0.1'
apply plugin: 'java'
apply plugin: 'idea'
apply plugin: 'eclipse'

ext {
    //公共参数或版本号属性自定义
    JAVA_VERSION = '1.8'
    xx2 = 'xx2'
}

buildscript {
    repositories {
        mavenLocal()
        maven {
            url 'https://maven.aliyun.com/repository/public'
        }
    }
}

subprojects {
	signing {
  	sign publishing.publications.mavenJava
  }
  eclipse {
    classpath {
      defaultOutputDir = file('target/classes')
      file {
        whenMerged {
        	entries.findAll {
          	it.path == 'src/main/java' || it.path == 'src/main/resources'
          }.each { 
          	it.output = "target/classes"
          }
          entries.findAll {
          	it.path == 'src/test/java' || it.path == 'src/test/resources'
          }.each { 
          	it.output = "target/test-classes"
          }
      	}
      }
    }
  }
	repositories {
  	mavenLocal()
    maven {
    	url 'https://maven.aliyun.com/repository/public'
    }
    maven {
    	url 'https://oss.sonatype.org/content/repositories/snapshots'
    }
	}
	apply plugin: 'java'
  apply plugin: 'maven-publish'
  apply plugin: 'signing'
  group rootProject.group
  sourceCompatibility = ${JAVA_VERSION}
  
  compileJava {
    sourceCompatibility = 1.8
    targetCompatibility = 1.8
    [compileJava]*.options*.encoding = 'UTF-8'
  }

  compileTestJava {
    sourceCompatibility = 1.8
    targetCompatibility = 1.8
    [compileTestJava]*.options*.encoding = 'UTF-8'
  }

  task sourcesJar(type: Jar) {
    from sourceSets.main.allJava
    classifier = 'sources'
  }
  
}
```