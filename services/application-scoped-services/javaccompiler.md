---
description: >-
  The javac compiler allows you to compile Java source code.
---

# JavacCompiler

The `javac` compiler is just a service wrapping the standard JDK `javac` API with some quality of life improvements. All the arguments for the compiler are created through the `JavacArgumentsBuilder` type.

## Examples

The following samples can be used within this sample script:

```java
@Dependent
public class CompilerScript {
    private final JavacCompiler javac;

    @Inject
    public CompilerScript(JavacCompiler javac) {
        this.javac = javac;
    }

    public void run() {
        // code goes here
    }
}
```

### Compiling "Hello World"

The most common case, taking some source code and compiling it. You are required to specify the name of the class being compiled as an internal name _(Example: `com/example/Foo`)_ and the source. Any extra arguments are optional.

```java
// Compile a 'hello world' application
JavacArguments arguments = new JavacArgumentsBuilder()
        .withClassName("HelloWorld")
        .withClassSource("""
                public class HelloWorld {
                    public static void main(String[] args) {
                        System.out.println("Hello world");
                    }
                }""")
        .build();

// Run compiler and handle results
CompilerResult result = javac.compile(arguments, null, null);
if (result.wasSuccess()) {
    CompileMap compilations = result.getCompilations();
    compilations.forEach((name, bytecode) -> {
        // Do something with name/bytecode pair
    });
}
```

### Handling compiler feedback/errors

Compiler feedback is accessible from the returned `CompilerResult` as `List<CompilerDiagnostic> getDiagnostics()`.

```java
result.getDiagnostics().forEach(diagnostic -> {
    if (diagnostic.level() != CompilerDiagnostic.Level.ERROR) return;
    System.err.println(diagnostic.line() + ":" + diagnostic.column() + " --> " + diagnostic.message());
});
```

### Changing the compiled bytecode target version

Adapting the setup from before, you can change the target bytecode version via `withVersionTarget(int)`. This takes the release version of Java you wish to target. This is equivalent to `javac --release N` where `N` i the version. Because this uses the JDK environment you ran Recaf with the supported versions here are tied to what `javac` supports.

```java
// Compile a 'hello world' application against Java 11
int version = 11;
JavacArguments arguments = new JavacArgumentsBuilder()
        .withVersionTarget(version)
        .withClassName("HelloWorld")
        .withClassSource("""
                public class HelloWorld {
                    public static void main(String[] args) {
                        System.out.println("Hello world");
                    }
                }""")
        .build();
```

### Downsampling the compiled bytecode instead of directly targeting it

Alternatively you may want to downsample compiled code instead of targeting that version from within the compiler. This allows you to use new language features while still targeting older versions of Java.

```java
// Compile a 'hello world' application but downsample it to an older version
JavacArguments arguments = new JavacArgumentsBuilder()
        .withDownsampleTarget(8) // Downsample to Java 8
        .withClassName("HelloWorld")
        .withClassSource("""
                public class HelloWorld {
                    public static void main(String[] args) {
                        System.out.println(message());
                    }
                    
                    private static String message() {
                        int r = new java.util.Random().nextInt(5);
                        
                        // Using switch expressions, which do not exist in Java 8
                        return switch (r) {
                            case 0 -> "Zero";
                            case 1 -> "One";
                            case 2 -> "Two";
                            default -> "Three or more";
                        };
                 }
                }""")
        .build();
```

### Compiling code with references to classes in the `Workspace`

All you need to do is call `compile(JavacArguments arguments, Workspace workspace, JavacListener listener)` with a non-null `Workspace` instance. This will automatically include it as a classpath entry, allowing you to compile code referencing types defined in the workspace.

There is also `compile(JavacArguments arguments, Workspace workspace, List<WorkspaceResource> supplementaryResources, JavacListener listener)` which allows you to supply extra classpath data without adding it to the workspace.

### Compiling code with debug info enabled

You can enable compiling with debug information by specifying `true` to the debug `with` operations in the arguments builder.

```java
JavacArguments arguments = new JavacArgumentsBuilder()
        .withDebugLineNumbers(true)
        .withDebugSourceName(true)
        .withDebugVariables(true)
```

### Loading and executing the compiled code

The `CompileMap` you get out of the `CompilerResult` is an implementation of `Map<String, byte[]>`. You can thus use the compilation map directly in a utility like [`ClassDefiner`](../../utilities/classdefiner.md). Using the hello world classes from the above examples:

```java
CompileMap compilations = result.getCompilations();
ClassDefiner definer = new ClassDefiner(compilations);
try {
    Class<?> helloWorld = definer.findClass("HelloWorld");
    Method main = helloWorld.getDeclaredMethod("main", String[].class);
    main.invoke(null, (Object) new String[0]);
} catch (Exception ex) {
    ex.printStackTrace();
}
```