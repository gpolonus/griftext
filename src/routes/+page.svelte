<script lang='ts'>
  import { onDestroy } from 'svelte';
  import { marked } from 'marked';
  import count from 'word-count';

  let text = localStorage.getItem('saved-text') || '';
  let view: 'edit' | 'view' = 'edit';
  let filename = localStorage.getItem('saved-title') || ''

  function save() {
    var file = new Blob([text], { type: 'md' });
    var a = document.createElement("a"), url = URL.createObjectURL(file);
    a.href = url;
    a.download = (new Date()).toISOString().split('T')[0] + '_' + filename;
    document.body.appendChild(a);
    a.click();

    setTimeout(function() {
      document.body.removeChild(a);
      window.URL.revokeObjectURL(url);
    }, 0);

    clear();
  }

  function clear() {
    text = ''
    localStorage.setItem('saved-text', '')

    filename = ''
    localStorage.setItem('saved-title', filename);
  }

  function countWords() {
    alert(`You've written ${count(text)} words.`);
  }

  function tempSave() {
    localStorage.setItem('saved-text', text);
    localStorage.setItem('saved-title', filename);
  }

  const interval = setInterval(tempSave, 3000)
  onDestroy(() => {
    clearInterval(interval);
  })

</script>

<style>
  :global(body) {
    text-align: center;
    font-family: 'Courier Prime', monospace;
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

  .contents {
    width: 100%;
    margin: 0 auto;
    max-width: 30rem;
    height: 50vh;
    padding-top: 1rem;
    padding-bottom: 1.5rem;
  }

  textarea {
    width: 100%;
    height: 100%;
    border: 0;
    border-top: 1px solid black;
    resize: none;
    outline: none;
    font-family: 'Courier Prime', monospace;
    font-size: 1rem;
  }

  .rendered-output {
    text-align: left;
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
  <link rel="preconnect" href="https://fonts.googleapis.com" />
  <link rel="preconnect" href="https://fonts.gstatic.com" />
  <link href="https://fonts.googleapis.com/css2?family=Courier+Prime:ital,wght@0,400;0,700;1,400;1,700&display=swap" rel="stylesheet" />
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
  <input bind:value={filename} placeholder="Untitled" />
</div>

<div class="contents">
{#if view === 'edit'}
  <textarea bind:value={text} placeholder="Write words here..."/>
{:else}
  <div class="rendered-output">
    {@html marked(text)}
  </div>
{/if}
</div>

<div class="save">
  <button on:click={save}> Save </button>
  <button on:click={clear}> Clear </button>
  <button on:click={countWords}> Count </button>
</div>

<div class="footer">
  <a href="https://grifstuf.com">grifstuf</a>
  <br />
  © Griffin Polonus 2021. All Rights Reserved.
</div>