---
description: >-
  The info importer can convert a variety of input types to Info instances.
  Common info sub-types are Java and Android classes plus a variety of supported file formats.
---

# InfoImporter

The info importer can import a `Info` from a `ByteSource`. Since `Info` defines `isClass/asClass` and `isFile/asFile` you can determine what sub-type the item is through those methods, or `instanceof` checks.

## Examples

Reading a class file:

```java
// Wrap input in ByteSource.
byte[] classBytes = Files.readAllBytes(Paths.get("HelloWorld.class"));
ByteSource source = ByteSources.wrap(classBytes);
// Parse into an info object.
Info read = importer.readInfo("HelloWorld", source);
// Cast to 'JvmClassInfo' with 'asX' methods
JvmClassInfo classInfo = read.asClass().asJvmClass();
// Or use instanceof
if (read instanceof JvmClassInfo classInfo) {
	// ...
}
```

Reading a generic file and doing an action based off the type of content it is _(Like text/images/video/audio/etc)_:

```java
// Wrap input in ByteSource.
byte[] textRaw = Files.readAllBytes(Paths.get("Unknown.dat"));
ByteSource source = ByteSources.wrap(textRaw);

// Parse into an info object.
Info read = importer.readInfo("Unknown.dat", source);

// Do action based on file type.
FileInfo readFile = read.asFile();
if (readFile.isTextFile()) {
	TextFileInfo textFile = readFile.asTextFile();
	String text = textFile.getText();
	// ...
} else if (readFile.isVideoFile()) {
	VideoFileInfo videoFile = readFile.asVideoFile();
	// ...
}
```