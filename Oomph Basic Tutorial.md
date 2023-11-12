# Oomph Basic Tutorial
Oomph is a project providing tooling to make the installation and configuration of Eclipse instances easier. The ultimate goal is to install and set-up an Eclipse IDE or product with a single click. Oomph also allows to share setups, as an example, you can preconfigure an IDE to work on a certain project and all developers can use this setup to get easily involved in the project.

Before we get started with the hands-on tutorial, we will introduce some basic concepts in this section.

Oomph stores configuration information in files, which can be easily shared. Those files are called “Setup Models”. There are two types of setups models in Oomph.

**Product setup models** define a complete Eclipse instance to be installed. This includes all plugins, which shall be contained in the Eclipse installation as well as default preferences and other basic settings. Product setups are comparable to the Eclipse packages you can download from eclipse.org (e.g. the Java EE Edition or the Modeling Edition). The difference being, you can chose exactly what should be in your product. Additionally, you can add plugins, which are not part of the release train, e.g. Checkstyle.

**Project setup models** define the set-up for a specific development project. This includes gut repositories, projects to be checked out, project-specific settings, etc..

Project setup models always need a product setup model first to create the Eclipse instance. However, Oomph already provides the standard Eclipse Packages as product setup models. So it is possible to just create a project setup model without the need to configure a product and vice versa.

Later in this tutorial, we will describe in detail how to specify product setup models and then project setup models. Before we cover this, a few words about how Oomph is working in general and about the different components it consists of. The following diagram shows a (simplified) overview of all relevant Oomph components. The **Oomph installer** (also called Eclipse installer) is a dedicated application, you can download and launch. As the name implies, it is responsible for installing Eclipse instances. Therefore, it presents the user a list of available project setup models and, subsequently, a list of available project setup models. Based on those two models, it will create an Eclipse installation for you. Additionally, it allows the user to select a few settings, which are related to the installation, such as the installation directory. We will describe the installer more in detail later.

![](https://eclipsesource.com/blogs/tutorials/oomph-basic-tutorial/images/image11_hub8fc797f91f8133ad195fcc2ff7bae81_22893_797x274_resize_q100_h2_box_3.webp)

Besides the Oomph installer, Oomph also provides three “runtime” components, which are typically installed in the Eclipse instances: the **Oomph Updater**, the **Oomph Project Setup** and the **Oomph Authoring** tooling, including the **Preference Recorder**.

The **Oomph Updater** is responsible for keeping an existing Eclipse instance up-to-date once it has been created. Therefore, on every start-up, it checks the product setup model for any changes in the configuration. If there are any changes, e.g. a new bundle has been added, it will apply those changes automatically to the existing Eclipse instance.

The **Oomph Project Setup** component reads a project setup model and applies it to a created Eclipse instance. This means, it configures the Eclipse with project specific settings. Some of those settings are already applied during the installation (e.g. the addition of bundles) and are therefore taking care of by the Oomph Installer. Some project specific things can only be done, once the Eclipse instance is already created and started, therefore, the Oomph Project Setup component runs in this Eclipse instance and manages this second step. As an example, it clones specific Git repositories, imports projects, configures Mylin queries, and so on. We describe the configuration of an project setup model in more detail in the section Project Setup Models.

Finally, Eclipse instances typically contain the **Oomph Authoring** tooling. This tooling allows you to create and modify product setup models and project setup models. This powerful tooling requires quite a few Eclipse framework components, such as Editors and EMF. Additionally, you typically want to share setup models with Git. Last but not least, the tooling allows you to capture some settings from a running instance, e.g. using the **Preference Recorder**. That is the reason, the **Oomph Authoring** is not part of the installer, but the running Eclipse instance. This way, the installer remains a very slim component, just responsible for using setup models. We will describe the Oomph Authoring tooling more in detail in the section “The Oomph Authoring Tooling”.

Installing the three Oomph runtime components into an Eclipse instance is optional. The “Oomph updater” and the “Oomph project setup” are typically wanted to keep the installation up to date and to enable the usage of project setup models. Therefore, we recommend you always include those two runtime components. The “Oomph Authoring” is only required, if users of the Eclipse instance are expected to create and edit Oomph profiles. As this also involves the creation of user specific profiles, based on common profiles, we recommend you additionally install this runtime component.

After this basic introduction, we will describe the Oomph components in more detail. In the first section, we explain how to use “The Oomph Installer / Eclipse Installer” from an end-user perspective. In the second section, we give a general introduction about “The Oomph Authoring Tooling. Subsequently, we describe how to create “Oomph Product Setup Models” and “Oomph Project Setup Models”.

The Oomph Installer / Eclipse Installer
---------------------------------------

The Oomph installer (also called Eclipse installer) is a stand-alone SWT application, which can read product/project setup models and, based on those, install pre-configured Eclipse instances. If you only want to consume a setup model, this is the only tool you need to use. You can download the installer for your operating system [from eclipse.org](https://wiki.eclipse.org/Eclipse_Oomph_Installer). After the installation, you can simply launch the installer. The installer supports two modes, “the simple mode” and “the advanced mode”. You can always switch between the two. Generally, the simple mode is easier to use, if you just want to install a default Eclipse, however, it does not provide all of the configuration options of the advanced mode. By default, the installer is launched in the simple mode, to switch to advanced mode, please click on the menu button in the top right corner.

![](https://eclipsesource.com/blogs/tutorials/oomph-basic-tutorial/images/image13_hu045418b286c5e87b1fe1791df97d8e8f_5281_604x113_resize_q100_h2_box_3.webp)

In the following sections, we will describe all of the options of the Oomph installer. Those, which are only available in advanced mode are marked as such.

### Select Product Setup Model

On the first page of the installer, you can select the product, you want to install from a list of available product setup models. The simple mode shows a flat list, the advanced mode groups them by “product catalogue”. By default, the list contains all standard packages provided by the Eclipse Packaging Project (EPP), such as “Eclipse for Java Developers” or “Eclipse for PHP Developers”. However, you can add your own product setup models here, which is described in this section. To select a product to be installed in the simple mode, you need only to click on it. In the advanced mode, select it and click “next”.

### Product Version (advanced)

The advanced mode allows you to specify the simultaneous release on which the installation will be based on, e.g. “Mars”, “Mars.1” or “Neon”. That means, that all bundles will be installed from the corresponding release update site. Please note, that a certain Eclipse version must be explicitly supported by a product setup model. Alternatively, you can also choose the “latest release”, which will use the most recent Eclipse release, or “latest”. This will use the developer builds of the upcoming Eclipse release. The simple mode will always use the latest release.

### Configure JVM

Both modes allow you to configure the JVM to be used for the Eclipse installation. (second page for the simple mode, first page for the advanced mode). This JVM will be used later to start the Eclipse installation you install with the Oomph installer.

### Configure installation directory

Both modes allow you to specify the directory, in which the Eclipse instance will be installed. Please note, that if you use bundle pools (see next section), the installation directory will mainly contain the executable launcher, the instance-scoped preferences, and some ressources. The bundles, of which an Eclipse installation contains will be placed in the bundle pool.

### Configure Bundle pools (advanced)

Bundle pools are a concept of p2 to save space on disk if you are running multiple Eclipse instance on one machine. While different installations of Eclipse can have different setting and maybe slightly different plugins installed, they typically share the majority of bundles among each other. In order to avoid downloading and storing those common bundle several times, bundle pools provide a common directory for bundles to be used by several Eclipse instances. Instead of the installation directory, all bundles are placed in those folders then, you only need one bundle pool per Eclipse version. On startup, every Eclipse instance will access its specific subset of bundles from the bundle pool during runtime. Probably the only disadvantage of using bundle pools is that installation are not self-contained anymore. So you cannot just zip them and send them around. However, when using Oomph, sharing of Eclipse installations is no longer a typical problem. Bundle pools are activated by default and as long as you do not want to take care of the location that is stored (in a .eclipse subfolder of your home directory), you do not necessarily have to care about them. In the simple mode, it is only possible to switch bundle pools on or off, the default is “on”.

### Add product setup models (advanced)

As described earlier, the Oomph installer already shows some default products to be installed. If you want to use an existing product setup model, provided by someone else, or a product you defined yourself, you need to add it to the installer first. To do that, select the “plus” icon in the advanced mode right on top of the list of product setup models. This allows you to directly add a file from disk or a URL, where the setup model is located. The latter allows you to host the setup model at any location, which is reachable over HTTP, e.g. GitHub or any web server. Therefore, it provides a very simple way of sharing product setup models. For example, we use this way to [share our EclipseSource Oomph Profile](https://eclipsesource.com/blogs/2015/08/17/introducing-the-eclipsesource-oomph-profile/). Custom profiles will be shown in the category “User Products”.

### Select Project Setup Model (advanced)

After you have selected a product, the advanced mode also allows you to select a project setup model. As described before, project setup models will do project specific set-ups, such as checking specific repositories, setting up Mylin queries, as well as installing additional bundles, which are only required to work on a specific projects. As for the product list, the installer shows a list of available public setups as before, you can also add your own custom setups models on a project level.

To select a specific project, double click it in the list. Please note, that it is possible to add several project setup models to the list below, before you trigger the installation. This will execute all project setup tasks from all selected projects, so in theory, you receive an Eclipse working on multiple projects. However, as the setups from different projects might collide, this is a rather special case in practice.

Project setup models can support different “Streams”. That means, the setup for this project is sightly different for different versions or releases. If a project supports more than one stream, you can choose the stream to be installed in the table of selected projects.

### Select Installation Folder

On the last page in both modes, you can select the installation folder for the Eclipse instance to be installed. In this folder, Oomph will create the launchable allowing you to start the installed Eclipse as well as instance specific resources and settings. Please note that if you use bundle pools, the installation directory will not contain the bundles of which the installation consists, they will be retrieved from the respective bundle pool instead (on start-up). The simple mode also allows to create a desktop and menu entry to start the installed instance. In the advanced mode, you must directly start the launchable in the installation folder or create the shortcut yourself.

### Project specific settings (advanced)

In the advanced mode and if you have selected a project specific setup model, you might need to enter some project-specific variables. Those are defined in the project setup model and can therefore differ from project to project. A typical example is the user name, which is used to access git repositories. If you are in doubt about an available variable, you should contact the party, who provides the project setup model you want to install.

The Oomph authoring tooling enables you to create and modify product and project setups models. It is implemented as a plugin for the Eclipse IDE and is already part of most Eclipse packages as well as the [Eclipse Oomph profile](https://eclipsesource.com/blogs/2015/08/17/introducing-the-eclipsesource-oomph-profile/). Alternatively, you can download and install it from the [Oomph update site](https://projects.eclipse.org/projects/tools.oomph/downloads) (Feature Oomph setup). Once you have installed the tooling, it allows you to create Oomph artifacts with the default Eclipse “New” wizard. To version them, you can add them to an empty project, which is shared in a git repository. In the following two sections, we go in detail about product setup models and project setup models.

Both artifacts can be opened in a tree-based editor. It displays a hierarchy of elements, which specify the setup model in detail. In the following two sections, we describe the most relevant elements of a product setup model and a project setup model.

![](https://eclipsesource.com/blogs/tutorials/oomph-basic-tutorial/images/image03_hu1090f44b36d9e2c433bc05fb46ac2c8f_15706_506x440_resize_q100_h2_box_3.webp)

Both, product and project setup models are extensively based on using variables. This improves the maintainability of Oomph profiles and also allows you to connect generic profiles to user specific settings. As an example, a project setup needs to know about the JRE in order to be used for launching applications. When specifying a project setup model, you do not know, where the JRE of a future user is located, therefore, the profile is bound to a variable, which can then be instantiated by the later user of a profile, meaning the user can select the JRE to be used. Some variables are predefined by Oomph, for example the location of the Eclipse Git and Gerrit server.

Oomph Product Setup Models
--------------------------

In this section, we describe how to define a custom product setup model. To create one, right click a container project, click “new” and select Oomph => Setup Product Model. The initial wizard allows you to specify a Label, a name, a description, the release train your product is based on, an installable unit ID, the container project, and a file name. The wizard focusses on the use case, where you want to specify a custom product based on Eclipse. Therefore, it will configure the product to contain basic features such as the Eclipse platform, the Eclipse IDE platform and Eclipse RCP. Additionally, it will add a feature using the ID you have specified under “installable unit ID”. If you want to configure a product based on Eclipse, this ID should be the ID of the root feature of your product. In the following, we describe the use case to build a custom IDE as we have done for the [EclipseSource Oomph Profile](https://eclipsesource.com/blogs/2015/08/17/introducing-the-eclipsesource-oomph-profile/). In this use case, we just bundle existing plugins for the Eclipse IDE, we do not provide any custom features. However, technically, both use cases are working the same and the way you specify them is similar. As a reference and to see the final result of this tutorial, you have can a look at the [EclipseSource Oomph Profile](https://eclipsesource.com/blogs/2015/08/17/introducing-the-eclipsesource-oomph-profile/), which can be [found on github](https://github.com/eclipsesource/oomph).

![](https://eclipsesource.com/blogs/tutorials/oomph-basic-tutorial/images/image04_hue85204da95872773b17c92c1fa95a759_16061_525x555_resize_q100_h2_box_3.webp)

Once you have create the product setup model, it is opened with the Oomph editor. As you can see in the following screenshot, the initial model contains one root node and three elements on the first level. The root node represents the product itself. In its properties, you can adapt the Label, the id, and the description. This information will be shown in the Oomph installer, when users select your product.

The node “BrandingInfo” contains two key/value pairs, which specify the name of the folder, in which the product will be installed. This is not the installation folder name, which can be specified in the installer before the installation. Oomph will create a subfolder in the installation folder that is named as specified in the product setup model. For our use case, we leave those values to the default “eclipse”. The next element “Installation” is required by Oomph, but can be ignored for now.

![](https://eclipsesource.com/blogs/tutorials/oomph-basic-tutorial/images/image09-1_hue241ced2e60b0dc54b047af5d3860e80_3309_165x114_resize_q100_h2_box_3.webp)

The fourth element “Mars” is a “Product Version”, it has been created because we selected “Mars” in the initial creation wizard. Product versions allow us to maintain a product setup model for several Eclipse release streams in parallel, e.g. for Mars and Luna. At the moment, our product just supports Mars. You can add additional product versions with a right click on the product (the root node). If we unfold the Mars product version node, we see the core element of the initial product, a “P2 Director” element. This element specifies a list of features, which will be installed into our product.

![](https://eclipsesource.com/blogs/tutorials/oomph-basic-tutorial/images/image07_hu8ecafe9dd55929d32dd690f44fad8faf_3454_415x128_resize_q100_h2_box_3.webp)

In the following sections, we will describe the above mentioned elements as well as the other most relevant elements in more detail. Please note that this is not a complete list, we just focus on the core features of Oomph. If you feel something is missing, please contact us, as we might be able to extend this tutorial in the future.

### Products and Product Versions

The root of every product setup model is a “product”. It can contain elements that describe steps to be taken during the installation. A product contains a “product version”. Product versions are like branches or release streams for a product, this allows you to maintain a product in multiple versions. A typical use case is to maintain a product for several Eclipse major versions, e.g. Luna, Mars and Neon. Users of your product setup model can select which product version they want to install in the Oomph installer. Product versions can be created directly as children elements of a product. If you work with product versions, all following elements can be either created in the product or a sub element of the product version. Elements in the product (root) will affect every installation of the product setup model. Elements underneath a product version will only be applied if the user has selected this specific version. So at the end, product versions will inherit everything from the containing product and can add version specific things. Both, products and product versions allow us to define a name and a label.

### P2 Director

As one of the most important elements, the “P2 Director” specifies the content of your Eclipse installation. More precisely, the features and bundles to be installed. As described before, P2 directors can be added to the product or a specific product version only. To add bundles and features, the P2 director element supports two sub elements, which can be added as children elements.

Additionally, P2 director elements can specify a label, which is shown to the user during the installation for information purposes. The template of product setup models uses variables to specify the label: $ ${scope.product.label} (${scope.product.version.label}). This will resolve to “Product Label” + “Product Version Label”, e.g. “My Product Mars”. The usage of the second variable only makes sense, if the P2 director is placed inside a product version.

Oomph Project Setup Models
--------------------------

In this section, we describe how to define a custom project setup model. In order to create one, right click a container project, click “new” and select Oomph => Setup Project Model.

The wizard allows you to select between three templates: Eclipse Project, GitHub Project, and Simple Project. If you choose Eclipse or GitHub, the wizard will assist you in adding your specific repository from which you plan to check the sources. In turn, this will save configuration effort. In the following, we will assume you set-up a project setup model for an Eclipse project, however, this can easily be transferred to GitHub or any other hosting service. The initial wizard allows you to specify a Label, a name and a description for your project.

If you use the Eclipse or GitHub project template, you just need to specify the Git path within the Git repository, as Oomph already knows the URL for the repository itself and will use a variable to define it. For Eclipse projects, you can configure under “Remote URIs”, whether the repo should be accessed using Git, Gerrit or both. Based on the repository information, Oomph will be able to clone the project repository, if you execute the project setup model later on.

The next thing to consider is configuring the projects to be imported to your Eclipse workspace after the repo has been cloned. There are two ways of configuring it, please see the next section to learn more about them. This also covers the next two properties in the initial wizard to configure an “installable unit”.

The final option in the initial wizard is selecting the default JRE for you project.

![](https://eclipsesource.com/blogs/tutorials/oomph-basic-tutorial/images/image00-2_hue04b649d5736658a0120e4fc82680762_18559_532x639_resize_q100_h2_box_3.webp)

After finishing the wizard, the Oomph authoring tooling creates a pre-configured project setup file for you. In the following sections, we will describe possible adaptations of this default, starting with the configuration of which projects are imported after the repo is cloned.

![](https://eclipsesource.com/blogs/tutorials/oomph-basic-tutorial/images/image11%20%281%29_hu646f0aa4cf81168a1c569fb2473cf2b5_8628_599x181_resize_q100_h2_box_3.webp)

### Simple Project Import

Oomph allows you to specify which projects are to be imported into the workspace after the Git repo has been cloned. Thereby, you can directly start to work on the sources after you have installed your IDE with the corresponding project setup model.

There are two way of specifying the project import in Oomph. The first and simple one is to import all projects from a git clone. The second one, and this only works for projects based on bundles and features, is to specify a “root feature”, which transitively includes all the bundles you wish to be imported. This includes the use of a concept called “Modular Target” (aka “targlet”).

In the following, we describe both ways of importing projects. Please note, that the project setup model creation wizard will set-up the second option for you, so, if you do not want to use “targlets”, you need to adapt it.

#### Projects Import Task

In order to simply import projects from a git clone, please add a “Projects Import” task to your project setup model:

![](https://eclipsesource.com/blogs/tutorials/oomph-basic-tutorial/images/image2_hu30fb493f0048fa2134d82f23c900cc04_74834_624x356_resize_q100_h2_box_3.webp)

After adding the project import task, you need to configure it using a “Source Locator”, which can be created as a child element of the project import task.

![](https://eclipsesource.com/blogs/tutorials/oomph-basic-tutorial/images/image6-1_hu5400bb0d587c6dd43bf4c3194ffc5df9_11506_595x51_resize_q100_h2_box_3.webp)

The source locator specifies the location from which to import the projects. You should use variables for the “Root Folder” value, typically the git clone location: ${git.clone.location}

![](https://eclipsesource.com/blogs/tutorials/oomph-basic-tutorial/images/image9_hu21459a999b1bfc50bbcd7e7731ad4d30_8294_379x95_resize_q100_h2_box_3.webp)

To filter imported projects, you can add “predicates” as a child node to the source locator, e.g. you could only import “plugins”:

![](https://eclipsesource.com/blogs/tutorials/oomph-basic-tutorial/images/image7_hub129f37eddb25d443028304de55ed880_5571_354x70_resize_q100_h2_box_3.webp)

Using the simple project import, Oomph will import all projects from the clone location into your workspace (depending on the predicates) without checking whether the OSGi dependencies of the project are satisfied or not. This is useful for plain Java projects or if you want to use your own “classic” target definition file instead of the targlet platform created by Oomph.

#### Project import using the Targlets Task

The alternative to the simple project import task is the “Targlets” (“Modular Target”) task, which is specific for OSGi-based projects. Instead of adding a “Projects Import” task, please add a “Targlet” task to your project setup model. If you have used the project setup model creation wizard, Oomph will already create a template targlet for you.

![](https://eclipsesource.com/blogs/tutorials/oomph-basic-tutorial/images/image13%20%281%29_hu4897ea00516747b0f16caf6dfb7b65bb_77821_622x437_resize_q100_h2_box_3.webp)

The ”Targlets” task defines what makes up your project setup. This not only includes features to be imported from repositories, but also dependencies like the eclipse platform. A Targlet contains requirements, those features which need to be resolved. It further can contain Source Locators, this is used to resolve the requirements from the sources by importing the features and its plugins. And finally it contains repositories, which are used to resolve the dependencies as binaries by adding them to a target definition.

During the resolution, all features (and their projects) which are exclusively available from a git repository will be imported as projects, all other features will be used in the oomph target. Oomph will transitively resolve all requirements, so if your project has one “root feature”, it is enough to add this one to the targlet.

![](https://eclipsesource.com/blogs/tutorials/oomph-basic-tutorial/images/image3_huce73e070484315047533e2e56d91cfb3_21672_326x169_resize_q100_h2_box_3.webp)

Please note, that targlets only work with OSGi projects. The advantage of using targlets is that you can ensure, that the resulting workspace will be fully resolved without any dependency issues. Please also note, that you cannot use a default target definition file in combination with a targlet and your installation will not finish if a dependency is missing.

### Working Sets

The working sets element allows you to pre-set-up workings sets in which imported projects are structured in the workspace. The working sets provided by Oomph are more powerful than the traditional working set support of Eclipse, it allows you to assign projects based on rules. As an example, you can define a working set for regular projects and another one for all test projects. As this is based on a rule (e.g. a name convention), new projects will automatically be assigned to the right working set.

Working Sets can be created as child elements of the Working Sets root node and should have a name assigned. If you right click a Working Set, you can create Predicates, which influence the content of the Working Set. Those Predicates can also be combined. In the following we describe the most frequently used Predicates. To test your dynamic working sets, you can apply them to your current workspace. Right click in the Oomph project setup editor and select “Working Sets Preview”. A dialog will shows you how the project which is in your current workspace would be structured following the current configuration.

#### Repository Predicate

This allows you to specify a repository from which projects are included. The repository is defined by selecting one project from this repository. All projects from the same repository will then be included in this Working Set. In the sample project setup model, created by the initial wizard, there is an initial Working Set including everything from the same repository, which contains the project “org.eclipse.cool.stuff-feature”.

#### Name Predicate

The Name Predicate allows you to define a regular expression, which is matched against project names. Projects that match the expression will then be included into the Working Set. As an example, a Name Predicate with the pattern “._.test._” will include all test projects.

#### Inclusion/Exclusion Predicate

These predicates allow you to depend on the content of another working set and either include or exclude it in the working set. As an example, if you have configured a working set containing all tests, you could configure a second working set now with the main code, which just excludes the test working set.

#### And/Or/Not Predicate

If you need multiple predicates (e.g. name combined with exclusion), you can use “And”, “Or” and “Not” predicates to combine them. Create those predicates first and then create other predicates as children elements. All children will then be combined following the conditional statement.

### Additional Tasks

Please note that there is a variety of additional tasks to be configured in a project setup model. Some of them, such as the “P2 director” or the “Eclipse.ini” tasks are also available in product setups models. In this case, you can enhance a product setup model with project specifics, e.g. an additional plugin to be installed.

If you want to get started with Oomph, we recommend to use an existing setup model as a template, e.g. [our EclipseSource Oomph Profile](https://eclipsesource.com/blogs/2015/08/17/introducing-the-eclipsesource-oomph-profile/). Please let us know, if you felt a specific topic was missing in this tutorial!