---
description: >-
  The decompile manager tracks supported decompilers, pre/post processing
  and provides some  ease-of-access decompilation calls
---

# DecompileManager

The decompile manager allows you to:

* Decompile a `JvmClassInfo` or `AndroidClassInfo` with:
  * The current preferred JVM or Android decompiler
  * A provided decompiler
* Register and unregister additional decompilers
* Pre-process classes before they are decompiled
* Post-process output from decompilers

Using the decompile calls in this manager will schedule the tasks in a shared thread-pool. Calling the decompile methods on the `Decompiler` instances directly is a blocking operation. If you want to decompile many items it would be best to take advantage of the manager due to the pool usage.

## Choosing a decompiler

There are a number of ways to grab a `Decompiler` instance. The operations are the same for JVM and Android implementations.

```java
// Currently configured target decompiler
JvmDecompiler decompiler = decompilerManager.getTargetJvmDecompiler();

// Specific decompiler by name
JvmDecompiler decompiler = decompilerManager.getJvmDecompiler("cfr");

// Any decompiler matching some condition, falling back to the target decompiler
JvmDecompiler decompiler = decompilerManager.getJvmDecompilers().stream()
                .filter(d -> d.getName().equals("procyon"))
                .findFirst().orElse(decompilerManager.getTargetJvmDecompiler());
```

## Decompiling a class

If you want to pass a specific decompiler, get an instance and pass it to the decompile functions provided by `DecompileManager`:

* `decompile(Workspace, JvmClassInfo)` - Uses the target decompiler
* `decompile(JvmDecompiler, Workspace, JvmClassInfo)` - Uses the specified decompiler

```java
JvmDecompiler decompiler = ...;

// Handle result when it's done.
decompilerManager.decompile(decompiler, workspace, classToDecompile)
        .whenComplete((result, throwable) -> {
            // Throwable thrown when unhandled exception occurs.
        });

// Or, block until result is given, then handle it in the same thread.
// Though at this point, you should really just invoke the decompile method on the
// decompiler itself rather than incorrectly use the pooled methods provided by
// the decompile manager.
DecompileResult result = decompilerManager.decompile(decompiler, workspace, classToDecompile)
    .get(1, TimeUnit.SECONDS);
```

## Pre-processing decompiler input

The decompiler manager allows you to register filters for inputs fed to decompilers. This allows you to modify the contents of classes in any arbitrary way prior to decompilation, without actually making a change to the class in the workspace.

Here is an example where we strip out all debug information _(Generics, local variable names, etc)_:
```java
JvmBytecodeFilter debugRemovingFilter = (workspace, classInfo, bytecode) -> {
    // The contents we return here will be what is fed to the decompiler, instead of the original class present in the workspace.
    ClassWriter writer = new ClassWriter(0);
    classInfo.getClassReader().accept(writer, ClassReader.SKIP_DEBUG);
    return writer.toByteArray();
};

// The filter can be added/removed to/from all decompilers by using the decompile manager.
decompilerManager.addJvmBytecodeFilter(debugRemovingFilter);
decompilerManager.removeJvmBytecodeFilter(debugRemovingFilter);

// If you want to only apply the filter to one decompiler, you can do that as well.
decompiler.addJvmBytecodeFilter(debugRemovingFilter);
decompiler.removeJvmBytecodeFilter(debugRemovingFilter);
```

## Post-processing decompiler output

Similar to pre-processing, the output of decompilation can be filtered via the decompiler manager's filters. They operate on the `String` output of decompilers.

```java
OutputTextFilter tabConversionFilter = (workspace, classInfo, code) -> {
    return code.replace("    ", "\t");
};

// Add/remove to/from all decompilers
decompilerManager.addOutputTextFilter(tabConversionFilter);
decompilerManager.removeOutputTextFilter(tabConversionFilter);

// Add/remove to/from a single decompiler
decompiler.addOutputTextFilter(tabConversionFilter);
decompiler.removeOutputTextFilter(tabConversionFilter);
```