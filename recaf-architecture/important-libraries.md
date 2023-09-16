---
description: A brief overview of the major dependencies Recaf uses in each module.
---

# Important libraries

## Core

**JVM Bytecode Manipulation**: Recaf uses [ASM](https://asm.ow2.io/) and [CafeDude](https://github.com/Col-E/CAFED00D) to parse bytecode. Most operations will be based on ASM since heavily abstracts away the class file format, making what would otherwise be tedious work simple. CafeDude is used for lower level operations and patching classes that are not compliant with ASM.

**Android to Java Conversion**: Recaf uses [dex-translator](https://github.com/Col-E/dex-translator/) to map the Dalvik bytecode of classes into JVM bytecode. This process is a bit lossy, but allows the use of JVM tooling _(like the different decompilers)_ on Android content.

**Android Dalvik Bytecode Manipulation**: We are currently investigating on how to handle Dalvik manipulation.

**ZIP Files**: Recaf uses [LL-Java-Zip](https://github.com/Col-E/LL-Java-Zip) to read ZIP files. The behavior of LL-Java-Zip is configurable and can mirror interpreting archives in different ways. This is important for Java reverse engineering since the JVM itself has some odd parsing quirks that most other libraries do not mirror. More information about this can be read on the LL-Java-Zip project page.

**Source Parsing**: Recaf uses [OpenRewrite](https://github.com/openrewrite/rewrite) to parse Java source code. The major reasons for choosing this over other more mainstream parsing libraries are that:

1. The AST model is error resilient. This is important since code Recaf is decompiling may not always yield perfectly correct Java code, especially with more intense forms of obfuscation. The ability to ignore invalid sections of the source while maintaining the ability to interact with recognizable portions is very valuable.
2. The AST model bakes in the type, when known, to all AST nodes. For a node such as a method reference, you can easily access the name of the reference, the method descriptor of the reference, and the owning class defining the method. This information is what all of our context-sensitive actions must have access to in order to function properly.
3. The AST supports easy source transformation options. In the past if a user wanted to remap a class or member, we would apply the mapping, decompile the mapped class, then replace the text contents with the new decompilation. This process can be slower on larger classes due to longer decompilation times. If we can skip that step and instead instantly transform the AST to update the text we can save a lot of time in these cases.
4. The AST supports code formatting. We can allow the user to apply post-processing to decompiled code to give it a uniform style of their choosing, or allow them to format code to that style on demand with a keybind.

**CDI**: Recaf uses [Weld](https://weld.cdi-spec.org/) as its CDI implementation. You can read the [CDI](cdi.md) article for more information.

## UI

**JavaFX**: Recaf uses [JavaFX](https://openjfx.io/) as its UI framework. The observable property model it uses makes managing live updates to UI components when backing data is changed easy. Additionally, it is styled via CSS which makes customizing the UI for Recaf-specific operations much more simple as opposed to something like Swing.

**AtlantaFX**: Recaf uses [AtlantaFX](https://github.com/mkpaz/atlantafx) to handle common theming.

**Ikonli**: Recaf uses [Ikonli](https://github.com/kordamp/ikonli) for scalable icons. Specifically the [Carbon](https://kordamp.org/ikonli/cheat-sheet-carbonicons.html) pack.

**Docking**: Recaf uses [Tiwul-FX's docking](https://github.com/panemu/tiwulfx-dock) framework for handling dockable tabs across all open windows.
