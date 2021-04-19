---
title: 开发日记#1 - 通过Maven插件整合Vue工程到Spring
date: 2021-03-30 16:40:53
tags: ["SpringBoot", "Maven", "Vue"]
---

# 工程预览

![MavenWithVue](/images/20210409114257.png)

<!--more-->

# 具体配置

## 指定前端工程打包路径

在`vue.config.js`指定打包路径：

```js
"use strict";
module.exports = {
    outputDir: "../resources/static",
};
```

## Vue Router

使用默认的 Hash 模式

## Maven 清理时删除已打包的静态资源目录

```xml
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-clean-plugin</artifactId>
	<configuration>
		<filesets>
			<fileset>
				<directory>src/main/resources/static</directory>
			</fileset>
			<fileset>
				<directory>src/main/webapp/dist</directory>
			</fileset>
		</filesets>
	</configuration>
</plugin>
```

## 执行 npm 命令的 Maven 插件

```xml
<!--执行npm-->
<plugin>
	<groupId>com.github.eirslett</groupId>
	<artifactId>frontend-maven-plugin</artifactId>
	<version>1.11.3</version>
	<configuration>
		<workingDirectory>${basedir}/src/main/webapp</workingDirectory>
	</configuration>
	<executions>
		<!-- Install our node and npm version to run npm/node scripts-->
		<execution>
			<id>install node and npm</id>
			<goals>
				<goal>install-node-and-npm</goal>
			</goals>
			<configuration>
				<nodeVersion>v14.16.0</nodeVersion>
					<npmVersion>v6.14.11</npmVersion>-->
				<nodeDownloadRoot>https://npm.taobao.org/mirrors/node/</nodeDownloadRoot>
				<npmDownloadRoot>https://registry.npm.taobao.org/npm/-/</npmDownloadRoot>
			</configuration>
		</execution>
		<!-- Set NPM Registry -->
		<execution>
			<id>npm set registry</id>
			<goals>
				<goal>npm</goal>
			</goals>
			<configuration>
				<!--<arguments>config set registry https://registry.npmjs.org</arguments>-->
				<arguments>config set registry https://registry.npm.taobao.org</arguments>
			</configuration>
		</execution>
		<!-- Set SSL privilege -->
		<execution>
			<id>npm set non-strict ssl</id>
			<goals>
				<goal>npm</goal>
			</goals>
			<!-- Optional configuration which provides for running any npm command -->
			<configuration>
				<arguments>config set strict-ssl false</arguments>
			</configuration>
		</execution>
		<!-- Install all project dependencies -->
		<execution>
			<id>npm install</id>
			<goals>
				<goal>npm</goal>
			</goals>
			<!-- optional: default phase is "generate-resources" -->
			<phase>generate-resources</phase>
			<!-- Optional configuration which provides for running any npm command -->
			<configuration>
				<arguments>install</arguments>
			</configuration>
		</execution>
		<!-- Build and minify static files -->
		<execution>
			<id>npm run build</id>
			<goals>
				<goal>npm</goal>
			</goals>
			<configuration>
				<arguments>run build</arguments>
			</configuration>
		</execution>
	</executions>
</plugin>
```

## 配置 Maven 打包时对静态文件的处理

```xml
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-resources-plugin</artifactId>
	<configuration>
		<nonFilteredFileExtensions>
			<nonFilteredFileExtension>woff</nonFilteredFileExtension>
			<nonFilteredFileExtension>ttf</nonFilteredFileExtension>
			<nonFilteredFileExtension>png</nonFilteredFileExtension>
			<nonFilteredFileExtension>ico</nonFilteredFileExtension>
		</nonFilteredFileExtensions>
		<encoding>utf-8</encoding>
	</configuration>
</plugin>
```
