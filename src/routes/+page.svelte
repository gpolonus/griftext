<script lang='ts'>
  import { onDestroy } from 'svelte';
  import { marked } from 'marked';
  let text = localStorage.getItem('saved-text') || '';
  let view: 'edit' | 'view' = 'edit';
  let filename = (new Date()).toISOString().split('T')[0]

  function save() {
    var file = new Blob([text], {type: 'md'});
    var a = document.createElement("a"), url = URL.createObjectURL(file);
    a.href = url;
    a.download = filename;
    document.body.appendChild(a);
    a.click();
    setTimeout(function() {
      document.body.removeChild(a);
      window.URL.revokeObjectURL(url);
    }, 0);
  }

  function clear() {
    text = ''
    localStorage.setItem('saved-text', '')
  }

  function tempSave() {
    localStorage.setItem('saved-text', text);
  }

  const interval = setInterval(tempSave, 6000)
  onDestroy(() => {
    clearInterval(interval);
  })

</script>

<style>
  :global(body) {
    text-align: center;
    font-family: 'Anonymous Pro', monospace;
  }

  .view-switch {
    padding-bottom: 1rem;
  }

  .filename input {
    margin-bottom: 1rem;
    font-size: 1.5rem;
    border: 1px solid transparent;
    border-radius: 0.5rem;
    outline: 0;
    text-align: center;
  }

  .filename input:focus {
    border: 1px solid black;
  }

  textarea {
    width: 100%;
    max-width: 30rem;
    height: 50vh;
    border: 0;
    border-top: 1px solid black;
    padding-top: 1rem;
    resize: none;
    outline: none;
    font-family: monospace;
  }

  .rendered-output {
    width: 50%;
  }

  .footer {
    position: fixed;
    bottom: 1rem;
    left: 1rem;
    text-align: left;
    font-size: 0.75rem;
  }
</style>

<svelte:head>
  <!-- <link href="https://fonts.googleapis.com/css2?family=Anonymous+Pro:wght@400;700&display=swap" rel="stylesheet"/> -->
  <title>
    griftext: Journaling Tool
  </title>
</svelte:head>

<h1>
  griftext
</h1>

<div class="view-switch">
  <button on:click={() => view = view === 'edit' ? 'view' : 'edit'}>
    {#if view === 'edit'}View{:else}Edit{/if}
  </button>
</div>

<div class="filename">
  <input bind:value={filename} />
</div>

{#if view === 'edit'}
  <textarea bind:value={text} placeholder="Write words here..."/>
{:else}
  <div class="rendered-output">
    {@html marked(text)}
  </div>
{/if}

<div class="save">
  <button on:click={save}> Save </button>
  <button on:click={clear}> Clear </button>
</div>

<div class="footer">
  <a href="https://grifstuf.com">grifstuf</a>
  <br />
  © Griffin Polonus 2021. All Rights Reserved.
</div>