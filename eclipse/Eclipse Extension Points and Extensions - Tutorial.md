# Eclipse Extension Points and Extensions - Tutorial
In this exercise you create two new plug-ins. The first one contains an _extension point_. The second plug-in contributes an extension to the new extension point.

### [](#create-an-extension-point)[7.2. Create an extension point](#create-an-extension-point)

Open the _MANIFEST.MF_ file or the _plugin.xml_ file and select the _Extension Points_ tab.

![](https://www.vogella.com/tutorials/EclipseExtensionPoint/img/extensionpoint10.png)

Press the **Add…​** button.

Enter "com.vogella.extensionpoint.definition.greeter" as _Extension Point ID_ and "Greeter" as _Extension Point Name_ in the dialog. The _Extension Point Schema_ field is automatically populated based on your input.

![](https://www.vogella.com/tutorials/EclipseExtensionPoint/img/extensionpoint20.png)

Press the **Finish** button. The definition of the extension is generated and the schema editor opens. Switch to the _Definition_ tab.

![](https://www.vogella.com/tutorials/EclipseExtensionPoint/img/extensionpoint30.png)

### [](#add-attributes-to-the-extension-point)[7.3. Add attributes to the extension point.](#add-attributes-to-the-extension-point)

For that click on the **New Element** button. Give the new element the name "client".

![](https://www.vogella.com/tutorials/EclipseExtensionPoint/img/extensionpoint40.png)

Select the "client" element and press _New Attribute_. Give the new element the name "class" and the type "java". Enter `com.vogella.extensionpoint.definition.IGreeter` as interface name

![](https://www.vogella.com/tutorials/EclipseExtensionPoint/img/extensionpoint50.png)

This interface doesn’t exit yet. Press on the hyperlink called _Implements_ to create it based on the following code.

```
package com.vogella.extensionpoint.definition;

public interface IGreeter {
    void greet();
}
```

Go back to your extension point definition and add a choice to the extension point. For this select the _extension_ entry, right-click on it and select . This defines how often the extension "client" can be provided by contributing plug-ins. We will set no restrictions (unbound).

![](https://www.vogella.com/tutorials/EclipseExtensionPoint/img/extensionpoint70.png)

![](https://www.vogella.com/tutorials/EclipseExtensionPoint/img/extensionpoint80.png)

![](https://www.vogella.com/tutorials/EclipseExtensionPoint/img/extensionpoint90.png)

![](https://www.vogella.com/tutorials/EclipseExtensionPoint/img/extensionpoint92.png)

### [](#export-the-package)[7.4. Export the package](#export-the-package)

Select the _MANIFEST.MF_ file switch to the _Runtime_ tab and export the package which contains the `IGreeter` interface.

![](https://www.vogella.com/tutorials/EclipseExtensionPoint/img/extensionpoint100.png)

### [](#add-dependencies)[7.5. Add dependencies](#add-dependencies)

Add the following dependencies via the _MANIFEST.MF_ file of your new plug-in:

*   `org.eclipse.core.runtime`
    
*   `org.eclipse.e4.core.di`
    

### [](#evaluating-the-registered-extensions)[7.6. Evaluating the registered extensions](#evaluating-the-registered-extensions)

The defining plug-in also evaluates the existing extension points. In the following example you create a handler class which can evaluate the existing extensions.

Create the following class.

```
package com.vogella.extensionpoint.definition;

import org.eclipse.core.runtime.CoreException;
import org.eclipse.core.runtime.IConfigurationElement;
import org.eclipse.core.runtime.IExtensionRegistry;
import org.eclipse.core.runtime.ISafeRunnable;
import org.eclipse.core.runtime.SafeRunner;
import org.eclipse.e4.core.di.annotations.Execute;

public class EvaluateContributionsHandler {
    private static final String IGREETER_ID =
            "com.vogella.extensionpoint.definition.greeter";
    @Execute
    public void execute(IExtensionRegistry registry) {
        IConfigurationElement[] config =
                registry.getConfigurationElementsFor(IGREETER_ID);
        try {
            for (IConfigurationElement e : config) {
                System.out.println("Evaluating extension");
                final Object o =
                        e.createExecutableExtension("class");
                if (o instanceof IGreeter) {
                    executeExtension(o);
                }
            }
        } catch (CoreException ex) {
            System.out.println(ex.getMessage());
        }
    }

    private void executeExtension(final Object o) {
        ISafeRunnable runnable = new ISafeRunnable() {
            @Override
            public void handleException(Throwable e) {
                System.out.println("Exception in client");
            }

            @Override
            public void run() throws Exception {
                ((IGreeter) o).greet();
            }
        };
        SafeRunner.run(runnable);
    }
}
```

The code above uses the `ISafeRunnable` interface. This interface protects the plug-in which defines the extension point from malfunction extensions. If an extension throws an `Exception`, it will be caught by `ISafeRunnable` and the remaining extensions will still get executed.

Review the _.exsd_ schema file and the reference to this file in the _plugin.xml_ file.

### [](#create-a-menu-entry-and-add-it-to-your-product)[7.7. Create a menu entry and add it to your product](#create-a-menu-entry-and-add-it-to-your-product)

Add a dependency to the `com.vogella.extensionpoint.definition` plug-in in the _MANIFEST.MF_ file of your application plug-in.

Afterwards create a new menu entry called _Evaluate extensions_ and define a command and handler for this menu entry. In the handler point to the `EvaluateContributionsHandler` class.

|  | A better approach would be to add the menu entry via a model fragment or a model processor, but I leave that as an additional exercise to the reader. |

### [](#providing-an-extension)[7.8. Providing an extension](#providing-an-extension)

Create a new simple plug-in called _com.vogella.extensionpoint.contribution_.

Open the _MANIFEST.MF_ editor of this new plug-in and select the _Dependencies_ tab.

Add the `com.vogella.extensionpoint.definition` and `org.eclipse.core.runtime` plug-ins as dependencies. Make sure your plug-in has the _This plug-in is a singleton_ flag set on the _Overview_ tab for your _MANIFEST.MF_ file.

![](https://www.vogella.com/tutorials/EclipseExtensionPoint/img/extensionpoint110.png)

Switch to the _Extensions_ tab and select **Add…​**. Select your custom extension point and press the **Finish** button.

![](https://www.vogella.com/tutorials/EclipseExtensionPoint/img/extensionpoint120.png)

Add a client to your extension point via right-click.

![](https://www.vogella.com/tutorials/EclipseExtensionPoint/img/extensionpoint130.png)

![](https://www.vogella.com/tutorials/EclipseExtensionPoint/img/extensionpoint140.png)

Create the `GreeterGerman` class with the following code.

```
package com.vogella.extensionpoint.contribution;

import com.vogella.extensionpoint.definition.IGreeter;

public class GreeterGerman implements IGreeter {

    @Override
    public void greet() {
        System.out.println("Moin, moin!");
    }
}
```

### [](#validate)[7.10. Validate](#validate)

Start your application via the product configuration file. This updates the plug-ins included in the run configuration.

In your running application select your new menu entry. The handler class write the output of your extensions to the _Console_ view of your Eclipse IDE.