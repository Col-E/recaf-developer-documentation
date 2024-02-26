---
description: >-
  Services are provided to plugins and scripts via CDI injection. Some services
  are only available when a workspace is open in Recaf.
---

# Using services in plugins & scripts

## Plugins

Plugins can use services by annotating the class with `@Dependent` and annotating the constructor with `@Inject`. For services that are application scoped, see this example:

```java
import jakarta.enterprise.context.Dependent;
import jakarta.inject.Inject;
import software.coley.recaf.plugin.*;
import software.coley.recaf.services.workspace.WorkspaceManager;

// Dependent is a CDI annotation which loosely translates to being un-scoped.
// Plugin instances are managed by Recaf so the scope is bound to when plugins are loaded in practice.
@Dependent
class MyPlugin implements Plugin {
    private final WorkspaceManager workspaceManager;

    // Example, injecting the 'WorkspaceManager' service which is '@ApplicationScoped'
    @Inject
    public MyPlugin(WorkspaceManager workspaceManager) {
        this.workspaceManager = workspaceManager;
    }

    @Override
    public void onEnable() { ... }

    @Override
    public void onDisable() { ... }
}
```

For services that are `@WorkspaceScoped` they will not be available on plugin initialization since no `Workspace` will be set when this occurs. You can use CDI constructs like `Instance<T>` to work around this. As an example here is a basic plugin that injects the `InheritanceGraph` service:

```java
import jakarta.enterprise.context.Dependent;
import jakarta.enterprise.inject.Instance;
import jakarta.inject.Inject;
import software.coley.recaf.plugin.*;
import software.coley.recaf.services.inheritance.InheritanceGraph;
import software.coley.recaf.services.workspace.WorkspaceManager;

@Dependent
class MyPlugin implements Plugin {
    // We will use the workspace manager to listen to when new workspaces are opened.
    // When this occurs we can access instances of workspace scoped services.
    @Inject
    public MyPlugin(WorkspaceManager workspaceManager, 
                Instance<InheritanceGraph> graphProvider) {
        // No workspace open, wait until one is opened by the user.
        if (workspaceManager.getCurrent() == null) {
            workspaceManager.addWorkspaceOpenListener(newWorkspace -> {
                // This will be called AFTER the 'newWorkspace' value has been assigned
                // as the 'current' workspace in the workspace manager.
                // At this point, all workspace scoped services are re-allocated by CDI
                // to target the newly opened workspace.
                //
                // Thus, we can get our inheritance graph of the workspace here.
                InheritanceGraph graph = graphProvider.get();
            });
        } else {
            // There is a workspace, so we can immediately get the graph for the current workspace.
            InheritanceGraph graph = graphProvider.get();
        }
    }

    @Override
    public void onEnable() { ... }

    @Override
    public void onDisable() { ... }
}
```

## Scripts

Scripts are ran when a user requests them, so you _generally_ do not need to care about whether a service is `@ApplicationScoped` or `@WorkspaceScoped`. The assumption is the user will run the script when it is needed. So a script that uses workspace-scoped content will only be used when a workspace is opened. Of course, if the script is going to load a new workspace, then you will need to follow the same process as described for plugins when using workspace scoped services.

Simple scripts do not use services. Scripts using the full class form will be able to use services.

```java
// ==Metadata==
// @name Content loader
// @description Script to load content from a pre-set path.
// @version 1.0.0
// @author Col-E
// ==/Metadata==

import jakarta.enterprise.context.Dependent;
import jakarta.inject.Inject;
import org.slf4j.Logger;
import software.coley.recaf.analytics.logging.Logging;
import software.coley.recaf.info.JvmClassInfo;
import software.coley.recaf.services.workspace.WorkspaceManager;
import software.coley.recaf.services.workspace.io.ResourceImporter;
import software.coley.recaf.workspace.model.BasicWorkspace;
import software.coley.recaf.workspace.model.Workspace;
import software.coley.recaf.workspace.model.resource.WorkspaceResource;
import java.nio.file.Paths;

@Dependent
public class LoadContentScript {
    private static final Logger logger = Logging.get("load-script");
    private final ResourceImporter importer;
    private final WorkspaceManager workspaceManager;

    // We're injecting the importer to load 'WorkspaceResource' instances from paths on our system
    // then we use the workspace manager to set the current workspace to the loaded content.
    @Inject
    public LoadContentScript(ResourceImporter importer, WorkspaceManager workspaceManager) {
        this.importer = importer;
        this.workspaceManager = workspaceManager;
    }

    // Scripts following the class model must define a 'void run()'
    public void run() {
        String path = "C:/Samples/Test.jar";
        try {
            // Load resource from path, wrap it in a basic workspace
            WorkspaceResource resource = importer.importResource(Paths.get(path));
            Workspace workspace = new BasicWorkspace(resource);

            // Assign the workspace so the UI displays its content
            workspaceManager.setCurrent(workspace);
        } catch (Exception ex) {
            logger.error("Failed to read content from '{}' - {}", path, ex.getMessage());
        }
    }
}
```
