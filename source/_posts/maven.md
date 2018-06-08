---
title: Maven 
date: 2018-05-08 12:52:08
tags: Maven
categories: Maven
---
# Inheritance

     <packaging>pom</packaging>

Packaging values are: pom, jar, maven-plugin, ejb, war, ear, rar, par. This defaults to jar. The packaging type required to be pom for parent and aggregation (multi-module) projects. If the packaging is pom, the goal executed will be site:attach-descriptor. Now we may add values to the parent POM, which will be inherited by its children. 
    
     <!--在子POM中继承父POM-->
     <parent>....</parent>

# Dependency Management

在项目的父层，可以通过 dependencyManagement 元素来管理 jar 包的版本，让子项目中引用一个依赖而不用显示的列出版本号。  DependencyManagement configures values for child POMs and transitive dependencies. Dependency details can be set in one central location, which will propagate to all inheriting POMs. 

父项目：

	<properties>
	    <version.framework>1.0-SNAPSHOT</version.framework>
	    <javaee-api.version>1.0-SNAPSHOT</javaee-api.version>
	</properties>
	<dependencyManagement>
	    <dependencies>
	      <dependency>
	        <groupId>com.zhisheng</groupId>
	        <artifactId>framework-cache</artifactId>
	        <version>${version.framework}</version>
	      </dependency>
	      <dependency>  
	        <groupId>javax</groupId>  
	        <artifactId>javaee-api</artifactId>  
	        <version>${javaee-api.version}</version>  
	      </dependency>  
	    </dependencies>
	</dependencyManagement>


子项目：
    
    <!--引入父POM，从父POM继承大部分属性-->  
	<parent>  
	    <artifactId>parent</artifactId>  
	    <groupId>com.zhisheng</groupId>
	    <version>0.0.1-SNAPSHOT</version>  
		<relativePath>../parent/pom.xml</relativePath>
	</parent>
	<!--依赖关系，不需要指定版本号-->  
	<dependencies>  
	    <dependency>  
	        <groupId>javax</groupId>  
	        <artifactId>javaee-api</artifactId>  
	    </dependency>  
	    <dependency>
	        <groupId>com.zhisheng</groupId>
	        <artifactId>framework-cache</artifactId>
	    </dependency>
	    <dependency>  
	        <groupId>com.fasterxml.jackson.core</groupId>  
	        <artifactId>jackson-annotations</artifactId>  
	    </dependency>  
	</dependencies>

这样做的好处：


- 统一管理项目的版本号，确保应用的各个项目的依赖和版本一致，才能保证测试的和发布的是相同的成果，因此，在顶层 pom 中定义共同的依赖关系。同时可以避免在每个使用的子项目中都声明一个版本号，这样想升级或者切换到另一个版本时，只需要在父类容器里更新，不需要任何一个子项目的修改；


- 如果某个子项目需要另外一个版本号时，只需要在 dependencies 中声明一个版本号即可。子类就会使用子类声明的版本号，覆盖父类版本号。



# Aggregation (or Multi-Module)
 A pom packaged project may aggregate the build of a set of projects by listing them as modules, which are relative paths to the directories or the POM files of those projects.

	<modules> 
		<module>my-project</module> 
		<module>another-project</module> 
		<module>third-project/pom-example.xml</module> 
	</modules>


# Inheritance v. Aggregation

Inheritance and aggregation create a nice dynamic to control builds through a single, high-level POM. You will often see projects that are both parents and aggregators. For example, the entire maven core runs through a single base POM org.apache.maven:maven, so building the Maven project can be executed by a single command: mvn compile. However, although both POM projects, an aggregator project and a parent project are not one in the same and should not be confused. A POM project may be inherited from - but does not necessarily have - any modules that it aggregates. Conversely, a POM project may aggregate projects that do not inherit from it.

# import dependencyManagement

	...
	<dependencyManagement>
	    <dependencies>
	        <dependency>
	            <groupId>other.pom.group.id</groupId>
	            <artifactId>other-pom-artifact-id</artifactId>
	            <version>SNAPSHOT</version>
	            <scope>import</scope>
	            <type>pom</type>
	        </dependency>   
	    </dependencies>
	</dependencyManagement>
	...
What then happens is that all the dependencies defined in the dependencyManagement section of the other-pom-artifact-id are included in your POM's dependencyManagement section. You can then reference these dependencies in the dependency section of your POM (and all of its child POMs) without having to include a version etc.

However if in your POM you simply define a normal dependency to other-pom-artifact-id then all dependencies from the dependency section of the other-pom-artifact-id are included transitively in your project - however the dependencies defined in the dependencyManagement section of the other-pom-artifact-id are not included at all.

So basically the two different mechanisms are used for importing/including the two different types of dependencies (managed dependencies and normal dependencies).

#####import (only available in Maven 2.0.9 or later)
This scope is only supported on a dependency of type pom in the <dependencyManagement> section. It indicates the dependency to be replaced with the effective list of dependencies in the specified POM's <dependencyManagement> section. Since they are replaced, dependencies with a scope of import do not actually participate in limiting the transitivity of a dependency.


ref.
http://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html#Dependency_Management