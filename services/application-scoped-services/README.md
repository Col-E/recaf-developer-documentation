---
description: Services bound to the duration of Recaf being open.
---

# Application scoped services

These services are available for injection at any point.

## API

These are the services defined in the `core` module.

* [AttachManager](attachmanager.md)
* [ConfigManager](configmanager.md)
* [DecompileManager](decompilemanager.md)
* InfoImporter
* JavacCompiler
* MappingFormatManager
* MappingGenerator
* MappingListeners
* NameGeneratorProviders
* PhantomGenerator
* PluginManager
* [ResourceImporter](resourceimporter.md)
* ScriptEngine
* ScriptManager
* SearchService
* WorkspaceManager

## UI

The `ui` module defines a number of new service types dedicated to UI behavior.

* Actions
* CellConfigurationService _(Wraps these services)_
  * ContextMenuProviderService
  * IconProviderService
  * TextProviderService
* ConfigComponentManager
* ConfigIconManager
* FileTypeAssociationService
* ResourceSummaryService
* NavigationManager
* WindowFactory
* WindowManager
* PathExportingManager
* PathLoadingManager

