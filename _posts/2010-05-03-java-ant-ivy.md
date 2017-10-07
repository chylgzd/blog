---
layout: post
title: Ant + Ivy + Maven 配置
category: [Java,Ant,Ivy,Maven]
comments: false
---

* content
{:toc}

下载：

[https://ant.apache.org/bindownload.cgi](https://ant.apache.org/bindownload.cgi)

[https://ant.apache.org/ivy/download.cgi](https://ant.apache.org/ivy/download.cgi)

### 下载安装

```
下载解压后，设置ANT_HOME并把bin目录加入到path环境

ivy只用到其jar包ivy-2.4.0.jar，把这个包复制到ANT_HOME/lib下
```

### 修改ivy配置

```
用zip打开ivy-2.4.0.jar并点开/org/apache/ivy/core/settings目录

直接编辑ivysettings-local.xml和ivysettings-public.xml

编辑完毕后eclipse配置ant home后，直接用ant editor打开build.xml，就可以选择节点任务运行编译了
```

#### ivysettings-local.xml本地jar包目录

```xml
<ivysettings>
	<property name="ivy.local.default.root"             value="/media/yourname/program/maven/apache-ivy/local" override="false"/>
	<property name="ivy.local.default.ivy.pattern"      value="[organisation]/[module]/[type]s/[artifact]-[revision]-[revision].[ext]" override="false"/>
	<property name="ivy.local.default.artifact.pattern" value="[organisation]/[module]/[type]s/[artifact]-[revision].[ext]" override="false"/>
	<resolvers>
		<filesystem name="local">
			<ivy pattern="${ivy.local.default.root}/${ivy.local.default.ivy.pattern}" />
			<artifact pattern="${ivy.local.default.root}/${ivy.local.default.artifact.pattern}" />
		</filesystem>
	</resolvers>
</ivysettings>
```

#### ivysettings-public.xml公用maven仓库

```xml
<ivysettings>
	<property name="local-maven2-pattern" value="/media/yourname/program/maven/repository/[organisation]/[module]/[revision]/[module]-[revision]" override="false" />
	<settings defaultResolver="main" />
	<caches repositoryCacheDir="/media/yourname/program/maven/apache-ivy/cache">
		<cache name="nocache" useOrigin="true" />
		<cache name="default" />
	</caches>
	<resolvers>
		<chain name="main">
			<filesystem name="local-maven-2" m2compatible="true" local="true" cache="nocache">
				<ivy pattern="${local-maven2-pattern}.pom" />
				<artifact pattern="${local-maven2-pattern}(-[classifier]).[ext]" />
			</filesystem>
			<ibiblio name="public" m2compatible="true" root="http://maven.aliyun.com/nexus/content/groups/public/" cache="default"/>
		</chain>
	</resolvers>
</ivysettings>
```

