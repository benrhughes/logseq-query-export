# Logseq Query Exporter Plugin

A plugin for Logseq that allows you to run a query and export the results to markdown.

![](./screenshot.png)

## Querying
You can run any query you'd like and see the resulting json. If you'd like to take advantage of sorting or exporting to markdown, the json from your query will need to have the following structure:

```json
[ 
    [ 
        { 
            content: "markdown content", 
            meta: 
            { 
                title: "title", 
                sortKey: 123 
            } 
        } 
    ] 
]
```

When you first open the plugin, the sample query will provide results in the expected format. For reference, that query is:

```cljs
[:find (pull ?b [
  :block/content
  {(:block/page :as meta) [(:block/original-name :as title) (:block/journal-day :as sortKey)]}
 ])
 :where
 [?b :block/refs ?p]
 [?p :block/name "logseq"]
]
```

## Generating Markdown 

The "Generate Markdown" button converts the JSON query results into markdown format and displays it in the query-result area. This allows you to preview the markdown before downloading it.

### Page and Hash References Processing

When exporting query results to markdown, you can customize how page references (`[[]]`) and hash references (`#`) are handled. The available options are:

- **Keep**: Leave the references as they are.
- **Replace with text**: Convert references to plain text.
- **Delete**: Remove the references entirely.

### Grouping by Title

If you check the "Group by Title" checkbox, entries with the same title will be grouped under that title, separated by `---`. Titles will not be repeated.

### Download Content 

The "Download Content" button downloads the current content of the query-result area. This could be either the JSON query results or the generated markdown, depending on what is currently displayed.


