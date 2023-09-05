---
description: Services bound to the duration of a workspace being open.
---

# Workspace scoped services

These services are available for injection only while a workspace is open. When a workspace is closed you will not be able to inject instances of these types until the next workspace is opened.

## API

These are the services outlined in the `api` module.&#x20;

* AggregateMappingManager
* AstService
* CallGraph
* [InheritanceGraph](inheritancegraph.md)
* MappingApplier

## Core

The `core` module defines no new service types.

## UI

These are the services outlined in the `ui` module.

* AstService
