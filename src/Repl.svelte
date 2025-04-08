<script lang="ts">
  import { saveAs } from 'file-saver';

  export let query: string;

  let sortOrder: 'asc' | 'desc' = 'asc';

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

  // Moved the setting of `sortedResults` back into `executeQuery` as it was originally correct.
  function executeQuery() {
    res = runQuery()
    .then((result) => {
      sortedResults = sortResult(result?.map(([item]) => item));
    });
  }

  // Restored the `sortResult` function to handle sorting logic.
  function sortResult(result: any[]) {
    return result?.sort((a, b) => {
      const comparison = (a.meta?.sortKey || 0) - (b.meta?.sortKey || 0);
      return sortOrder === 'asc' ? comparison : -comparison;
    });
  }

  // Replaced JSON.parse with a proper unescaping method for JSON-escaped strings.
  function unescapeJsonString(jsonEscapedString: string): string {
    return jsonEscapedString.replace(/\\n/g, '\n').replace(/\\"/g, '"').replace(/\\t/g, '\t').replace(/\\r/g, '\r');
  }

  function downloadMarkdown() {
    if (!sortedResults.length) return;

    const markdownContent = sortedResults
      .map((entry) => {
        const title = entry.meta?.title ? `# ${entry.meta.title}\n\n` : '';
        const content = unescapeJsonString(entry.content); // Properly unescape JSON-escaped markdown
        return `${title}${content}`;
      })
      .join('\n\n---\n\n');

    const blob = new Blob([markdownContent], { type: 'text/markdown;charset=utf-8' });
    saveAs(blob, 'results.md');
  }
</script>

<div class="root">
  <div class="query-input">
    <textarea bind:value={query} placeholder="Enter your query here"></textarea>
    <div class="controls">
      <select bind:value={sortOrder}>
        <option value="asc">Ascending</option>
        <option value="desc">Descending</option>
      </select>
      <button on:click={executeQuery}>Run Query</button>
      <button on:click={downloadMarkdown}>Download as Markdown</button>
    </div>
  </div>
  <div class="query-result">
    {#await res}
      <pre>...running query</pre>
    {:then result}
      <pre>{JSON.stringify(sortedResults, null, 2)}</pre>
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
    align-items: center;
    gap: 0.5em;
  }

  textarea {
    width: 100%;
    height: 200px;
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

  select {
    padding: 0.5em;
    font-size: 14px;
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
</style>
