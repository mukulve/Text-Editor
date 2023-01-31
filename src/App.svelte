<script>
  import Fuse from "fuse.js";
  import SvelteMarkdown from "svelte-markdown";

  let files,
    fileText,
    fileName,
    fileDir,
    searchTerm,
    searchResults = [],
    editedFiles = [],
    deviceWidth = 0,
    source,
    viewExplorer = false,
    viewPreview = false,
    width = 0;

  $: files, clearLocal();

  function clearLocal() {
    console.log("Cleaning Local Storage");
    localStorage.clear();
  }

  $: fileText, (source = fileText);
  $: viewPreview, txtwidth();
  $: deviceWidth, txtwidth();
  $: viewExplorer, txtwidth();
  $: editedFiles, txtwidth();

  async function txtwidth() {
    if (deviceWidth > 850 && viewPreview == true) {
      width = deviceWidth - 255 - 250 - 32 + "px";
    } else if (deviceWidth > 850 && viewPreview == false) {
      width = deviceWidth - 255 - 32 + "px";
    } else if (deviceWidth <= 850 && viewPreview == true) {
      width = deviceWidth - 5 - 250 - 32 + "px";
    } else {
      width = deviceWidth - 5 - 32 + "px";
    }
  }

  async function searchFiles() {
    await searchTerm;
    const options = {
      includeScore: true,
      keys: ["name"],
    };

    const fuse = new Fuse(Array.from(files), options);
    const result = fuse.search(searchTerm);
    searchResults = result;
  }

  /**
   * @param {number} index
   */
  function closeFile(index) {
    editedFiles.splice(index, 1);
    editedFiles = editedFiles;
    if (editedFiles[0]) {
      openFile(editedFiles[0].index);
    } else {
      fileText = "";
      fileDir = "";
      fileName = "";
    }
    event.stopPropagation();
  }

  /**
   * @param {number} index
   */
  async function openFile(index) {
    try {
      fileDir = await Array.from(files)[index].webkitRelativePath;
      fileName = await Array.from(files)[index].name;
      if (localStorage.getItem(fileDir)) {
        console.log(`Found Locally ${fileDir}`);
        fileText = localStorage.getItem(fileDir);
      } else {
        fileText = await Array.from(files)[index].text();
      }

      if (editedFiles.filter((e) => e.fileDir === fileDir).length <= 0) {
        editedFiles.push({ fileText, fileName, fileDir, index });
      }

      source = fileText;
      editedFiles = editedFiles;
      txtwidth();
    } catch {
      alert("An error has occured trying to open file");
    }
  }

  async function download() {
    if (fileText) {
      const { saveAs } = await import("file-saver");
      var blob = new Blob([fileText], {
        type: "text/plain;charset=utf-8",
      });
      saveAs(blob, fileDir.split("/").at(-1));
    } else {
      console.log("no file to download");
    }
  }

  async function saveLocal() {
    await fileText;
    await fileDir;
    localStorage.setItem(fileDir, fileText);
  }

  function theme() {
    window.document.body.classList.toggle("dark");
    window.document.body.classList.toggle("light");
  }
</script>

{#if files && deviceWidth <= 850 && viewExplorer}
  <div class="tabLeft">
    <h1>Explorer</h1>
    <input
      placeholder="Search File"
      bind:value={searchTerm}
      on:input={searchFiles}
    />
    {#if searchResults.length > 0}
      {#each searchResults as file, i}
        <button on:click={() => openFile(file.refIndex)}
          >{file.item.name}</button
        >
      {/each}
    {:else}
      {#each Array.from(files) as file, i}
        <button on:click={() => openFile(i)}>{file.name}</button>
      {/each}
    {/if}
  </div>
{/if}

{#if files && viewPreview}
  <div class="tabRight">
    {#if fileDir.includes(".svg") || fileDir.includes(".html") || files[0].type.includes("svg") || files[0].type.includes("html")}
      <h1 style="text-align: center;padding: 1rem;">Preview</h1>
      {@html fileText}
    {:else if fileDir.includes(".md") || files[0].name.includes(".md")}
      <h1 style="text-align: center;padding: 1rem;">Preview</h1>
      <SvelteMarkdown {source} />
    {:else}
      <h1 style="text-align: center;padding: 1rem;">No Preview Available</h1>
    {/if}
  </div>
{/if}
<nav>
  <div class="dropdown">
    <button class="dropbtn">File</button>
    <div class="dropdown-content">
      <label class="custom-file-upload">
        <h1>Open Folder</h1>
        <input type="file" webkitdirectory multiple bind:files />
      </label>
      <label class="custom-file-upload">
        <h1>Open File</h1>
        <input type="file" bind:files />
      </label>
    </div>
  </div>
  <button on:click={download}>Download Current File</button>
</nav>
{#if files}
  <main>
    {#if deviceWidth > 850}
      <div class="explorer">
        <h1>Explorer</h1>
        <input
          placeholder="Search File"
          bind:value={searchTerm}
          on:input={searchFiles}
        />
        {#if searchResults.length > 0}
          {#each searchResults as file, i}
            <button on:click={() => openFile(file.refIndex)}
              >{file.item.name}</button
            >
          {/each}
        {:else}
          {#each Array.from(files) as file, i}
            <button on:click={() => openFile(i)}>{file.name}</button>
          {/each}
        {/if}
      </div>
    {:else}
      <div class="explorer">
        {#if !viewExplorer}
          <button on:click={() => (viewExplorer = !viewExplorer)}>
            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 256 512"
              ><!--! Font Awesome Pro 6.2.1 by @fontawesome - https://fontawesome.com License - https://fontawesome.com/license (Commercial License) Copyright 2022 Fonticons, Inc. --><path
                d="M246.6 278.6c12.5-12.5 12.5-32.8 0-45.3l-128-128c-9.2-9.2-22.9-11.9-34.9-6.9s-19.8 16.6-19.8 29.6l0 256c0 12.9 7.8 24.6 19.8 29.6s25.7 2.2 34.9-6.9l128-128z"
              /></svg
            >
          </button>
        {:else}
          <button on:click={() => (viewExplorer = !viewExplorer)}>
            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 256 512"
              ><!--! Font Awesome Pro 6.2.1 by @fontawesome - https://fontawesome.com License - https://fontawesome.com/license (Commercial License) Copyright 2022 Fonticons, Inc. --><path
                d="M9.4 278.6c-12.5-12.5-12.5-32.8 0-45.3l128-128c9.2-9.2 22.9-11.9 34.9-6.9s19.8 16.6 19.8 29.6l0 256c0 12.9-7.8 24.6-19.8 29.6s-25.7 2.2-34.9-6.9l-128-128z"
              /></svg
            >
          </button>
        {/if}
      </div>
    {/if}
    <div class="textArea">
      {#if fileName && fileText}
        <div class="recentFiles">
          {#each editedFiles as file, i}
            <button
              on:click={() => openFile(file.index)}
              class:selected={file.fileDir == fileDir}
              class:unselected={file.fileDir != fileDir}
              ><h1>{file.fileName}</h1>
              <button on:click={() => closeFile(i)}
                ><svg
                  class:selected={file.fileDir == fileDir}
                  class:unselected={file.fileDir != fileDir}
                  xmlns="http://www.w3.org/2000/svg"
                  viewBox="0 0 320 512"
                  ><!--! Font Awesome Pro 6.2.1 by @fontawesome - https://fontawesome.com License - https://fontawesome.com/license (Commercial License) Copyright 2022 Fonticons, Inc. --><path
                    d="M310.6 150.6c12.5-12.5 12.5-32.8 0-45.3s-32.8-12.5-45.3 0L160 210.7 54.6 105.4c-12.5-12.5-32.8-12.5-45.3 0s-12.5 32.8 0 45.3L114.7 256 9.4 361.4c-12.5 12.5-12.5 32.8 0 45.3s32.8 12.5 45.3 0L160 301.3 265.4 406.6c12.5 12.5 32.8 12.5 45.3 0s12.5-32.8 0-45.3L205.3 256 310.6 150.6z"
                  /></svg
                ></button
              ></button
            >
          {/each}
        </div>
        <div class="dir">
          {fileDir}
        </div>
        <textarea bind:value={fileText} on:input={saveLocal} style:width />
      {:else}
        <div class="openFile">
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 640 512"
            ><!--! Font Awesome Pro 6.2.1 by @fontawesome - https://fontawesome.com License - https://fontawesome.com/license (Commercial License) Copyright 2022 Fonticons, Inc. --><path
              d="M392.8 1.2c-17-4.9-34.7 5-39.6 22l-128 448c-4.9 17 5 34.7 22 39.6s34.7-5 39.6-22l128-448c4.9-17-5-34.7-22-39.6zm80.6 120.1c-12.5 12.5-12.5 32.8 0 45.3L562.7 256l-89.4 89.4c-12.5 12.5-12.5 32.8 0 45.3s32.8 12.5 45.3 0l112-112c12.5-12.5 12.5-32.8 0-45.3l-112-112c-12.5-12.5-32.8-12.5-45.3 0zm-306.7 0c-12.5-12.5-32.8-12.5-45.3 0l-112 112c-12.5 12.5-12.5 32.8 0 45.3l112 112c12.5 12.5 32.8 12.5 45.3 0s12.5-32.8 0-45.3L77.3 256l89.4-89.4c12.5-12.5 12.5-32.8 0-45.3z"
            /></svg
          >
        </div>
      {/if}
    </div>
    <div class="preview">
      {#if !viewPreview}
        <button on:click={() => (viewPreview = !viewPreview)}>
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 256 512"
            ><!--! Font Awesome Pro 6.2.1 by @fontawesome - https://fontawesome.com License - https://fontawesome.com/license (Commercial License) Copyright 2022 Fonticons, Inc. --><path
              d="M9.4 278.6c-12.5-12.5-12.5-32.8 0-45.3l128-128c9.2-9.2 22.9-11.9 34.9-6.9s19.8 16.6 19.8 29.6l0 256c0 12.9-7.8 24.6-19.8 29.6s-25.7 2.2-34.9-6.9l-128-128z"
            /></svg
          >
        </button>
      {:else}
        <button on:click={() => (viewPreview = !viewPreview)}>
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 256 512"
            ><!--! Font Awesome Pro 6.2.1 by @fontawesome - https://fontawesome.com License - https://fontawesome.com/license (Commercial License) Copyright 2022 Fonticons, Inc. --><path
              d="M246.6 278.6c12.5-12.5 12.5-32.8 0-45.3l-128-128c-9.2-9.2-22.9-11.9-34.9-6.9s-19.8 16.6-19.8 29.6l0 256c0 12.9 7.8 24.6 19.8 29.6s25.7 2.2 34.9-6.9l128-128z"
            /></svg
          >
        </button>
      {/if}
    </div>
  </main>
{:else}
  <div class="openFileHome">
    <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 640 512"
      ><!--! Font Awesome Pro 6.2.1 by @fontawesome - https://fontawesome.com License - https://fontawesome.com/license (Commercial License) Copyright 2022 Fonticons, Inc. --><path
        d="M392.8 1.2c-17-4.9-34.7 5-39.6 22l-128 448c-4.9 17 5 34.7 22 39.6s34.7-5 39.6-22l128-448c4.9-17-5-34.7-22-39.6zm80.6 120.1c-12.5 12.5-12.5 32.8 0 45.3L562.7 256l-89.4 89.4c-12.5 12.5-12.5 32.8 0 45.3s32.8 12.5 45.3 0l112-112c12.5-12.5 12.5-32.8 0-45.3l-112-112c-12.5-12.5-32.8-12.5-45.3 0zm-306.7 0c-12.5-12.5-32.8-12.5-45.3 0l-112 112c-12.5 12.5-12.5 32.8 0 45.3l112 112c12.5 12.5 32.8 12.5 45.3 0s12.5-32.8 0-45.3L77.3 256l89.4-89.4c12.5-12.5 12.5-32.8 0-45.3z"
      /></svg
    >
    <label class="custom-file-upload">
      <h1>Open Folder</h1>
      <input type="file" webkitdirectory multiple bind:files />
    </label>
    <label class="custom-file-upload">
      <h1>Open File</h1>
      <input type="file" bind:files />
    </label>
  </div>
{/if}
<footer>
  <div class="theme">
    <button on:click={theme}>Change Theme</button>
  </div>
  <div class="links">
    <h4>Help</h4>
    {#if fileDir}
      <h4>{fileDir.split(".")[1]}</h4>
    {/if}
  </div>
</footer>
<svelte:window bind:innerWidth={deviceWidth} />
