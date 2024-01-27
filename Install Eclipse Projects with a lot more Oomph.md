# Install Eclipse Projects with a lot more Oomph
At EclipseCon last week, Eike Stepper and Ed Merks demonstrated [Eclipse Oomph](https://wiki.eclipse.org/Eclipse_Oomph_Installer), a provisioning and installation tool that can quickly install Eclipse features and plugins as well as materialise projects from repositories. Designed to improve the out-of-the-box experience when using Eclipse for the first time, the Eclipse Oomph installer presents different functionality sets in a simplified way to allow new Eclipse users to get up to speed with their specific flavour of development, such as a Java or C platform. The installer is bundled into all of the [Eclipse developer packages](https://www.eclipse.org/downloads/index-developer.php) for Mars, which is scheduled for release in June. 

Eclipse Oomph has a questionnaire to enable certain sensible preferences, such as whether line numbers or text spell checking should be enabled by default. These choices are saved in the user's home directory, and will be applied to any new installations or Eclipse workspaces automatically. It can be launched manually with 'Preferences → Oomph → Setup → Start Welcome Questionnaire...'

![](news/2015/03/eclipse-oomph/en/resources/Screen Shot 2015-03-21 at 01.22.32.png)

Projects can be imported with 'File → Import → Oomph → Projects into Workspace' through a list of projects populated from either Eclipse.org or GitHub.com:

![](news/2015/03/eclipse-oomph/en/resources/Screen Shot 2015-03-21 at 01.45.51.png)

After the projects have been cloned from the remote repositories, the projects are imported and set up to go.

InfoQ caught up with Eike Stepper, project lead of the Eclipse Oomph project, to find out more deatils, and started off by asking him to explain what it does:

> **Stepper**: Oomph is a provisioning technology that automates the entire process of installing project-specific IDEs and configuring user-specific preferences and the user’s favourite tools.

**InfoQ**: What made you create Eclipse Oomph in the first place?

> **Stepper**: I’m the project lead of the CDO model repository project for 12 years and I wanted to make it easier for my committers and contributors to get to a state where they can actually contribute to CDO. I originally created a very small and hacky tool to setup a CDO workspace. When I showed that to Ed Merks he wanted it for EMF, and we started to make it more flexible for other projects. Together we worked on the installer for the last year and a half because we felt it solves a problem that almost everybody has; and the result is Eclipse Oomph.

**InfoQ**:How does Oomph work?

> **Stepper**:At the core, Oomph is a model-based setup engine that performs setup tasks. For any given development environment the applicable setup tasks are organized in three local files:
> 
> ![](news/2015/03/eclipse-oomph/en/resources/Tasks.png)
> 
> We use the setup engine in two places – first, an installer application which can be used to bootstrap new installations and workspaces, and secondly, the engine is deployed into the installed (or existing) IDEs, where it ensures that the configuration is always in sync with the chosen setup tasks.

**InfoQ**: What kind of configurations can Eclipse Oomph manage?

> **Stepper**: The most important are p2 director tasks that modify the installation (installing features and plugins into the IDE); tasks to modify the eclipse.ini file (for example, adding more memory); copying files to specific locations or to modify the content of files based on regular expressions; tasks to clone Git repositories and to import projects from them; tasks to specify what JVM to use and what to put in the Eclipse target platform (for plug-in development); tasks that orchestrate Maven builds and imports, modify user preferences, key bindings and many others. It is extensible so the users can implement their own tasks and let them be discovered through a remote task registry.

**InfoQ**: What is an Eclipse Oomph catalog?

> **Stepper**:In addition to the setup tasks in the three local files the configuration of a development environment can be linked to setup tasks from a set of remotely managed task catalogs. These catalogs define the tasks needed to configure specific Eclipse products and projects:
> 
> ![](news/2015/03/eclipse-oomph/en/resources/Catalogue.png)
> 
> The Oomph installer lets the user pick a product version to pre-configure an IDE and to keep it up to date; for example, an EPP package like the ones on Eclipse’s download page. It is then possible to add multiple streams from the project catalogs, so that their setup tasks are all executed to configure workspace projects when the Eclipse workspace first starts to, e.g., clone source code repositories, checkout projects, create dynamic working sets, and so on.
> 
> If an existing IDE has not been installed with the Oomph installer the product is automatically induced by Oomph and the user is then able to link projects from the remote catalogs to it and have the user setup tasks applied, as well.

**InfoQ**: How can projects create their own Oomph setup files to make it easier to check out the code?

> **Stepper**: There is a Project Setup Wizard which makes it easy to create a project-specific setup model. It asks a few questions and creates the model, which can be subsequently edited. The file is stored in a version control system, and can be linked into a project catalog. To add it into the global catalog at Eclipse a [bug needs to be submitted](https://bugs.eclipse.org/bugs/enter_bug.cgi?product=Oomph&component=Setup&bug_severity=enhancement&short_desc=Please%20include%20setup%20for%20project%20Xyz) against Tools/Oomph/Setup. To obtain the catalog information from an internal server the URLs of the catalog, product and project files can be redirected as needed.

Eclipse Oomph is available by default in all Eclipse Mars packages, available from the [Eclipse download site](https://www.eclipse.org/downloads/index-developer.php).  
￼￼