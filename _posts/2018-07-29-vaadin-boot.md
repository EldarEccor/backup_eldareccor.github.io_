---
#layout: post
title: Setting up Vaadin 10 with Spring Boot 2 and Gradle 4.9
date: 2018-07-28
categories: vaadin-boot
author: EldarEccor
---

# Vaadin vacation home

## The use case

No real motivation - we just decided to play around a bit and check out some cool stuff. The idea is to create a website for a vacation home. Ir was born after some drinks because a lot of use cases can be explored and maybe someone can use in the future...

## The toolset

Just to be sure the bigger picture is clear here is a short list of our little experiment's setup.  

 - [Spring Tool Suite 3.9.5](https://spring.io/tools/sts/all).
 - [Gradle 4.9](https://gradle.org/install)
 - [Java 8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
	- [Spring Boot 2.0.3](https://spring.io/projects/spring-boot)
	- [Vaadin 10](https://vaadin.com/start)
	- [Project Lombok](https://projectlombok.org)
 - [Github](https://www.github.com)
	- [Travis CI](https://travis-ci.org)
	- [Github Pages](https://pages.github.com)
	
## Initial Gradle setup

After importing the repository into eclipse we can start if to define our gradle project.

Based on the dependencies described above the basic setup is quite simple. We do not need a multi-project build for now. A simple gradle project structure will do.

```
.
+-- build.gradle
+-- settings.gradle
+-- src
|	+-- main
|		+-- java
|		+-- resources
|	+-- test
|		+-- java
|		+-- resources
```


For our stack to work properly we need to apply several plugins and configure them. Thanks to the fine folks at Spring and Vaadin we do not have to do much else ourselves anymore:

Vaadin defines a complete BOM for each release to prevent dependency conflicts and although gradle can't handle BOMs out of the box the spring boot plugin implicitly loads the spring dependency-management plugin. How convenient :) We base our dependencies on the Vaadin 10 BOM where possible and do not define versions on our own. The only exception for now is lombok, which is not included in the BOM. The first iteration of 'build.gradle' looks accordingly:  

**build.gradle**
```gradle
plugins {
    id 'io.franzbecker.gradle-lombok' version '1.14'
    id "org.springframework.boot" version '2.0.3.RELEASE'
}

ext {
	// Has to be 1.16.20 as 1.18 failed with gradle 4.9 & jdk8
	lombokVersion = '1.16.20'	
	vaadinVersion = '10.0.1'
}
   
apply plugin: 'java'

apply plugin: 'eclipse'	
eclipse {
	// always download sources and docs
   	classpath {
		downloadJavadoc = true
		downloadSources = true
   	}
}

apply plugin: 'org.springframework.boot'
springBoot {
	mainClassName = 'com.example.ExampleApplication'
}

apply plugin: 'io.spring.dependency-management'
dependencyManagement {
	imports {
		mavenBom "com.vaadin:vaadin-bom:${vaadinVersion}"
	}
}

apply plugin: "io.franzbecker.gradle-lombok"
lombok {
	version = lombokVersion
	// Dont check the hash
	sha256 = ""
}

repositories {
	mavenCentral()	
	maven { url "https://maven.vaadin.com/vaadin-addons" }
}

dependencies {	
	// Vaadin spring boot starter (version from BOM)
	compile("com.vaadin:vaadin-spring-boot-starter")
	
	//JUnit (version from BOM)
	testCompile("junit:junit")	
	// Spring boot test support (version from BOM)
	testCompile("org.springframework.boot:spring-boot-starter-test")
	
	// Lombok 
	compileOnly("org.projectlombok:lombok:${lombokVersion}")	
}

group = 'de.eldar'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = 1.8
```

After either using the configured eclipse plugin to generate a project or importing everything directly from STS we can now start writing some real code. For convenience we should however do one more thing first: Create the .gitignore file so eclipse or whatever git client you use won't constanly complain about all the generated files. I usually start with something minimal and add to it as I go, so for now we will use only some basic excludes. Note that for gradle projects you do not want to exclude all *.jar* files, because you will usually generate a wrapper jar (gradlew) that you *want* to commit.

**.gitignore**
```
# Eclipse
.project
.classpath
.factorypath
.settings/
bin/

# Gradle
.gradle
build/
```

Let the fun begin :)

## Hello world

By using the spring boot starter for Vaadin 10 we gain a lot of automatic configuration that will magically create a Vaadin application without making you think about the stuff. Servlets, Filters, Mappings are set up nicely without even lifting a single finger - viva la boot :) Of course it makes sense to take a look under the hood, but for now the focus is on seeing something work. Based on the conventions for Spring boot and the Annotations provided by Vaadin we can easily create our (canonical) application as well as our first Vaadin route (Routes are the new navigation concept in Vaadin 10) and some content. 

### The application class

The canonical boot application initilizes the application cotext by "pointing" at itself using the static initializer SpringApplication. Per convention Spring scans for all applicable beans and configurations in the application's own as well as all sub-packages.   

```java
/**
 * Our application class. Loads the boot magic and makes stuff happen :)
 */
@SpringBootApplication
public class VacationHomeApplication{  
 
	/**
	 * Canonical main for the spring boot application startup. 
	 * 
	 * @param args CLI args
	 */
	public static void main(String[] args) {
		SpringApplication.run(VacationHomeApplication.class, args);
	}
}
```
### First Vaadin content
Because the vaadin boot starter is on the classpath we can now simply define a so called route for our first Vaadin page. We just have to make sure to be in a sub package of our application for Spring to find us. To actually see something on the screen I included some silly vaadin components: A Button which toogles the visibility of a label:

```java
// "" is the default route for the Vaadin navigation
@Route("")
public class DefaultRoute extends VerticalLayout{

	private static final long serialVersionUID = 3412336558939266983L;
	
	private Button crazyButton;
	
	private Label crazyLabel;
	
	public DefaultRoute() {
		crazyButton = new Button("Click me softly...");
		crazyButton.addClickListener(e -> {
			crazyLabel.setVisible(!crazyLabel.isVisible());
		});
		
		crazyLabel = new Label("Jihhaaaa! Keep toggling...");
		crazyLabel.setSizeFull();
		crazyLabel.setVisible(false);
		
		setSizeFull();
		add(crazyButton);
		add(crazyLabel);
	}
}
```

Well... and... that should actually be it! 

Lets start our boot application and try it out. Because we did not define any logging configuration spring will run with its defaults, but still you should see at least the servlet mapping for vaadin in the console logs if everything went according to plan:

```
INFO 4448 --- [           main] o.a.container.JSR356AsyncSupport         : JSR 356 Mapping path /vaadinServlet
...
===========================================================
Vaadin is running in DEBUG MODE.
Add productionMode=true to web.xml to disable debug features.
===========================================================
```

When navigating to [the default boot adress](http://localhost:8080) you now see masterpiece in all its glory:

{:refdef: style="text-align: center;"}
![Breathtaking]({{ site.url }}/images/1stroute.png)
{: refdef}
