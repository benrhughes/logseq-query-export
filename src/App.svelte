<script lang="ts">
  import Repl from "./Repl.svelte";

  let inner: Element;
  let show = true;
  let query = `; your query should return data in the following format:
; [ [ { content: "markdown content", meta: { title: "title", sortKey: 123 } } ] ]
; sortKey is optional, but must be provided if you want to sort the results
; The example query below finds all blocks that reference the page "logseq"

[:find (pull ?b [
  :block/content
  {(:block/page :as meta) [(:block/original-name :as title) (:block/journal-day :as sortKey)]}
 ])
 :where
 [?b :block/refs ?p]
 [?p :block/name "logseq"]
]`;

  logseq.on("ui:visible:changed", async ({ visible }) => {
    show = visible;
  });

</script>

{#if show}
  <main>
    <button class="close-button" on:click={() => window.logseq.hideMainUI()}>Close</button>
    <div bind:this={inner} class="inner"><Repl bind:query={query} /></div>
  </main>
{/if}

<style>
  main {
    top: 0;
    bottom: 0;
    left: 0;
    right: 0;
    position: fixed;
    padding: 5vh 5vw;
    backdrop-filter: saturate(0.8) blur(10px);
  }

  .inner {
    border: 3px solid #000;
    background: #eee;
    height: 100%;
    border-radius: 8px;
    overflow: hidden;
    box-shadow: 0 0 20px #888;
  }

  .close-button {
    position: absolute;
    top: 10px;
    right: 10px;
    background: #ff5f5f;
    color: white;
    border: none;
    border-radius: 4px;
    padding: 0.5em 1em;
    cursor: pointer;
    font-size: 14px;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
  }

  .close-button:hover {
    background: #ff3b3b;
  }
</style>
