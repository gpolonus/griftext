<script>
  import { onDestroy } from 'svelte';
  import { marked } from 'marked';
  import count from 'word-count';

  let dialogEl;

  let file = $state();
  let text = $state(localStorage.getItem('saved-text') || '')
  let view = $state('edit')
  let filename = $state(localStorage.getItem('saved-title') || '')

  $effect(() => {
    if (filename) {
      filename = filename.split(' ').map(word => word && `${word[0].toUpperCase()}${word.substring(1)}`).join(' ')
    }
  })

  function getDate() {
    return (new Date()).toISOString().split('T')[0]
  }

  function save() {
    var file = new Blob([text], { type: 'md' });
    var a = document.createElement("a"), url = URL.createObjectURL(file);
    a.href = url;
    a.download = localStorage.getItem('saved-edit-date') + '_' + filename;
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

  function openFileModal() {
    dialogEl.showModal()
  }

  function openFile() {
    const openedFile = file[0]
    const reader = new FileReader();
    reader.readAsText(openedFile, 'UTF-8');
    reader.onload = (e) => {
      text = e.target?.result || ''
      const currentFileNameMatch = /\d{4}-\d{2}-\d{2}_(.*)\.txt/i.exec(openedFile.name)
      console.log(currentFileNameMatch, openedFile.name)
      if (currentFileNameMatch && currentFileNameMatch[1]) {
        filename = currentFileNameMatch[1]
      }

      clearModal()
    }
  }

  function clearModal() {
    dialogEl.close()
  }

  function tempSave() {
    if (localStorage.getItem('saved-text') !== text) {
      localStorage.setItem('saved-edit-date', getDate());
    }
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

  dialog {
    padding: 2rem;
    max-height: calc(100vh - 210px);
    overflow-y: auto;
  }

  dialog::backdrop {
    background-color: rgba(aquamarine, 0.5);
  }

  dialog h1 {
    text-transform: capitalize;
  }

  dialog div {
    margin-bottom: 1rem;
  }

  dialog input {
    padding: 5px;
    border: 1px solid black;
    cursor: pointer;
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
  <button onclick={() => view = view === 'edit' ? 'view' : 'edit'}>
    {#if view === 'edit'}View{:else}Edit{/if}
  </button>
</div>

<div class="filename">
  <input bind:value={filename} placeholder="Untitled" />
</div>

<div class="contents">
{#if view === 'edit'}
  <textarea bind:value={text} placeholder="Write words here..."></textarea>
{:else}
  <div class="rendered-output">
    {@html marked(text)}
  </div>
{/if}
</div>

<div class="save">
  <button onclick={save}> Save </button>
  <button onclick={clear}> Clear </button>
  <button onclick={countWords}> Count </button>
  <button onclick={openFileModal}> Open </button>
</div>


<dialog bind:this={dialogEl}>
  <h1>Open File</h1>
  <div>
    <label for="textFile">Choose a file to open:</label>
  </div>
  <div>
    <input bind:files={file} type="file" id="textFile" name="textFile" accept="text/plain" />
  </div>
  <div>
    <button onclick={openFile} disabled='{file ? file.length === 0 : true}'>Open</button>
    <button onclick={clearModal}>Close</button>
  </div>
</dialog>

<div class="footer">
  <a href="https://grifstuf.com">grifstuf</a>
  <br />
  © Griffin Polonus 2021. All Rights Reserved.
</div>