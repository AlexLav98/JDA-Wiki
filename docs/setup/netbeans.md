[Netbeans]: https://netbeans.org/
[CI server]: https://ci.dv8tion.net/job/JDA

# Netbeans setup
Netbeans is a IDE for Java, JavaScript, HTML5, PHP, etc. development build by [Netbeans].

!!! info "Notes"
    Replace `JDA_VERSION_HERE` with the listed version [here](../index.md).  
	There are multiple ways to setup Netbeans. Select the one that fits best for your needs.
	
	- [Setup Maven](#setup-maven)
	- [Setup from Jar](#setup-from-jar)

## Setup Maven
1. Create a new Maven Java Application.  
![step 1](/assets/img/netbeans/maven_step_1.png)
2. Open the pom.xml in the Project Files.  
![step 2](/assets/img/netbeans/maven_step_2.png)
3. Add jcenter as a repository and JDA as a dependency to maven.
```xml
<repositories>
  <repository>
    <id>jcenter</id>
	<url>https://jcenter.bintray.com</url>
  </repository>
</repositories>

<dependencies>
  <dependency>
    <groupId>net.dv8tion</groupId>
	<artifactId>JDA</artifactId>
	<version>JDA_VERSION_HERE</version>
  </dependency>
</dependencies>
```
4. Continue with [Getting started](../usage/index.md)

## Setup from Jar
1. Download the latest (binary) version of JDA (with dependencies), as well as the Javadocs from the [CI server].  
![step 1](/assets/img/netbeans/jar_step_1.png)
2. Make a new Java Application.  
![step 2](/assets/img/netbeans/jar_step_2.png)
3. Right-click the "Libraries" folder in your project and select "Add JDA/Folder...".  
![step 3](/assets/img/netbeans/jar_step_3.png)
4. Find the JDA-JDA_VERSION_HERE-with-dependencies.jar and add it.
5. Right-click on the newly-added Jar file and select "Edit...".  
![step 5](/assets/img/netbeans/jar_step_5.png)
6. Select "Browse..." next to the Javadoc section and add the JDA-JDA_VERSION_HERE-javadoc.jar  
![step 6](/assets/img/netbeans/jar_step_6.png)
7. Cpmtronue with [Getting started](../usage/index.md)