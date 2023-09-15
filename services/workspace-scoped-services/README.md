---
description: Services bound to the duration of a workspace being open.
---

# Workspace scoped services

These services are available for injection only while a workspace is open. When a workspace is closed you will not be able to inject instances of these types until the next workspace is opened.

## Core

These are the services defined in the `core` module.&#x20;

* AggregateMappingManager
* AstService
* CallGraph
* [InheritanceGraph](inheritancegraph.md)
* MappingApplier

## UI

These are no new services defined in the `ui` module.

