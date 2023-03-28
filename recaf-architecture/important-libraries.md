---
description: A brief overview of the major dependencies Recaf uses in each module.
---

# Important libraries

## API

**JVM Bytecode Manipulation**: Recaf uses [ASM](https://asm.ow2.io/) and [CafeDude](https://github.com/Col-E/CAFED00D) to parse bytecode. Most operations will be based on ASM since heavily abstracts away the class file format, making what would otherwise be tedious work simple. CafeDude is used for lower level operations and patching classes that are not compliant with ASM.

**Android Dalvik Bytecode Manipulation**: Recaf uses [DexLib & Smali](https://github.com/JesusFreke/smali/) to parse and manipulate Android bytecode.

**ZIP Files**: Recaf uses [LL-Java-Zip](https://github.com/Col-E/LL-Java-Zip) to read ZIP files. We do not use the standard `java.util.zip` classes because it does allow you to choose how you read ZIP files. Most ZIP parsing software scans for ZIP headers starting at the beginning of the file going forward. However when reading from JAR files the JVM scans at the end going backwards. This can lead to a difference in what you see in common ZIP tools vs what actually gets loaded by the JVM at runtime. In order to allow users to choose which behavior they want when reading from a ZIP we need to use our own library for that.

**Source Parsing**: Recaf uses [OpenRewrite](https://github.com/openrewrite/rewrite) to parse Java source code. The major reasons for choosing this over other more mainstream parsing libraries are that:

1. The AST model is error resilient. This is important since code Recaf is decompiling may not always yield perfectly correct Java code, especially with more intense forms of obfuscation. The ability to ignore invalid sections of the source while maintaining the ability to interact with recognizable portions is very valuable.
2. The AST model bakes in the type, when known, to all AST nodes. For a node such as a method reference, you can easily access the name of the reference, the method descriptor of the reference, and the owning class defining the method. This information is what all of our context-sensitive actions must have access to in order to function properly.
3. The AST supports easy source transformation options. In the past if a user wanted to remap a class or member, we would apply the mapping, decompile the mapped class, then replace the text contents with the new decompilation. This process can be slower on larger classes due to longer decompilation times. If we can skip that step and instead instantly transform the AST to update the text we can save a lot of time in these cases.
4. The AST supports code formatting. We can allow the user to apply post-processing to decompiled code to give it a uniform style of their choosing, or allow them to format code to that style on demand with a keybind.

## Core

**CDI**: Recaf uses [Weld](https://weld.cdi-spec.org/) as its CDI implementation. You can read the [CDI](cdi.md) article for more information.

## UI

**JavaFX**: Recaf uses [JavaFX](https://openjfx.io/) as its UI framework. The observable property model it uses makes managing live updates to UI components when backing data is changed easy. Additionally, it is styled via CSS which makes customizing the UI for Recaf-specific operations much more simple as opposed to something like Swing.

**AtlantaFX**: Recaf uses [AtlantaFX](https://github.com/mkpaz/atlantafx) to handle common theming.

**Ikonli**: Recaf uses [Ikonli](https://github.com/kordamp/ikonli) for scalable icons. Specifically the [Carbon](https://kordamp.org/ikonli/cheat-sheet-carbonicons.html) pack.

**Docking**: Recaf uses [Tiwul-FX's docking](https://github.com/panemu/tiwulfx-dock) framework for handling dockable tabs across all open windows.
