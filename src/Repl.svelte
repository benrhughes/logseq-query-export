<script lang="ts">
  export let query: string;

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
  function executeQuery() {
    res = runQuery();
  }

  function sortResult(result: any[]) {
    result?.sort((a, b) => {
      return (a[0].sortKey || 0) - (b[0].sortKey || 0);
    });
    return result;
  }
</script>

<div class="root">
  <div class="query-input">
    <textarea bind:value={query} placeholder="Enter your query here"></textarea>
    <button on:click={executeQuery}>Run Query</button>
  </div>
  <div class="query-result">
    {#await res}
      <pre>...running query</pre>
    {:then result}
      <pre>{JSON.stringify(sortResult(result), null, 2)}</pre>
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

  textarea {
    width: 100%;
    height: 150px;
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
    overflow-y: auto; /* Enable vertical scrolling */
  }

  .error {
    color: red;
  }
</style>
