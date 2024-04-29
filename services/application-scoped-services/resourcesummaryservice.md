---
description: >-
  The resource summary service allows visualizing information in a workspace
  on a per-resource basis based on a set of configured summarizers.
---

# ResourceSummaryService

The resource summary service is used to display a _"summary"_ of `WorkspaceResource` items in a `Workspace` when it is opened in the GUI. There are a set of default `ResourceSummarizer` items but more can be added via this service.

## Registering additional summarizers

An additional summarizer can be added via `addSmmarizer(ResourceSummarizer)`. These interfaces are invoked once for each `WorkspaceResource` in a given `Workspace` once it is opened. Content is displayed by passing data to the `SummaryConsumer consumer` parameter.

```java
summaryService.addSummarizer((workspace, resource, consumer) -> {
	// Return false to indicate that summarizing was skipped.
	// When summaries are made a horizontal line is put between sections for clarity.
	if (shouldSkip(workspace, resource))
		return false;
    
	long count = resource.classBundleStream().flatMap(Bundle::stream).count();
	consumer.appendSummary(new Label("There are " + count + " classes"));
	return true;
});
```

The `SummaryConsumer` has two methods:

- `appendSummary(Node)` - For content spanning the full width of the report.
- `appendSummary(Node, Node)` For content to display in two columns the first on the left and the second on the right.
  - All content is aligned to a grid, so multiple calls to this method do not have to worry about sizing discrepancies between calls.
