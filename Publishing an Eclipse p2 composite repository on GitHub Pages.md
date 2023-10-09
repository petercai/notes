# Publishing an Eclipse p2 composite repository on GitHub Pages 
I had already described the process of publishing an Eclipse p2 composite update site:

*   [Publish an Eclipse p2 composite repository on Bintray](https://www.lorenzobettini.it/2016/02/publish-an-eclipse-p2-composite-repository-on-bintray/)
*   [Publish an Eclipse p2 repository on Sourceforge with rsync](https://www.lorenzobettini.it/2015/01/publish-an-eclipse-p2-repository-on-sourceforge-with-rsync/)

Well, now that [Bintray is shutting down](https://jfrog.com/blog/into-the-sunset-bintray-jcenter-gocenter-and-chartcenter/), and Sourceforge is quite slow in serving an Eclipse update site, I decided to publish my Eclipse p2 composite update sites on [GitHub Pages](https://pages.github.com/).

GitHub Pages might not be ideal for serving binaries, and it has a [few limitations](https://docs.github.com/en/github/working-with-github-pages/about-github-pages#guidelines-for-using-github-pages). However, such limitations (e.g., published sites may be no larger than 1 GB, sites have a soft bandwidth limit of 100GB per month and sites have a soft limit of 10 builds per hour) are not that crucial for an Eclipse update site, whose artifacts are not that huge. Moreover, at least my projects are not going to serve more than 100GB per month, unfortunately, I might say 😉

In this tutorial, I’ll show how to do that, so that you can easily apply this procedure also to your projects!

The procedure is part of the Maven/Tycho build so that it is fully automated. Moreover, the pom.xml and the ant files can be fully reused in your own projects (just a few properties have to be adapted). **The idea is that you can run this Maven build (basically, “mvn deploy”) on any CI server** (as long as you have write-access to the GitHub repository hosting the update site – more on that later). **Thus, you will not depend on the pipeline syntax of a specific CI server** (Travis, GitHub Actions, Jenkins, etc.), though, depending on the specific CI server you might have to adjust a few minimal things.

These are the main points:

*   The Eclipse project is hosted on a GitHub repository. The example used in this tutorial can be found at [https://github.com/LorenzoBettini/p2composite-github-pages-example](https://github.com/LorenzoBettini/p2composite-github-pages-example).
*   The corresponding composite update site is hosted on another GitHub repository; in such a repository, the [publishing source for GitHub Pages is configured to be based on the _master_ branch](https://docs.github.com/en/github/working-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site), instead of the default _gh-pages_ branch. The GitHub repository for the composite update site for this example can be found at [https://github.com/LorenzoBettini/p2composite-github-pages-example-updates](https://github.com/LorenzoBettini/p2composite-github-pages-example-updates) and can be used as an Eclipse update site by using the URL [https://lorenzobettini.github.io/p2composite-github-pages-example-updates](https://lorenzobettini.github.io/p2composite-github-pages-example-updates), following the standard GitHub Pages convention. You must first create this GitHub repository and initialize it with at least one commit. You can configure the source for GitHub pages even afterward, at most, before announcing your update site.

The p2 children repositories and the p2 composite repositories will be published with standard Git operations since we publish them in a GitHub repository.

Let’s recap what p2 composite update sites are. Quoting from [https://wiki.eclipse.org/Equinox/p2/Composite\_Repositories\_(new)](https://wiki.eclipse.org/Equinox/p2/Composite_Repositories_(new))

> As repositories continually grow in size they become harder to manage. The goal of composite repositories is to make this task easier by allowing you to have a parent repository which refers to multiple children. Users are then able to reference the parent repository and the children’s content will transparently be available to them.

In order to achieve this, all published p2 repositories must be available, each one with its own p2 metadata that should never be overwritten. On the contrary, the metadata that we will overwrite will be the one for the composite metadata, i.e., _compositeContent.xml_ and _compositeArtifacts.xml_.

Directory Structure
-------------------

I want to be able to serve these composite update sites:

*   the main one collects all the versions
*   a composite update site for each major version (e.g., 1.x, 2.x, etc.)
*   a composite update site for each major.minor version (e.g., 1.0.x, 1.1.x, 2.0.x, etc.)

What I aim at is to have the following paths:

*   **releases**: in this directory, all p2 simple repositories will be uploaded, each one in its own directory, named after version.buildQualifier, e.g., 1.0.0.v20210307-2037, 1.1.0.v20210307-2104, etc. Your Eclipse users can then use the URL of one of these single update sites to stick to that specific version.
*   **updates**: in this directory, the metadata for major and major.minor composite sites will be uploaded.
*   **root**: the main composite update site collecting all versions.

To summarize, we’ll end up with a remote directory structure like the following one

| 

1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

17

18

19

20

21

22

23

24

25

26

27

28

29

30

31

32

33

34



 | 

├──  compositeArtifacts.xml

├──  compositeContent.xml

├──  p2.index

├──  README.md

├──  releases

│    ├──  1.0.0.v20210307-2037

│    │    ├──  artifacts.jar

│    │    ├──  ...

│    │    ├──  features  ...

│    │    └──  plugins  ...

│    ├──  1.0.0.v20210307-2046  ...

│    ├──  1.1.0.v20210307-2104  ...

│    └──  2.0.0.v20210308-1304  ...

└──  updates

├──  1.x

│    ├──  1.0.x

│    │    ├──  compositeArtifacts.xml

│    │    ├──  compositeContent.xml

│    │    └──  p2.index

│    ├──  1.1.x

│    │    ├──  compositeArtifacts.xml

│    │    ├──  compositeContent.xml

│    │    └──  p2.index

│    ├──  compositeArtifacts.xml

│    ├──  compositeContent.xml

│    └──  p2.index

└──  2.x

├──  2.0.x

│    ├──  compositeArtifacts.xml

│    ├──  compositeContent.xml

│    └──  p2.index

├──  compositeArtifacts.xml

├──  compositeContent.xml

└──  p2.index



 |

Thus, if you want, you can provide these sites to your users (I’m using the URLs that correspond to my example):

*   **https://lorenzobettini.github.io/p2composite-github-pages-example-updates** for the main global update site: every new version will be available when using this site;
*   **https://lorenzobettini.github.io/p2composite-github-pages-example-updates/updates/1.x** for all the releases with major version 1: for example, the user won’t see new releases with major version 2;
*   **https://lorenzobettini.github.io/p2composite-github-pages-example-updates/updates/1.x/1.0.x** for all the releases with major version 1 and minor version 0: the user will only see new releases of the shape 1.0.0, 1.0.1, 1.0.2, etc., but NOT 1.1.0, 1.2.3, 2.0.0, etc.

If you want to change this structure, you have to carefully tweak the ant file we’ll see in a minute.

Building Steps
--------------

During the build, before the actual deployment, we’ll have to update the composite site metadata, and we’ll have to do that locally.

The steps that we’ll perform during the Maven/Tycho build are:

*   Clone the repository hosting the composite update site (in this example, [https://github.com/LorenzoBettini/p2composite-github-pages-example-updates](https://github.com/LorenzoBettini/p2composite-github-pages-example-updates));
*   Create the p2 repository (with Tycho, as usual);
*   Copy the p2 repository in the cloned repository in a subdirectory of the _releases_ directory (the name of the subdirectory has the same qualified version of the project, e.g., 1.0.0.v20210307-2037);
*   Update the composite update sites information in the cloned repository (using the p2 tools);
*   Commit and push the updated clone to the remote GitHub repository (the one hosting the composite update site).

First of all, in the parent POM, we define the following properties, which of course you need to tweak for your own projects:

|  | 

<!\-\- Required properties for releasing -->

<github-update-repo>git@github.com:LorenzoBettini/p2composite-github-pages-example-updates.git</github-update-repo>

<github-local-clone>${project.build.directory}/checkout</github-local-clone>

<releases-directory>${github-local-clone}/releases</releases-directory>

<current-release-directory>${releases-directory}/${qualifiedVersion}</current-release-directory>

<!\-\- The label for the Composite sites -->

<site.label>Composite Site Example</site.label>



 |

It should be clear which properties you need to modify for your project. In particular, the _github-update-repo_ is the URL (with authentication information) of the GitHub repository hosting the composite update site, and the _site.label_ is the label that will be put in the composite metadata.

Then, in the parent POM, we configure in the _pluginManagement_ section all the versions of the plugin we are going to use (see the sources of the example on GitHub).

The most interesting configuration is the one for the _tycho-packaging-plugin_, where we specify the format of the qualified version:

|  | 

<plugin>

<groupId>org.eclipse.tycho</groupId>

<artifactId>tycho-packaging-plugin</artifactId>

<version>${tycho-version}</version>

<configuration>

<format>'v'yyyyMMdd'-'HHmm</format>

</configuration>

</plugin>



 |

Moreover, we create a profile **release-composite** (which we’ll also use later in the POM of the _site_ project), where we disable the standard Maven plugins for _install_ and _deploy_. Since we are going to release our Eclipse p2 composite update site during the deploy phase, but we are not interested in installing and deploying the Maven artifacts, we skip the standard Maven plugins bound to those phases:

| 

1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

17

18

19

20

21

22

23

24

25



 | 

<profile>

<!\-\- Activate this profile to perform the release to GitHub Pages -->

<id>release-composite</id>

<build>

<pluginManagement>

<plugins>

<plugin>

<artifactId>maven-install-plugin</artifactId>

<executions>

<execution>

<id>default-install</id>

<phase>none</phase>

</execution>

</executions>

</plugin>

<plugin>

<artifactId>maven-deploy-plugin</artifactId>

<configuration>

<skip>true</skip>

</configuration>

</plugin>

</plugins>

</pluginManagement>

</build>

</profile>



 |

The interesting steps are in the _site_ project, the one with _<packaging>eclipse-repository</packaging>_. Here we also define the profile **release-composite** and we use a few plugins to perform the steps involving the Git repository described above (remember that these configurations are inside the profile release-composite, of course in the _build_ _plugins_ section):

| 

1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

17

18

19

20

21

22

23

24

25

26

27

28

29

30

31

32

33

34

35

36

37

38

39

40

41

42

43

44

45

46

47

48

49

50

51

52

53

54

55

56

57

58

59

60

61

62

63

64

65

66

67

68

69

70

71

72

73

74

75

76

77

78

79

80

81

82

83

84

85

86

87

88

89

90

91

92

93

94

95

96

97

98

99

100

101

102

103

104

105

106

107

108

109

110

111

112

113

114

115

116

117

118

119

120

121

122

123

124

125

126

127

128

129

130

131

132

133

134

135

136

137

138

139

140

141

142

143

144

145

146

147

148

149

150

151

152

153

154

155

156

157

158

159

160

161

162

163

164

165

166

167

168

169

170

171

172

173

174



 | 

<plugin>

<groupId>org.codehaus.mojo</groupId>

<artifactId>build-helper-maven-plugin</artifactId>

<executions>

<!\-\- sets the following properties that we use in our Ant scripts

      parsedVersion.majorVersion

      parsedVersion.minorVersion

      bound by default to the validate phase

    -->

<execution>

<id>parse-version</id>

<goals>

<goal>parse-version</goal>

</goals>

</execution>

</executions>

</plugin>

<plugin>

<groupId>org.codehaus.mojo</groupId>

<artifactId>exec-maven-plugin</artifactId>

<configuration>

<executable>git</executable>

</configuration>

<executions>

<execution>

<id>git-clone</id>

<phase>prepare-package</phase>

<goals>

<goal>exec</goal>

</goals>

<configuration>

<arguments>

<argument>clone</argument>

<argument>--depth=1</argument>

<argument>-b</argument>

<argument>master</argument>

<argument>${github-update-repo}</argument>

<argument>${github-local-clone}</argument>

</arguments>

</configuration>

</execution>

<execution>

<id>git-add</id>

<phase>verify</phase>

<goals>

<goal>exec</goal>

</goals>

<configuration>

<arguments>

<argument>-C</argument>

<argument>${github-local-clone}</argument>

<argument>add</argument>

<argument>-A</argument>

</arguments>

</configuration>

</execution>

<execution>

<id>git-commit</id>

<phase>verify</phase>

<goals>

<goal>exec</goal>

</goals>

<configuration>

<arguments>

<argument>-C</argument>

<argument>${github-local-clone}</argument>

<argument>commit</argument>

<argument>-m</argument>

<argument>Release ${qualifiedVersion}</argument>

</arguments>

</configuration>

</execution>

<execution>

<id>git-push</id>

<phase>deploy</phase>

<goals>

<goal>exec</goal>

</goals>

<configuration>

<arguments>

<argument>-C</argument>

<argument>${github-local-clone}</argument>

<argument>push</argument>

<argument>origin</argument>

<argument>master</argument>

</arguments>

</configuration>

</execution>

</executions>

</plugin>

<plugin>

<artifactId>maven-resources-plugin</artifactId>

<executions>

<execution>

<id>copy-repository</id>

<phase>package</phase>

<goals>

<goal>copy-resources</goal>

</goals>

<configuration>

<outputDirectory>${current-release-directory}</outputDirectory>

<resources>

<resource>

<directory>${project.build.directory}/repository</directory>

</resource>

</resources>

</configuration>

</execution>

</executions>

</plugin>

<plugin>

<groupId>org.eclipse.tycho.extras</groupId>

<artifactId>tycho-eclipserun-plugin</artifactId>

<configuration>

<repositories>

<repository>

<id>2020-12</id>

<layout>p2</layout>

<url>https://download.eclipse.org/releases/2020-12</url>

</repository>

</repositories>

<dependencies>

<dependency>

<artifactId>org.eclipse.ant.core</artifactId>

<type>eclipse-plugin</type>

</dependency>

<dependency>

<artifactId>org.apache.ant</artifactId>

<type>eclipse-plugin</type>

</dependency>

<dependency>

<artifactId>org.eclipse.equinox.p2.repository.tools</artifactId>

<type>eclipse-plugin</type>

</dependency>

<dependency>

<artifactId>org.eclipse.equinox.p2.core.feature</artifactId>

<type>eclipse-feature</type>

</dependency>

<dependency>

<artifactId>org.eclipse.equinox.p2.extras.feature</artifactId>

<type>eclipse-feature</type>

</dependency>

<dependency>

<artifactId>org.eclipse.equinox.ds</artifactId>

<type>eclipse-plugin</type>

</dependency>

</dependencies>

</configuration>

<executions>

<!\-\- add our new child repository -->

<execution>

<id>add-p2-composite-repository</id>

<phase>package</phase>

<goals>

<goal>eclipse-run</goal>

</goals>

<configuration>

<applicationsArgs>

<args>-application</args>

<args>org.eclipse.ant.core.antRunner</args>

<args>-buildfile</args>

<args>packaging-p2composite.ant</args>

<args>p2.composite.add</args>

<args>-Dsite.label="${site.label}"</args>

<args>-Dcomposite.base.dir=${github-local-clone}</args>

<args>-DunqualifiedVersion=${unqualifiedVersion}</args>

<args>-DbuildQualifier=${buildQualifier}</args>

<args>-DparsedVersion.majorVersion=${parsedVersion.majorVersion}</args>

<args>-DparsedVersion.minorVersion=${parsedVersion.minorVersion}</args>

</applicationsArgs>

</configuration>

</execution>

</executions>

</plugin>



 |

Let’s see these configurations in detail. In particular, it is important to understand how the goals of the plugins are bound to the phases of the default lifecycle; remember that on the phase _package_, Tycho will automatically create the p2 repository and it will do that before any other goals bound to the phase _package_ in the above configurations:

*   with the _build-helper-maven-plugin_ we parse the current version of the project, in particular, we set the properties holding the major and minor versions that we need later to create the composite metadata directory structure; its goal is automatically bound to one of the first phases (_validate_) of the lifecycle;
*   with the _exec-maven-plugin_ we configure the execution of the Git commands:
    *   we clone the Git repository of the update site (with _–depth=1_ we only get the latest commit in the history, the previous commits are not interesting for our task); this is done in the phase _pre-package_, that is before the p2 repository is created by Tycho; the Git repository is cloned in the output directory _target/checkout_
    *   in the phase _verify_ (that is, after the phase _package_), we commit the changes (which will be done during the phase _package_ as shown in the following points)
    *   in the phase _deploy_ (that is, the last phase that we’ll run on the command line), we push the changes to the Git repository of the update site
*   with the _maven-resources-plugin_ we copy the p2 repository generated by Tycho into the _target/checkout/releases_ directory in a subdirectory with the name of the qualified version of the project (e.g., 1.0.0.v20210307-2037);
*   with the _tycho-eclipserun-plugin_ we create the composite metadata; we rely on the Eclipse application _org.eclipse.ant.core.antRunner_, so that we can execute the p2 Ant task for managing composite repositories ([_p2.composite.repository_](https://wiki.eclipse.org/Equinox/p2/Composite_Repositories_(new)#Composite_Repositories_as_part_of_the_build)). The Ant tasks are defined in the Ant file **packaging-p2composite.ant**, stored in the _site_ project. In this file, there are also a few properties that describe the layout of the directories described before. Note that we need to pass a few properties, including the _site.label_, the directory of the local Git clone, and the major and minor versions that we computed before.

Keep in mind that in all the above steps, non-existing directories will be automatically created on-demand (e.g., by the _maven-resources-plugin_ and by the p2 Ant tasks). This means that the described process will work seamlessly the very first time when we start with an empty Git repository.

Now, from the parent POM on your computer, it’s enough to run

|  | 

mvn deploy  -Prelease-composite



 |

and the release will be performed. When cloning you’ll be asked for the password of the GitHub repository, and, if not using an SSH agent or a keyring, also when pushing. Again, this depends on the URL of the GitHub repository; you might use an HTTPS URL that relies on the GitHub token, for example.

If you want to make a few local tests before actually releasing, you might stop at the phase _verify_ and inspect the _target/checkout_ to see whether the directories and the composite metadata are as expected.

You might also want to add another _execution_ to the _tycho-eclipserun-plugin_ to add a reference to another Eclipse update site that is required to install your software. The Ant file provides a task for that, _p2.composite.add.external_ that will store the reference into the innermost composite child (e.g., into 1.2.x); here’s an example that adds a reference to the Eclipse main update site:

| 

1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

17

18

19

20

21

22

23

24

25



 | 

<execution>

<!\-\- Add composite of required software update sites...

    (if already present they won't be added again) -->

<id>add-eclipse-repository</id>

<phase>package</phase>

<goals>

<goal>eclipse-run</goal>

</goals>

<configuration>

<applicationsArgs>

<args>-application</args>

<args>org.eclipse.ant.core.antRunner</args>

<args>-buildfile</args>

<args>packaging-p2composite.ant</args>

<args>p2.composite.add.external</args>

<args>-Dsite.label="${site.label}"</args>

<args>-Dcomposite.base.dir=${github-local-clone}</args>

<args>-DunqualifiedVersion=${unqualifiedVersion}</args>

<args>-DbuildQualifier=${buildQualifier}</args>

<args>-DparsedVersion.majorVersion=${parsedVersion.majorVersion}</args>

<args>-DparsedVersion.minorVersion=${parsedVersion.minorVersion}</args>

<args>-Dchild.repo="https://download.eclipse.org/releases/${eclipse-version}"</args>

</applicationsArgs>

</configuration>

</execution>



 |

For example, in my Xtext projects, I use this technique to add a reference to the Xtext update site corresponding to the Xtext version I’m using in that specific release of my project. This way, my update site will be “self-contained” for my users: when using my update site for installing my software, p2 will be automatically able to install also the required Xtext bundles!

Releasing from GitHub Actions
-----------------------------

The Maven command shown above can be used to perform a release from your computer. If you want to release your Eclipse update site directly from GitHub Actions, there are a few more things to do.

First of all, we are talking about a GitHub Actions workflow stored and executed in the GitHub repository of your project, NOT in the GitHub repository of the update site. In this example, it is [https://github.com/LorenzoBettini/p2composite-github-pages-example](https://github.com/LorenzoBettini/p2composite-github-pages-example).

In such a workflow, we need to push to another GitHub repository. To do that

*   create a [GitHub personal access token](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token) (selecting _repo_);
*   create a secret in the GitHub repository of the project (where we run the GitHub Actions workflow), in this example it is called _ACTIONS_TOKEN_, with the value of that token;
*   when running the Maven deploy command, we need to override the property _github-update-repo_ by specifying a URL for the GitHub repository with the update site using the HTTPS syntax and the encrypted _ACTIONS_TOKEN_; in this example, it is _https://x-access-token:${{ secrets.ACTIONS_TOKEN }}@github.com/LorenzoBettini/p2composite-github-pages-example-updates_;
*   we also need to configure in advance the Git _user_ and _email_, with some values, otherwise, Git will complain when creating the commit.

To summarize, these are the interesting parts of the _release.yml_ workflow (see the full version here: [https://github.com/LorenzoBettini/p2composite-github-pages-example/blob/master/.github/workflows/release.yml](https://github.com/LorenzoBettini/p2composite-github-pages-example/blob/master/.github/workflows/release.yml)):

| 

1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

17

18

19

20

21

22

23

24

25

26

27



 | 

name: Release with Maven

on:

push:

branches:

-  release

jobs:

build:

runs-on: ubuntu-latest

steps:

\- uses: actions/checkout@v2

\- name: Set up JDK 11

uses: actions/setup-java@v1

with:

java-version: 11

\- name: Configure Git

run: |

git  config  --global  user.name  'GitHub Actions'

git  config  --global  user.email  'lorenzo.bettini@users.noreply.github.com'

\- name: Build with Maven

run: >

mvn  deploy

-Prelease-composite

-Dgithub-update-repo=https://x-access-token:${{  secrets.ACTIONS_TOKEN  }}@github.com/LorenzoBettini/p2composite-github-pages-example-updates

working-directory: p2composite.example.parent



 |

The workflow is configured to be executed only when you push to the **release** branch.

Remember that we are talking about the Git repository hosting your project, not the one hosting your update site.

Final thoughts
--------------

With the procedure described in this post, you publish your update sites and the composite metadata during the Maven build, so you never deal manually with the GitHub repository of your update site. However, you can always do that! For example, you might want to remove a release. It’s just a matter of cloning that repository, do your changes (i.e., remove a subdirectory of _releases_ and update manually the composite metadata accordingly), commit, and push. Now and then you might also clean up the history of such a Git repository (the history is not important in this context), by pushing with _–force_ after resetting the Git history. By the way, by tweaking the configurations above you could also do that every time you do a release: just commit with _amend_ and _push force_!

Finally, you could also create an additional GitHub repository for snapshot releases of your update sites, or for milestones, or release candidate.

Happy releasing! 🙂