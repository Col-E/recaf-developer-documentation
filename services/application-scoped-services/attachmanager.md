---
description: >-
  The attach manager outlines capabilities pertaining to Java's attach API for
  loading agents in remote processes.
---

# AttachManager

The `AttachManager` allows you to:&#x20;

* Inspect what JVM processes are running on the current machine
  * Get value of `System.getProperties(`) of the remote JVM
  * Get the name of the remote JVM's starting main class _(Entry point)_&#x20;
  * Get JMX bean information from the remote JVM
  * Register listeners for when new JVM processes start
* Connect to these JVM processes and represent their content as a `WorkspaceRemote`&#x20;

## Inspecting available JVMs

The Attach API lists available JVM's via `VirtualMachine.list()`. `AttachManager` builds on top of that, offering easier access to each available JVM's properties, starting main class, and JMX beans.

By default, Recaf does not scan the system for running JVM's unless the attach window is open. The refresh rate for scans is once per second. When a new JVM is found Recaf queries it for the information listed above and caches the results. These operations typically involve a bit of tedious error handling and managing the connection state to the remote JVM, but now all you need is just a single call to one of `AttachManager`'s getters.

### Example: Manual scans & printing discovered JVMs

```java
// Register listener to print the number of new/old VM's on the system
//  - Each parameter is Set<VirtualMachineDescriptor>
attachManager.addPostScanListener((added, removed) -> {
    logger.info("Update: {} new instances, {} instances were closed",
        added.size(), removed.size());
});
// Scan for new JVMs every second
Executors.newScheduledThreadPool(1)
    .scheduleAtFixedRate(attachManager::scan, 0, 1, TimeUnit.SECONDS);
```

### Example: Get information of a remote JVM

```java
// The 'descriptor' is a VirtualMachineDescriptor, which we can listen for new values of
// by looking at the example above.
int pid = attachManager.getVirtualMachinePid(descriptor);
Properties remoteProperties = attachManager.getVirtualMachineProperties(descriptor);
String mainClass = attachManager.getVirtualMachineMainClass(descriptor);
```

### Example: Access JMX bean information of a remote JVM

```java
// Recaf has a wrapper type for the JMX connection which grants one-liner access to common beans.
JmxBeanServerConnection jmxConnection = attachManager.getJmxServerConnection(descriptor);

// Available beans
MBeanInfo beanClassLoading = jmxConnection.getClassloadingBeanInfo();
MBeanInfo beanCompilation = jmxConnection.getCompilationBeanInfo();
MBeanInfo beanOperatingSystem = jmxConnection.getOperatingSystemBeanInfo();
MBeanInfo beanRuntime = jmxConnection.getRuntimeBeanInfo();
MBeanInfo beanThread = jmxConnection.getThreadBeanInfo();
MBeanInfo beanOperatingSystem = jmxConnection.getOperatingSystemBeanInfo();
MBeanInfo beanOperatingSystem = jmxConnection.getOperatingSystemBeanInfo();

// Iterating over bean contents
MBeanAttributeInfo[] attributes = beanRuntime.getAttributes();
try {
    for (MBeanAttributeInfo attribute : attributes) {
        String name = attribute.getName();
        String description = attribute.getDescription();
        Object value = beanInfo.getAttributeValue(jmxConnection, attribute);
        logger.info("{} : {} == {}", name, description, value);
    }
} catch (Exception ex) {
    logger.error("Failed to retrieve attribute values", ex);
}
```

## Connecting to remote JVMs

To interact with remote JVMs instrumentation capabilities Recaf initializes a small TCP server in the remote JVM using [Instrumentation Server](https://github.com/Col-E/InstrumentationServer/). The `WorkspaceRemoteVmResource` type wraps the client instance that interacts with this server. To connect to the remote server you need to call `connect()` on the created `WorkspaceRemoteVmResource` value.

```java
// Once we create a remote resource, we call 'connect' to activate the remote server in the process.
// For 'WorkspaceRemoteVmResource' usage, see the section under the workspace model category.
WorkspaceRemoteVmResource vmResource = attachManager.createRemoteResource(descriptor);
vmResource.connect();

// Can set the current workspace to load the remote VM in the UI.
workspaceManager.setCurrent(new BasicWorkspace(vmResource));
```
