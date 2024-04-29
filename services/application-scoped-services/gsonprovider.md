---
description: >-
  This provider manages a shared Gson instance used by various services
  for serialization purposes.
---

# GsonProvider

The Gson provider manages a shared `Gson` instance and allows services and plugins to register custom JSON serialization support. This serves multiple purposes:

- JSON formatting is shared across services that use it for serialization
- Services can register custom serialization for private types _(See `KeybindingConfig` for an example of this)_
- Plugins that register custom config containers with `ConfigManager` can register support for their own custom types.

## Registering custom serialization

The Gson provider offers methods that allows registering the following:

- `TypeAdapter` - For handling serialization and deserialization of a type.
- `InstanceCreator` - For assisting construction of types that do not provide no-arg constructors.
- `JsonSerializer` - For specifying serialization support only.
- `JsonDeserializer` - For specifying deserialization support only.

For more details on how to use each of the given types, see the [Gson JavaDocs](https://www.javadoc.io/doc/com.google.code.gson/gson/latest/com.google.gson/com/google/gson/package-summary.html).

**TypeAdapter:**
```java
gsonProvider.addTypeAdapter(ExampleType.class, new TypeAdapter<>() {
    @Override
    public void write(JsonWriter out, ExampleType value) throws IOException {
        // Manual serialization of 'value'  goes here using 'out'
    }
    @Override
    public ExampleType read(JsonReader in) throws IOException {
        // Manual deserialization of the type from 'in' goes here
        return new ExampleType(...);
    }
});
```

**InstanceCreator:**
```java
gsonProvider.addTypeInstanceCreator(ExampleType.class, type -> new ExampleType(...));
```

**JsonSerializer:**
```java
gsonProvider.addTypeSerializer(ExampleType.class, (src, typeOfSrc, context) -> {
    // Manual serialization of 'T src' into a 'JsonElement' return value goes here
    return new JsonObject();
});
```

**JsonDeserializer:**
```java
gsonProvider.addTypeDeserializer(ExampleType.class, (json, typeOfT, context) -> {
    // Manual deserialization of (JsonElement json) goes here
    return new ExampleType(...);
});
```
