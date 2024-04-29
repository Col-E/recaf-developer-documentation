---
description: >-
  Allows defining classes at runtime from raw bytes.
---

# ClassDefiner

The `ClassDefiner` is a utility which takes in one or more classes as a `Map<String, byte[]>`. Keys are the class name format you'd use for `Class.forName(String)` and thus would look like `java.lang.String`. Values are the raw bytes of the class file.

## Usage

```java
String name = "com/example/StringMapper"; // Internal name format
String sourceName = name.replace('/', '.'); // Source name format

// Example code to generate an interface
ClassWriter cw = new ClassWriter(0);
cw.visit(V1_8, ACC_PUBLIC | ACC_INTERFACE, name, "java/lang/Object", null, null);
cw.visitMethod(ACC_PUBLIC, "map", "(Ljava/lang/String;)Ljava/lang/String;", null, null);
cw.visitEnd();
byte[] bytes = cw.toByteArray();

// Definer with a single class
ClassDefiner cd = new ClassDefiner(sourceName, bytes);

// Definer with a map of entries
ClassDefiner cd = new ClassDefiner(Map.of(sourceName, bytes));
```