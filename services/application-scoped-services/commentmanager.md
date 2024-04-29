---
description: >-
  The comment manager maintains a database of comments for classes,
  fields and methods. These comments can be accessed programmatically
  with path nodes. Comments are automatically injected into the decompile process.
---

# CommentManager

The comment manager allows you to:

* Access all comments within a `Workspace` via `WorkspaceComments`
  * Look up comment information on a class, field or method via `PathNode`
  * Iterate over commented classes as `ClassComments`
  * Add, update, or remove comments utilizing `ClassComments`
* Create listeners which are notified when:
  * New comment containers *(per class)* are created
  * Comments are updated

Comments are displayed in the UI by injecting them into the decompilation process. Before a class is sent to the decompiler, it is intercepted and unique annotations are inserted at commented locations. After the decompiler yields output the unique annotations are replaced with the comments they are associated with. The comments are stored in the Recaf directory, no modifications are made to the contents of the `Workspace` when adding/editing/removing comments.

## Getting the desired `WorkspaceComments`

Recaf can store comments across multiple `Workspace` instances. So how do you get the comments from the one you want?

Assuming Recaf has a `Workspace` already opened and you want the comments of the current workspace:
```java
// Will be 'null' if no workspace is open in Recaf
WorkspaceComments workspaceComments = commentManager.getCurrentWorkspaceComments();
```

If you have a `Workspace` instance you want to get the comments of:
```java
Workspace workspace = ... // Get or create a workspace here
WorkspaceComments workspaceComments = commentManager.getOrCreateWorkspaceComments(workspace);
```

> **Note:** Comments in relationship to a `Workspace` are stored by a unique identifier associated with the `Workspace`. The identifier is pulled from the workspace's primary resource.
> - `WorkspaceResource` instances loaded from file paths use the file path as the unique ID.
> - `WorkspaceResource` instances loaded from directory paths use the directory path as the unique ID.
> - `WorkspaceResource` instances loaded from a remove agent connection use the remote [VM's identifier](https://docs.oracle.com/javase/8/docs/jdk/api/attach/spec/com/sun/tools/attach/VirtualMachine.html#id--) as the unique ID.

## Getting an existing comment

For classes:
```java
ClassPathNode classPath = workspace.findClass("com/example/Foo");

// Option 1: Lookup via generic PathNode
String comment = workspaceComments.getComment(classPath);

// Option 2: Lookup via ClassPathNode
String comment = workspaceComments.getClassComment(classPath);

// Option 3: Lookup the container of the class, then its comment
ClassComments classComments = workspaceComments.getClassComments(classPath);
if (classComments != null)
    String comment = classComments.getClassComment();
```

For fields and methods:
```java
ClassPathNode classPath = workspace.findClass("com/example/Foo");
ClassMemberPathNode memberPath = preMappingPath.child("stringField", "Ljava/lang/String;");

// Option 1: Lookup via generic PathNode
String comment = workspaceComments.getComment(memberPath);

// Option 2: Lookup the container of the class, then its comment
ClassComments classComments = workspaceComments.getClassComments(classPath);
if (classComments != null) {
    String comment = classComments.getMethodComment(memberPath.getValue());
    // You can also pass strings for the name/descriptor
    String comment = getMethodComment("stringField", "Ljava/lang/String;");
}
```

## Getting all existing comments

A `ClassComments` by itself does not expose a reference to the `ClassPathNode` it is associated with. The instances created by the comment manager during runtime do keep a reference though, so using `instanceof DelegatingClassComments` will allow you to get the associated `ClassPathNode` and in turn iterate over the class's fields and methods.
```java
WorkspaceComments workspaceComments = commentManager.getCurrentWorkspaceComments();
for (ClassComments classComments : workspaceComments) {
    String classComment = classComments.getClassComment();
    // This subtype of class comment container records the associated class path.
    if (classComments instanceof DelegatingClassComments delegatingClassComments) {
        ClassPathNode classPath = delegatingClassComments.getPath();
        // We can iterate over the fields/methods held by the path's class
        ClassInfo classInfo = classPath.getValue();
        for (FieldMember field : classInfo.getFields()) {
            String fieldComment = classComments.getFieldComment(field);
        }
        for (MethodMember method : classInfo.getMethods()) {
            String fieldComment = classComments.getMethodComment(method);
        }
    }
}
```

## Adding / editing / removing comments

Adding and modifying comments is done by `set` methods on a `ClassComments` instance. Comments can be removed by passing `null` as the comment parameter.
```java
ClassPathNode classPath = workspace.findClass("com/example/Foo");
ClassComments classComments = workspaceComments.getOrCreateClassComments(classPath);

// Adding a comment to the class
classComments.setClassComment("class comment");

// Removing the comment
classComments.setClassComment(null);

// Adding a comment to a field in the class
classComments.setFieldComment("stringField", "Ljava/lang/String;", "Field comment");

// Adding a comment to a method in the class
classComments.setMethodComment("getStringField", "()Ljava/lang/String;", "Method comment");
```

## Listening for new comments

If you want to track where comments are being made, you can register a `CommentUpdateListener`:
```java
commentManager.addCommentListener(new CommentUpdateListener() {
    // Only implementing the class comment method, the same idea can
    // be applied to the field and method listener calls.
    @Override
    public void onClassCommentUpdated(@Nonnull ClassPathNode path, @Nullable String comment) {
        if (comment == null)
            logger.info("Comment removed on class '{}'", path.getValue().getName());
        else
            logger.info("Comment updated on class '{}'", path.getValue().getName());
    }
});
```