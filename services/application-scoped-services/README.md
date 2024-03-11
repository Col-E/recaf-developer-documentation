---
description: Services bound to the duration of Recaf being open.
---

# Application scoped services

These services are available for injection at any point.

## API

These are the services defined in the `core` module.

* [AttachManager](attachmanager.md)
* CommentManager
* [ConfigManager](configmanager.md)
* [DecompileManager](decompilemanager.md)
* GsonProvider
* [InfoImporter](infoimporter.md)
* [JavacCompiler](javaccompiler.md)
* MappingFormatManager
* MappingGenerator
* MappingListeners
* NameGeneratorProviders
* PhantomGenerator
    * JPhantomGenerator
* PluginManager
* [ResourceImporter](resourceimporter.md)
* ScriptEngine
    * JavacScriptEngine
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
* NavigationManager
* PathExportingManager
* PathLoadingManager
* ResourceSummaryService
* WindowFactory
* WindowManager

