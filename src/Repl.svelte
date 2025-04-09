<script lang="ts">
  import { saveAs } from 'file-saver';

  export let query: string;

  type RefAction = 'do-nothing' | 'replace-with-text' | 'delete';

  let sortOrder: 'asc' | 'desc' = 'asc';
  let pageRefsAction: RefAction = 'do-nothing';
  let hashRefsAction: RefAction = 'do-nothing';

  async function runQuery() {
    const errorKey = "#lspmsg#error#";
    return await logseq.DB.datascriptQuery(query).then((res) => {
      if (errorKey in res) {
        throw res[errorKey];
      }
      return res;
    });
  }

  let res: ReturnType<typeof runQuery>;
  let sortedResults: any[] = [];
  let queryResult: string = '';

  function executeQuery() {
    res = runQuery()
    .then((result) => {
      sortedResults = sortResult(result?.map(([item]) => item));
      queryResult = JSON.stringify(sortedResults, null, 2); // Set queryResult to display JSON by default
    });
  }

  function sortResult(result: any[]) {
    return result?.sort((a, b) => {
      const comparison = (a.meta?.sortKey || 0) - (b.meta?.sortKey || 0);
      return sortOrder === 'asc' ? comparison : -comparison;
    });
  }

  function unescapeJsonString(jsonEscapedString: string): string {
    try {
      // Escape control characters and ensure the string is JSON-safe
      const sanitizedString = jsonEscapedString
        .replace(/\\/g, '\\\\') // Escape backslashes
        .replace(/\n/g, '\\n') // Escape newlines
        .replace(/\r/g, '\\r') // Escape carriage returns
        .replace(/\t/g, '\\t') // Escape tabs
        .replace(/"/g, '\\"'); // Escape double quotes

      return JSON.parse(`"${sanitizedString}"`);
    } catch (error) {
      console.error("Failed to unescape JSON string:", error, "Input:", jsonEscapedString);
      // Return the original string if parsing fails
      return jsonEscapedString;
    }
  }

  function generateMarkdown() {
    if (!sortedResults.length) return;

    const markdownContent = sortedResults
      .map((entry) => {
        const title = entry.meta?.title ? `# ${entry.meta.title}\n\n` : '---\n\n';
        let content = unescapeJsonString(entry.content); // Properly unescape JSON-escaped markdown

        if (pageRefsAction === 'replace-with-text') {
          content = content.replace(/\[\[(.*?)\]\]/g, '$1');
        } else if (pageRefsAction === 'delete') {
          content = content.replace(/\[\[.*?\]\]/g, '');
        }

        if (hashRefsAction === 'replace-with-text') {
          content = content.replace(/#(\w+)/g, '$1');
        } else if (hashRefsAction === 'delete') {
          content = content.replace(/#\w+/g, '');
        }

        return `${title}${content}`;
      })
      .join('\n\n');

    queryResult = markdownContent; // Update the query-result area with markdown
  }

  function downloadContent() {
    if (!queryResult) return;

    const blob = new Blob([queryResult], { type: 'text/plain;charset=utf-8' });
    saveAs(blob, 'results.txt');
  }
</script>

<div class="root">
  <div class="query-input">
    <textarea bind:value={query} placeholder="Enter your query here"></textarea>
    <div class="controls">
      <div class="control-group">
        <label for="sort-order">Sort Order:</label>
        <select id="sort-order" bind:value={sortOrder}>
          <option value="asc">Asc</option>
          <option value="desc">Desc</option>
        </select>
      </div>
      <div class="control-group">
        <button on:click={executeQuery}>Run Query</button>
        <div class="vertical-separator"></div>
      </div>
      <div class="control-group">
        <label for="page-refs-action">[[]] Page Refs:</label>
        <select id="page-refs-action" bind:value={pageRefsAction}>
          <option value="do-nothing">Keep</option>
          <option value="replace-with-text">Replace with text</option>
          <option value="delete">Delete</option>
        </select>
      </div>
      <div class="control-group">
        <label for="hash-refs-action"># Page Refs:</label>
        <select id="hash-refs-action" bind:value={hashRefsAction}>
          <option value="do-nothing">Keep</option>
          <option value="replace-with-text">Replace with text</option>
          <option value="delete">Delete</option>
        </select>
      </div>
      <div class="control-group">
        <button on:click={generateMarkdown}>Generate Markdown</button>
        <div class="vertical-separator"></div>
        <button on:click={downloadContent}>Download Content</button>
      </div>
    </div>
  </div>
  <div class="query-result">
    {#await res}
      <pre>...running query</pre>
    {:then result}
      <pre>{queryResult}</pre>
    {:catch error}
      <pre class="error">{error.message}</pre>
    {/await}
  </div>
</div>

<style>
  .root {
    display: flex;
    flex-direction: column;
    gap: 1em;
    width: 100%;
    height: 100%;
  }

  .query-input {
    display: flex;
    flex-direction: column;
    gap: 0.5em;
  }

  .controls {
    display: flex;
    flex-wrap: wrap;
    gap: 1em;
    margin-left: 0.5em;
  }

  .control-group {
    display: flex;
    flex-direction: row;
    align-items: center;
    gap: 0.5em;
  }

  label {
    font-size: 14px;
    font-weight: bold;
  }

  select, button {
    padding: 0.5em;
    font-size: 14px;
    flex-grow: 1; /* Ensures alignment with selects */
  }

  textarea {
    width: 100%;
    height: 250px;
    font-family: monospace;
    font-size: 14px;
    padding: 0.5em;
  }

  button {
    align-self: flex-start;
    padding: 0.5em 1em;
    font-size: 14px;
    cursor: pointer;
  }

  .query-result {
    white-space: pre-wrap;
    font-family: monospace;
    font-size: 14px;
    padding: 1em;
    background: #f9f9f9;
    border: 1px solid #ddd;
    border-radius: 4px;
    overflow-y: auto;
  }

  .error {
    color: red;
  }

  .vertical-separator {
    width: 1px;
    background-color: #ccc;
    height: 100%;
    margin: 0 0.5em;
  }
</style>
