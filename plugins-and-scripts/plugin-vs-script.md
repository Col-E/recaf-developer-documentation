---
description: >-
  Plugins allow you to more easily manage creating complex behaviors, where
  scripts are for short one-shot actions.
---

# Plugin vs Script

## What is a plugin?

A plugin is a class that implements the interface `software.coley.recaf.plugin.Plugin`. When Recaf launches it looks in the plugins directory for JAR files that contain these plugin classes. It will then attempt to load and initialize them. Because a plugin is distributed as a JAR file a plugin developer can create complex logic and organize it easily across multiple classes in the JAR.&#x20;

An example use case for a plugin that a script cannot satisfy would be integrating with [services](broken-reference) defined in the UI module to add additional capabilities to the UI.

## What is a script?

A script is a single Java source file that is executed by users whenever they choose. They can be written as a full class to support similar capabilities to plugins, or in a short-hand that offers automatic imports of most common utility classes, but no access to services.

### Full class script

A full class script is just a regular class that defines a non-static void `run()`. The `run()` method is called whenever the user executes the script.

```java
// ==Metadata==
// @name Hello world
// @description Prints hello world
// @version 1.0.0
// @author Author
// ==/Metadata==

class MyScript {
    // must define 'void run()'
    void run() {
        System.out.println("hello");
    }
}
```

You can access any of Recaf's services by declaring a constructor annotated with `@Inject`. For more information see the article on [using services in plugins and scripts](using-services-in-plugins-and-scripts.md).

### Shorthand script

A shorthand script lets you write your logic without needing to declare a class and `run()` method. These shorthand scripts are given a variable reference to the current workspace, and a SLF4J logger. You can access the current workspace as `workspace` and the logger as `log`.

```java
// ==Metadata== 
// @name What is open?
// @description Prints what kinda workspace is open
// @version 1.0.0
// @author Author
// ==/Metadata==

String name = "(empty)";
if (workspace != null)
    name = workspace.getClass().getSimpleName();
log.info("Workspace = {}", name);
```

Another example working with the provided `workspace`:

```java
// Print out all enum names in the current workspace, if one is open.
if (workspace == null) return;
workspace.findClasses(Accessed::hasEnumModifier).stream()
    .map(c -> c.getValue().getName())
    .forEach(name -> log.info("Enum: {}", name));
```
