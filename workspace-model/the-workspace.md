---
description: A workspace is the outline of one or more input files.
---

# The workspace

<figure><img src="../.gitbook/assets/Workspace-model.png" alt=""><figcaption></figcaption></figure>

**Legend**

1. The primary resource is the input that is targeted for editing. If you drag and drop a single JAR file into Recaf, then this will represent that JAR file. The representation is broken down into pieces...
2. The JVM class bundle contains all the `.class` files in the input that are not treated specially by the JVM.
3. JAR files allow you to have multiple versions of the same class for different JVM versions via [_"Multi-release JAR"_](https://www.baeldung.com/java-multi-release-jar). This is a map of JVM version numbers to bundles of classes associated with that specific version.
4. Android's APK files may contain multiple containers of classes known as DEX files. This is a mapping of each DEX file to the classes contained within it.
5. The file bundle contains all other regular files that are not ZIP archives.
6. ZIP archives are represented as embedded resources, allowing a ZIP in a ZIP, or JAR in a JAR, to be navigable within Recaf.
7. Workspaces can have multiple inputs. These additional inputs can be used to enhance performance of some services such as inheritance graphing, recompilation, and SSVM virtualization just to name a few. These supporting resources are not intended to be editable and are just there to _"support"_ services as described before.
8. Recaf adds a few of its own supporting resources, but manages them separately from the supporting resources list.
9. The runtime resource allows Recaf to access classes in the current JVM process like `java.lang.String`.
10. The phantom resource analyzes content in the primary and supporting resources when the workspace is first created and then generates missing references. This allows services like recompilation to compile against libraries that are not directly provided as a supporting resource. This generally works well clean inputs but often fails when obfuscated code is present, which isn't that bad since you shouldn't be recompiling against obfuscated code anyway.

## Loading workspaces

You will typically be using the `BasicWorkspace` implementation of `Workspace` for most operations. Creating one of these requires a primary resource, and optionally a collection of supporting resources. You can load resources via `ResourceImporter`.

## Exporting workspaces

You will create an instance of `WorkspaceExportOptions` and configure it however you like. Then you can call `WorkspaceExportOptions.create()` to create a `WorkspaceExporter` instance. The `WorkspaceExporter` is re-usable and exports workspaces via `WorkspaceExporter.export(Workspace)`. There are also helper methods in `WorkspaceManager` if you prefer to use those.

## Listeners

The `WorkspaceManager` allows you to register listeners for multiple workspace events.

* `WorkspaceOpenListener`: When a new workspace is opened within the manager.
* `WorkspaceCloseListener`: When a prior workspace is closed within the manager.
* `WorkspaceModificationListener`: When the active workspace's model is changed _(Supporting resource added/removed)_

When creating services and CDI enabled classes, you can annotate the class with `@AutoRegisterWorkspaceListeners` to automatically register and unregister the class based on what is necessary for the CDI scope.

## Finding classes/files in the workspace

The `Workspace` interface defines some `find` operations allowing for simple name look-ups of classes and files.

| Method                                                                             | Usage                                                                                                                                                                                       |
| ---------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ClassPathNode findClass(String internalName)`                                     | Finds the first available `ClassInfo` by the given name, and wraps it in a `ClassPathNode`.                                                                                                 |
| `ClassPathNode findJvmClass(String internalName)`                                  | Finds the first available `JvmClassInfo` by the given name, and wraps it in a `ClassPathNode`.                                                                                              |
| `ClassPathNode findLatestVersionedJvmClass(String internalName)`                   | Finds the most up-to-date `JvmClassInfo` from all available versioned bundles, wrapping it in a `ClassPathNode`.                                                                            |
| `ClassPathNode findVersionedJvmClass(String internalName, int version)`            | Finds the first available `JvmClassInfo` matching the given version _(Floored to next available older version)_, and wraps it in a `ClassPathNode`                                          |
| `ClassPathNode findAndroidClass(String internalName)`                              | Finds the first available `AndroidClassInfo` by the given name, and wraps it in a `ClassPathNode`.                                                                                          |
| `DirectoryPathNode findPackage(String name)`                                       | Finds the first available `ClassInfo` defined in the given package, or any sub-package, then wraps the path in a `DirectoryPathNode`.                                                       |
| `SortedSet<ClassPathNode> findClasses(Predicate<ClassInfo> filter)`                | Collects all `ClassInfo` values in the workspace that match the given predicate, and wraps each in a `ClassPathNode`. The returned set ordering for paths is alphabetic order.              |
| `SortedSet<ClassPathNode> findJvmClasses(Predicate<JvmClassInfo> filter)`          | Collects all `JvmClassInfo` values in the workspace that match the given predicate, and wraps each in a `ClassPathNode`. The returned set ordering for paths is alphabetic order.           |
| `SortedSet<ClassPathNode> findVersionedJvmClasses(Predicate<JvmClassInfo> filter)` | Collects all versioned `JvmClassInfo` values in the workspace that match the given predicate, and wraps each in a `ClassPathNode`. The returned set ordering for paths is alphabetic order. |
| `SortedSet<ClassPathNode> findAndroidClasses(Predicate<AndroidClassInfo> filter)`  | Collects all `AndroidClassInfo` values in the workspace that match the given predicate, and wraps each in a `ClassPathNode`. The returned set ordering for paths is alphabetic order.       |
| `FilePathNode findFile(String filePath)`                                           | Finds any available `FileInfo` by the given name, and wraps it in a `FilePathNode`.                                                                                                         |
| `SortedSet<FilePathNode> findFiles(Predicate<FileInfo> filter)`                    | Collects all `FileInfo` values in the workspace that match the given predicate, and wraps each in a `FilePathNode`. The returned set ordering for paths is alphabetic order.                |
