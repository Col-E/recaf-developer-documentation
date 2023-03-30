---
description: >-
  The resource importer can convert a variety of input types to
  WorkspaceResource instances.
---

# ResourceImporter

The resource importer can import a `WorkspaceResource` from a variety of inputs such as:

* `ByteSource` - Delegates to some data-source providing content as a `byte[]`
* `File` - Can point to a file, or directory
* `Path` - Can point to a file, or directory
* `URL` - Can point to any content source that can be streamed from.
* `URI` - Can point to any content source that can be streamed from.

## Examples of reading from different content types

When providing content from an in-memory source, `ByteSource` can be used:

```java
// Any content that can be represented in 'byte[]' can be wrapped into a 'ByteSource'
byte[] helloBytes = "Hello".getBytes(StandardCharsets.UTF_8);
ByteSource source = ByteSources.wrap(helloBytes);
WorkspaceResource resource = importer.importResource(source);

// The utility class 'ByteSources' has a number of helpful methods, example paths:
Path path = Paths.get("test.jar");
ByteSource source = ByteSources.forPath(path);
WorkspaceResource resource = importer.importResource(source);

// The utility class 'ZipCreationUtils' also may be useful if you want to easily
//  bundle multiple items together into one source.
// It can make a ZIP from a Map<String, byte[]> or from individual items by using
//  a builder pattern via 'ZipCreationUtils.builder()'.cocc
String name = "com/example/Demo";
byte[] bytes = ...
Map<String, byte[]> map = new LinkedHashMap<>();
map.put(name + ".class", bytes);
map.put(JarFileInfo.MULTI_RELEASE_PREFIX + "9/" + name + ".class", bytes);
map.put(JarFileInfo.MULTI_RELEASE_PREFIX + "10/" + name + ".class", bytes);
map.put(JarFileInfo.MULTI_RELEASE_PREFIX + "11/" + name + ".class", bytes);
byte[] zipBytes = ZipCreationUtils.createZip(map);
ByteSource zipSource = ByteSources.wrap(zipBytes);
WorkspaceResource resource = importer.importResource(zipSource);
```

When providing content from an on-disk source, using a `File` or `Path` reference can be used:

```java
// NIO Path
Path path = Paths.get("test.jar");
WorkspaceResource resource = importer.importResource(path);

// Old IO File
File file = new File("test.jar");
WorkspaceResource resource = importer.importResource(file);
```

When providing content from a URL/URI, content can be either on-disk or remote. So long as streaming for the URL scheme is supported:

```java
// URI from local file
URI uri = File.createTempFile("prefix", "test.zip").toURI();
WorkspaceResource resource = importer.importResource(uri);

// URL from a remote file
URL url = new URL("https://example.com/example.zip");
WorkspaceResource resource = importer.importResource(url);
```

