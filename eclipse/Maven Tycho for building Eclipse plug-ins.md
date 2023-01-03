# Maven Tycho for building Eclipse plug-ins
### [](#using-root-level-features)[11.1. Using root level features](#using-root-level-features)

A root level feature is a feature which can be updated or uninstalled independently of the product.

To define that a feature is a root level feature, add installMode="root" behind the feature in the product definition file. You need to use a text editor for this at the moment, the product configuration editor does currently not expose that in its user interface.

### [](#enable-the-generation-of-default-pom-files-for-eclipse-components)[11.2. Enable the generation of default pom files for Eclipse components](#enable-the-generation-of-default-pom-files-for-eclipse-components)

To enable the automatic determination of build information for your Eclipse components, add a _.mvn/extensions.xml_ descriptor to the root of your directory. This requires at **least** Apache Maven 3.3 and Maven Tycho 0.24.0, but we recommend to use the latest versions.

The contents of the _.mvn/extensions.xml_ must look like this:

```
<extensions>
  <extension>
    <groupId>org.eclipse.tycho.extras</groupId>
    <artifactId>tycho-pomless</artifactId>
    <version>2.7.4</version>
  </extension>
</extensions>
```

The following rules are used for the automatic pom generation. First of all `.qualifier` from the Eclipse components is automatically mapped to `-SNAPSHOT` for the Maven build.

Table 1. Pom Properties Mapping  
| Property | Mapping |
| --- | --- |
| 

packaging

 | 

eclipse-plugin if MANIFEST.MF is found

eclipse-feature if feature.xml is found

eclipse-test-plugin if Bundle-SymbolicName ends with .tests

 |
| 

groupId

 | 

same as in the parent pom

 |
| 

arifactId

 | 

eclipse-plugin: Bundle-SymbolicName from MANIFEST.MF

eclipse-feature: feature id from feature.xml

 |
| 

version

 | 

Bundle-Version from MANIFEST.MF or

Feature version from feature.xml

 |

### [](#defining-a-specialized-pom-file-for-an-eclipse-component)[11.3. Defining a specialized pom file for an Eclipse component](#defining-a-specialized-pom-file-for-an-eclipse-component)

If you define a pom file for an Eclipse component, you need to specify the _packaging_ attribute in the pom file.

|  | 

The build requires that the version numbers of each single Maven artifacts and its Eclipse plug-ins are in sync. Sometimes (this is rare these days) developers forget to update the Maven version numbers. In this case the build complains about a version mismatch. Run the following command to correct existing inconsistencies.

```
mvn -Dtycho.mode=maven org.eclipse.tycho:tycho-versions-plugin:update-pom
```





 |

This attribute defines what kind of Eclipse component you are building. For example, a plug-in must set this attribute to `eclipse-plugin`.

Table 2. Package attributes for Eclipse components  
| Package Attribute | Description |
| --- | --- |
| 

eclipse-plugin

 | 

Used for plug-ins

 |
| 

eclipse-test-plugin

 | 

Used for test plug-ins or fragments

 |
| 

eclipse-feature

 | 

Used for features

 |
| 

eclipse-repository

 | 

Used for p2 update sites and Eclipse products

 |
| 

eclipse-target-definition

 | 

Target definition used for the Tycho build

 |

In addition to packaging attribute, the pom of an Eclipse component must also specify the name and the version of the component. The artifact ID and version in the pom file must match the `Bundle-Symbolic-Name` and the `Bundle-Version` from the _MANIFEST.MF_ file. Eclipse components typical use `qualifier` as a suffix in the `Bundle-Version` to indicate that this should be replaced by the build system with a build qualifier. Maven uses "SNAPSHOT" for this, but Tycho maps these values correctly. Each `module` has again a `pom.xml` configuration file which defines attributes specifically to the corresponding Eclipse component, e.g., the package attribute.

It must also contain a link to the main or the configuration pom so that the build can determine the configuration.

The following listing contains an example pom file for a plug-in.

```
<project>
    <modelVersion>4.0.0</modelVersion>

    <!-- Link to the parent pom -->
    <parent>
        <artifactId>com.example.todo.build.parent</artifactId>
        <groupId>com.example.e4.rcp</groupId>
        <version>0.1.0-SNAPSHOT</version>
        <relativePath>../com.example.todo.build.parent</relativePath>
    </parent>

    <groupId>com.example.e4.rcp</groupId>
    <artifactId>com.vogella.imageloader.services</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>eclipse-plugin</packaging>
</project>
```

### [](#source-encoding)[11.4. Source Encoding](#source-encoding)

You can set the source code encoding via the `project.build.sourceEncoding` parameter.

```
<properties>
 <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
</properties>
```

|  | 

If you do not set this encoding property the maven build throws warnings similar to the following: ﻿ \[WARNING\] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!

 |

### [](#building-source-features)[11.5. Building source features](#building-source-features)

Tycho allows to generates source features which allows users to browse your code once installed in your IDE.

```
<!-- enable source feature generation -->
<build>
    <plugins>
      <!-- more entries -->

      <plugin>
        <groupId>org.eclipse.tycho</groupId>
        <artifactId>tycho-source-plugin</artifactId>
        <version>${tycho.version}</version>
        <executions>
          <execution>
            <id>plugin-source</id>
            <goals>
              <goal>plugin-source</goal>
            </goals>
          </execution>
          <execution>
            <id>feature-source</id>
            <goals>
              <goal>feature-source</goal>
            </goals>
            <configuration>
              <excludes>
                <!-- provide plug-ins not containing any source code -->
                <plugin id="com.vogella.tycho.product" />
                <plugin id="com.vogella.tycho.target" />
                <plugin id="com.vogella.tycho.update" />
                <!-- also possible to exclude feature-->
              </excludes>
            </configuration>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
```

You can add the generated source features to your existing updates side. For this, open the category file and add the features and plug-ins to one of your categories with the `.sources` suffix.

```
<?xml version="1.0" encoding="UTF-8"?>
<site>
   <feature id="com.vogella.tycho.feature" version="0.0.0">
      <category name="tychoexample"/>
   </feature>
   <feature id="com.vogella.tycho.feature.source" version="0.0.0">
      <category name="tychoexample-source"/>
   </feature>

   <category-def name="tychoexample" label="Tycho example"/>
   <category-def name="tychoexample-source" label="Tycho example source bundles"/>
</site>
```

### [](#mirroring-a-p2-update-site-with-maven-tycho)[11.6. Mirroring a p2 update site with Maven Tycho](#mirroring-a-p2-update-site-with-maven-tycho)

Tycho allows to mirror a p2 update site. This reduces the dependency to the remote side as you can use your local copy for building the Eclipse application.

The following is an example for such a mirror task which mirrors two features from the latest Eclipse release

```
<?xml version="1.0" encoding="UTF-8"?>
<project
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd"
    xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
    <modelVersion>4.0.0</modelVersion>
    <name>Mirror Update Site</name>

    <prerequisites>
        <maven>3.0</maven>
    </prerequisites>

    <groupId>com.vogella.p2.mirror</groupId>
    <artifactId>mirror</artifactId>
    <version>3.5.0-SNAPSHOT</version>
    <packaging>pom</packaging>
    <properties>
        <tycho.version>2.3.0</tycho.version>
        <tycho-extras.version>2.3.0</tycho-extras.version>
    </properties>

    <build>
    <plugins>
        <plugin>
            <groupId>org.eclipse.tycho.extras</groupId>
            <artifactId>tycho-p2-extras-plugin</artifactId>
            <version>${tycho.version}</version>
            <executions>
                <execution>
                    <phase>prepare-package</phase>
                    <goals>
                        <goal>mirror</goal>
                    </goals>
                </execution>
            </executions>

            <configuration>
                <source>
                    <!-- source repositories to mirror from -->
                    <repository>
                        <url>https://download.eclipse.org/releases/latest/</url>
                        <layout>p2</layout>
                        <!-- supported layouts are "p2-metadata", "p2-artifacts", and "p2" (for joint repositories; default) -->
                    </repository>

                </source>

                <!-- starting from here all configuration parameters are optional -->
                <!-- they are only shown here with default values for documentation purpose -->

                <!-- List of IUs to mirror. If omitted, allIUs will be mirrored. -->
                <!-- Omitted IU version element means latest version of the IU -->

                <ius>
                   <!--
                    <iu>
                        <query>
                            <expression>id == $0 &amp;&amp; version == $1</expression>
                            <parameters>org.eclipse.platform.sdk,4.19.0.I20210303-1800</parameters>
                        </query>
                    </iu>
                    -->

                    <iu>
                        <id>org.eclipse.equinox.sdk.feature.group</id>
                    </iu>

                    <iu>
                        <id>org.eclipse.e4.rcp.feature.group</id>
                    </iu>
                </ius>

                <!-- The destination directory to mirror to. -->

                <destination>${project.build.directory}/repository</destination>
                <!-- Whether only strict dependencies should be followed. -->
                <!-- "strict" means perfect version match -->

                <followStrictOnly>false</followStrictOnly>
                <!-- Whether or not to follow optional requirements. -->
                <includeOptional>true</includeOptional>
                <!-- Whether or not to follow non-greedy requirements. -->
                <includeNonGreedy>true</includeNonGreedy>

                <!-- Filter properties. E.g. filter only one platform -->

                <filter>
                    <osgi.os>linux</osgi.os>
                    <osgi.ws>gtk</osgi.ws>
                    <osgi.arch>x86_64</osgi.arch>
                </filter>

                <!-- Whether to filter the resulting set of IUs to only -->
                <!-- include the latest version of each IU -->

                <latestVersionOnly>false</latestVersionOnly>
                <!-- don't mirror artifacts, only metadata -->
                <mirrorMetadataOnly>false</mirrorMetadataOnly>

                <!-- whether to compress the content.xml/artifacts.xml -->

                <compress>true</compress>
                <!-- whether to append to the target repository content -->
                <append>true</append>

            </configuration>
        </plugin>
    </plugins>
</build>
</project>
```

You would execute this task independently of your build and use the newly created local folder in your normal build.

To use your mirror, configure Maven to use it via the _settings.xml_ file in your home folder. Alternatively you can adjust your target definition file to point to the mirror.

```
<settings>
 <mirrors>
  <mirror>
   <id>Mars_mirror</id>
   <mirrorOf>Mars</mirrorOf>
   <name>Local mirror of Mars repository</name>
   <url>file://home/vogella/Marsclone</url>
   <layout>p2</layout>
   <mirrorOfLayouts>p2</mirrorOfLayouts>
  </mirror>
 </mirrors>
</settings>
```

### [](#removing-compiler-warning-messages-from-the-build)[11.7. Removing compiler warning messages from the build](#removing-compiler-warning-messages-from-the-build)

You can remove compiler warnings from the Maven build by configuring the `tycho-compiler-plugin`. This is demonstrated with the following snippet.

```
<build>
    <plugins>
      <plugin>
        <groupId>org.eclipse.tycho</groupId>
        <artifactId>tycho-compiler-plugin</artifactId>
        <version>${tycho.version}</version>
        <configuration>
          <compilerArgs>
            <arg>-warn:-raw,unchecked</arg>
          </compilerArgs>
        </configuration>
      </plugin>
    </plugins>
 </build>
```

### [](#using-the-last-git-commit-as-build-qualifier-special-build-settings)[11.8. Using the last Git commit as build qualifier special build settings](#using-the-last-git-commit-as-build-qualifier-special-build-settings)

You can configure the `tycho-packaging-plugin` to use the last Git commit as build qualifier instead of the build time. The following pom file shows how to do this. This allows building reproducible results if no change happened in the Git repository. This only works if the project is in a Git repository.

```
<project>
 <modelVersion>4.0.0</modelVersion>
 <groupId>com.vogella.tycho.jgit.plugin</groupId>
 <artifactId>com.vogella.tycho.jgit.plugin</artifactId>
 <version>1.0.0-SNAPSHOT</version>

 <packaging>eclipse-plugin</packaging>

 <properties>
  <tycho.version>1.2.0</tycho.version>
  <tycho-extras.version>1.2.0</tycho-extras.version>
  <repo.url>https://download.eclipse.org/releases/2018-12</repo.url>
 </properties>

 <repositories>
  <repository>
   <id>eclipse-repo</id>
   <url>${repo.url}</url>
   <layout>p2</layout>
  </repository>

 </repositories>

 <build>
  <plugins>
   <plugin>
    <groupId>org.eclipse.tycho</groupId>
    <artifactId>tycho-maven-plugin</artifactId>
    <version>${tycho.version}</version>
    <extensions>true</extensions>
   </plugin>

   <plugin>
    <groupId>org.eclipse.tycho</groupId>
    <artifactId>target-platform-configuration</artifactId>
   </plugin>
  </plugins>
  <pluginManagement>
   <plugins>
    <plugin>
     <groupId>org.eclipse.tycho</groupId>
     <artifactId>tycho-packaging-plugin</artifactId>
     <version>${tycho.version}</version>
     <dependencies>
      <dependency>
       <groupId>org.eclipse.tycho.extras</groupId>
       <artifactId>tycho-buildtimestamp-jgit</artifactId>
       <version>${tycho-extras.version}</version>
      </dependency>
     </dependencies>
     <configuration>
      <timestampProvider>jgit</timestampProvider>
      <jgit.ignore>
       pom.xml
      </jgit.ignore>
      <jgit.dirtyWorkingTree>ignore</jgit.dirtyWorkingTree>
     </configuration>
    </plugin>
   </plugins>
  </pluginManagement>
 </build>

</project>
```

### [](#generating-and-updating-pom-files-for-eclipse-components)[11.9. Generating and updating POM files for Eclipse components](#generating-and-updating-pom-files-for-eclipse-components)

#### [](#generating-pom-files)[11.9.1. Generating POM files](#generating-pom-files)

Tycho supports the generation of POM files via its `tycho-pomgenerator-plugin:generate-poms` goal. This goals search from the existing directory for Eclipse components which do not yet have a pom files and generates them.

You can use the following command to generate the pom files. The `-DgroupId` allows you to specify the group ID for the pom files.

```
# group ID is set to com.vogella.tychoexample
mvn org.eclipse.tycho:tycho-pomgenerator-plugin:generate-poms \
    -DgroupId=com.vogella.tychoexample
```

This command generates pom files for all Eclipse components below the current directory and a parent pom in the current directory which includes all Eclipse components as modules.

For identifying test bundles the generator assumes that those bundles have _.test_ as suffix. ﻿ By using `-DtestSuffix=.mytestBundleSuffix` as additional parameter you can override this default.

If you try to execute this generated pom file, the build fail because you have not yet configured your dependencies.

If you want to include Eclipse components which are not in the current directory you can point Tycho to these directories via the `-DextraDirs` parameter and define from which directory it should start.

```
mvn org.eclipse.tycho:tycho-pomgenerator-plugin:generate-poms \
 -DgroupId=com.vogella.tycho.build -DbaseDir=features -DextraDirs="../../another_plugin_dir
```

### [](#setting-version-numbers)[11.11. Setting version numbers](#setting-version-numbers)

After releasing an application or several plug-ins, the version number should be increased. If you do not use pomless builds, there are two locations where version numbers are defined. On the one hand the _pom.xml_ file and on the other hand the _MANIFEST.MF_ file.

This can be done easily by using the ﻿Tycho Versions Plugin.

```
# setting the version in pom.xml and MANIFEST.MF files
mvn org.eclipse.tycho:tycho-versions-plugin:set-version -DnewVersion=X.Y.Z
    -Dtycho.mode=maven
```

Eclipse usually uses semantic versioning for plug-ins, so a version number like {major}.{minor}.{patch} is used. See [Semantic versioning](http://www.osgi.org/wiki/uploads/Links/SemanticVersioning.pdf) for more information.

|  | 

The `tycho.mode=maven` property ﻿disables the target platform calculation and dependency ﻿resolution, which is usually done by Tycho. This results in better performance, since those features are not necessary for updating the version.

 |

More information about versioning with Tycho can be found in the [Tycho version wiki](https://eclipse.org/tycho/sitedocs/tycho-release/tycho-versions-plugin/plugin-info.html).

### [](#unpack-built-plug-ins)[11.12. Unpack built plug-ins](#unpack-built-plug-ins)

In case a plug-in contains native code, e.g., dll files or others, which do not work from inside a JAR file, the plug-in itself should be unpacked.

To archive that the plug-in’s _MANIFEST.MF_ must consist of the following property.

With this property in the _MANIFEST.MF_ file the plug-in won’t be packed as JAR file, but just be available as directory in the _plugins_ directory of a product.

### [](#signing-plug-ins)[11.13. Signing plug-ins](#signing-plug-ins)

If the user installs plug-ins which are not signed, the user received a warning that the plug-in is not signed.

The Tycho build system allows to sign plug-ins and executables. You can purchase a certificate from a certification authority or create your own signature and use this for signing. If you use a custom generated certificate the user receives a warning that the certificate is not official.

For testing a self-generated certificate is sufficient. You can generate such a certificate on the command line with the `keytool` from the Java Development Kit. This requires that the corresponding directory is in your current execution path.

```
# the following goes in one line

keytool -genkey -alias   -keystore eclipse-signing.keystore \
-storepass PASSWORD -validity 4360

# this command triggers a set of questions

What is your first and last name?
  [Unknown]:  Jim Knopf
What is the name of your organizational unit?
  [Unknown]:  IT
What is the name of your organization?
  [Unknown]:  test Company
What is the name of your City or Locality?
  [Unknown]:  Hamburg
What is the name of your State or Province?
  [Unknown]:  Germany
What is the two-letter country code for this unit?
  [Unknown]:  DE
Is CN=Jim Knopf, OU=IT, O=test Company, L=Hamburg, ST=Germany, C=DE correct?
  [no]:  y

Enter key password for <selfsigned>
    (RETURN if same as keystore password):
```

To sign the plug-ins add the configuration for the `maven-jarsigner-plugin`

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jarsigner-plugin</artifactId>
    <version>1.2</version>
        <configuration>
            <keystore>file:///home/vogella/eclipse-signing.keystore</keystore>
                <storepass>voclipse</storepass>
                    <alias>selfsigned</alias>
                      <keypass>PASSWORD</keypass>
        </configuration>
        <executions>
            <execution>
                <id>sign</id>
                <goals>
                    <goal>sign</goal>
                </goals>
            </execution>
         </executions>
</plugin>
```

### [](#building-platform-specific-fragments-or-bundles)[11.14. Building platform specific fragments or bundles](#building-platform-specific-fragments-or-bundles)

As of [Bug 567782](https://bugs.eclipse.org/bugs/show_bug.cgi?id=567782) you can use pomless builds for platform specific fragments. It is also possible to specify it in the pom of the element in case you do not use pomless builds.

More infos on pom for platform specific artifacts

When building a fragment for a certain operating system, you can override the <environments> configuration from your parent POM. This requires a pom file in fragment or bundle.

```
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.vogella.tycho</groupId>
    <artifactId>com.vogella.tycho.example.linux.x86_64</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>pom</packaging>

    <parent>
        <groupId>com.vogella.tycho</groupId>
        <artifactId>org.eclipse.tycho.parent</artifactId>
        <version>1.0.0-SNAPSHOT</version>
    </parent>

    <build>
        <plugins>
            <plugin>
                <groupId>org.eclipse.tycho</groupId>
                <artifactId>target-platform-configuration</artifactId>
                <version>${tycho.version}</version>
                <configuration>
                    <environments>
                        <environment>
                            <os>linux</os>
                            <ws>gtk</ws>
                            <arch>x86_64</arch>
                        </environment>
                    </environments>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

The following entries are possible:

```
<configuration>
                    <environments>
                        <environment>
                            <os>linux</os>
                            <ws>gtk</ws>
                            <arch>x86_64</arch>
                        </environment>
                    </environments>
                </configuration>
```

```
<configuration>
                    <environments>
                        <environment>
                            <os>macosx</os>
                            <ws>cocoa</ws>
                            <arch>x86_64</arch>
                        </environment>
                    </environments>
                </configuration>
```

```
<configuration>
                    <environments>
                        <environment>
                            <os>win32</os>
                            <ws>win32</ws>
                            <arch>x86_64</arch>
                        </environment>
                    </environments>
                </configuration>
```

### [](#shell-script-for-generating-pom-files)[11.15. Shell script for generating pom files](#shell-script-for-generating-pom-files)

Sometimes it is useful to generate the pom files automatically. Here is an example shell script which creates a module entry for each directory in the current directory.

```
#!/bin/bash

cd $(dirname $0)

BASEDIR=`basename pwd`
echo "Creating pom.xml for $BASEDIR"

cat > pom.xml <<- EOM
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.vogella.tycho</groupId>
    <artifactId>$BASEDIR</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>pom</packaging>

    <parent>
        <groupId>com.vogella.tycho</groupId>
        <artifactId>com.vogella.tycho.parent</artifactId>
        <version>1.0.0-SNAPSHOT</version>
    </parent>

    <modules>
EOM

modules=`ls -l|  grep '^d'  | awk '{ print "<module>"$9"</module>" }'`

echo "$modules" >> pom.xml


cat >> pom.xml <<- EOM
    </modules>
</project>
EOM

cat pom.xml
```

### [](#building-native-components)[11.16. Building native components](#building-native-components)

Building native components, for example based on C or C++ code, is typically done via antrunner calling the native build system like make.

See the following commit for an example: