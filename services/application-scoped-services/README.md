---
description: Services bound to the duration of Recaf being open.
---

# Application scoped services

These services are available for injection at any point.

## API

These are the services outlined in the `api` module. Some are given basic implementations within the `api` module, while others may be given more _"heavy"_ implementations in the `core` module.

* [AttachManager](attachmanager.md)
* [ConfigManager](configmanager.md)
* [DecompileManager](decompilemanager.md)
* InfoImporter
* JavacCompiler
* MappingFormatManager
* MappingGenerator
* PluginManager
* [ResourceImporter](resourceimporter.md)
* ScriptEngine
* SearchService
* WorkspaceManager

## Core

The `core` module defines no new service types, but provides implementations for a number of interfaces from the `api` module.

## UI

The `ui` module defines a number of new service types dedicated to UI behavior.

* ContextMenuProviderService
* IconProviderService
* ConfigComponentManager
* ConfigIconManager
* ResourceSummaryService
* NavigationManager
* ScriptManager
* WindowFactory
* WindowManager
* PathExportingManager
* PathLoadingManager

