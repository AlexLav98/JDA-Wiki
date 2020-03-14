[Eclipse foundation]: https://www.eclipse.org/
[Download]: https://www.eclipse.org/downloads/
[gradle-plugin]: https://marketplace.eclipse.org/content/buildship-gradle-integration
[GitHub release]: https://github.com/DV8FromTheWorld/JDA/releases/
[Jenkins builds]: https://ci.dv8tion.net/job/JDA/

# Eclipse setup
Eclipse (Eclipse IDE) is an IDE for Java development build by the [Eclipse foundation].  
You can download it [here][download].

!!! info "Notes"
    Replace `JDA_VERSION_HERE` with the listed version [here](../index.md).  
	There are multiple ways to setup Eclipse. Select the one that fits best for your needs.
	
	- [Setup Gradle](#setup-gradle) (recommendet)  
	- [Setup Maven](#setup-maven)  
	- [Setup from Jar](#setup-from-jar)

## Setup Gradle

!!! info "Note"
    If you have *Eclipse IDE for Java Developers* installed, skip to step 2.  
	Otherwise, you need to install the *Buildship Gradle Integration plugin* first.

1. Install the *Buildship Gradle Integration* plugin.
    1. Open up Eclipse and go to the Marketplace (Located under the "Help" tab).
	2. Search for "Gradle" and install *Buildship Gradle Integration* ([Plugin page][gradle-plugin])
	3. After the plugin was installed, restart Eclipse.
2. Right-click within *Package/Project Explorer* and Select `New -> Other...`
3. In the *Gradle* folder, select "Gradle Project".
4. Write a name for your project and click on "Finish". Your setup should look similar to this:  
![step 4](/assets/img/eclipse/gradle_step_4.png)
5. Delete the classes located under `src/main/java` and `src/test/java`
6. Open the `build.gradle` and replace its content with the following code:  
```gradle
plugins {
    id'application'
    id'com.github.johnrengelman.shadow' version '4.0.4'
}

mainClassName = 'com.example.jda.Bot'

version '1.0'
def jdaVersion = 'JDA_VERSION_HERE'

sourceCompatibility = targetCompatibility = 1.8

repositories {
    jcenter()
}

dependencies {
    compile "net.dv8tion:JDA:$jdaVersion"
}

compileJava.options.encoding = 'UTF-8'
```
7. Create the package of your bot in `src/main/java`. For example: `me.name.bot`
8. Create your first Class. For example: `Bot.java`
9. Configure the `mainClassName` in your `build.gradle` to point towards your Class (`me.name.bot.Bot` in our case)
10. Save everything, right-click your project and select `Gradle -> Refresh All`
    1. All required dependencies should now be downloaded.
11. To build a jar, run `gradlew shadowJar` in a terminal at the root folder of your project.
12. Continue with [Getting started](../usage/index.md)

## Setup Maven

!!! warning "Important"
    Requires the Maven-Plugin and local Maven installation!

1. Create a new Maven Project (`File -> New -> Other -> Maven -> Maven Project`)
2. Open the `pom.xml` and add the following part right after `</description>` to make the project support UTF-8:  
```xml
<properties>
  <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  <maven.compiler.source>1.8</maven.compiler.source>
  <maven.compiler.target>1.8</maven.compiler.target>
</properties>
```
3. Add the jcenter repository for JDA to your `pom.xml` (Only add the part in `<repository></repository>` if you alread have `<repositories></repositories>`):  
```xml
<repositories>
  <repository>
    <snapshots>
      <enabled>false</enabled>
    </snapshots>
    <id>jcenter</id>
    <name>jcenter-bintray</name>
    <url>http://jcenter.bintray.com</url>
  </repository>
</repositories>
```
4. Add JDA as dependency (Only add the part in `<dependency></dependency>` if you already have `<dependencies></dependencies>`):  
```xml
<dependencies>
  <dependency>
    <groupId>net.dv8tion</groupId>
    <artifactId>JDA</artifactId>
    <version>JDA_VERSION_HERE</version>
    <type>jar</type>
    <scope>compile</scope>
  </dependency>
</dependencies>
```
5. Add the following code to setup shading for your project (Note that this will force the compiler to use Java 8!).  
```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-compiler-plugin</artifactId>
      <version>3.5.1</version>
      <configuration>
        <source>1.8</source>
        <target>1.8</target>
      </configuration>
    </plugin>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-shade-plugin</artifactId>
      <version>2.4.3</version>
      <executions>
        <execution>
          <phase>package</phase>
          <goals>
            <goal>shade</goal>
          </goals>
          <configuration>
            <artifactSet>
              <excludes>
                <exclude>example</exclude> <!-- You may add jars to exclude from shading -->
              </excludes>
            </artifactSet>
          </configuration>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
```
6. Continue with [Getting started](../usage/index.md)

## Setup from jar
1. Downloaded the latest release (Bintray) of JDA (with dependencies):
    - [GitHub release] (recommendet)
	- [Jenkins builds]
2. Create a new Java Project.
3. Fill out the Project name and set it to use Java 8 or higher.  
![step 3](/assets/img/eclipse/jar_step_3.png)
4. Right-click the Project and select "Properties".
5. Click on "Java Build Path", then click on the "Libraries" tab and finally on "Add External JARs..."  
![step 5](/assets/img/eclipse/jar_step_5.png)
6. Add the downloaded JDA jarfile and expand its properties.
    - You can skip to step 11 if you don't want Javadoc and Source annotations (Not recommendet).  
![step 6](/assets/img/eclipse/jar_step_6.png)
7. Click on "Source Attachment" followed by "Edit..." and mark "External Locations". Finally click on "External File".  
![step 7](/assets/img/eclipse/jar_step_7.png)
8. Add the JDA-JDA_VERSION_HERE-sources.jar and click OK.
9. Click on "Javadoc Location", followed by "Edit..." and then mark "Javadoc in archive" before clicking on "Browse".  
![step 9](/assets/img/eclipse/jar_step_9.png)
10. Add the JDA-JDA_VERSION_HERE-javadoc.jar and click OK.
11. Continue with [Getting started](../usage/index.md)