---
description: The config manager tracks all ConfigContainer instances across Recaf
---

# ConfigManager

The config manager allows you to:

* Iterate over all `ConfigContainer` instances across Recaf
* Register and unregister your own `ConfigContainer` values
  * Useful for plugin developers who want to expose config values in the UI
* Register and unregister listeners which are notified when new `ConfigContainer` values are registered and unregistered.

## Iterating over registered containers

```java
for (ConfigContainer container : configManager.getContainers())
   logger.info("Container group={}, id={}", container.getGroup(), container.getId());
```

## Registering and unregistering new containers

To add content to the config window create a `ConfigContainer` instance with some `ConfigValue` values and register it in the config manager. The config window is configured to listen to new containers and add them to the UI.

```java
// If you have your own class to represent config values,
//  you will probably want to extend from 'BasicConfigContainer' and add values
//  in the class's constructor via 'addValue(ConfigValue);'
//  You can reference most existing config classes for examples of this setup.
ConfigContainer container = ...
configManager.registerContainer(container);
configManager.unregisterContainer(container);
```

When creating a `ConfigContainer` class, it generally would be easiest to extend `BasicConfigContainer` and then use the `addValue(ConfigValue)` method.

```java
public class MyThingConfig extends BasicConfigContainer {
    private final ObservableString value = new ObservableString(null);

    @Inject
    public MyConfig() {
        // Third party plugins should use 'EXTERNAL' as their group. 
        // This special group is treated differently in the config window UI,
        // such that the ID's specified are text literals, and not translation keys.
        super(ConfigGroups.EXTERNAL, "My thing config");
        
        // Add values
        addValue(new BasicConfigValue<>("My value", String.class, value));
    }

    public ObservableString getValue() {
        return value;
    }
}
```

Internal services within Recaf define their configs as `ApplicationScoped` so that they are discoverable by the manager when the manager is initialized. This allows all services to feed their configs into the system when the application launches.

## Listening for new containers

```java
configManager.addManagedConfigListener(new ManagedConfigListener() {
    @Override
    public void onRegister(@Nonnull ConfigContainer container) {
        logger.info("Registered container: {} with {} values", 
                container.getGroupAndId(), container.getValues().size());
    }
    @Override
    public void onUnregister(@Nonnull ConfigContainer container) {
        logger.info("Unregistered container: {} with {} values",
                container.getGroupAndId(), container.getValues().size());
    }
});
```
