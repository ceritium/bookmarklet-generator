<script>
  import { browser } from '$app/environment';
  import { onMount } from 'svelte';
	import LZString from 'lz-string';
  import { minify } from "terser";

  let ready = false;
  let code = "";
  let title = ""
  let selfUrl = ''
  let encodedCode = ''
  let compress = false;
  let uriEncoded = true;
  let jsError = null;

  let timeout

  const generateMarkletCode = async (code, compress, uriEncoded) => {
    let processedCode = code.trim()
    if (compress) {
      try {
        processedCode = (await minify({"inline.js": processedCode}, {compress: true})).code
        jsError = null
      } catch(e) {
        jsError = e.message
      }
    } else {
      jsError = null
    }

    processedCode = `(function(){${processedCode}})();`

    if (uriEncoded) {
      processedCode = encodeURIComponent(processedCode)
    }

    encodedCode = `javascript:${processedCode};`
  }

  $: {
    // debounce
    if (timeout) clearTimeout(timeout)
    timeout = setTimeout(() => {
      generateMarkletCode(code, compress, uriEncoded)
    }, 300)
  }

  $: linkCode = `<a href="${encodedCode}"> ${title} </a>`;
  $: {
    if(browser && ready) {
      const data = {code: code, title: title, compress: compress, uriEncoded: uriEncoded}
      const hash = LZString.compressToEncodedURIComponent(JSON.stringify(data))
      window.location.hash = hash;
      selfUrl = window.location.toString()
    }
  }

	onMount(() => {
		const hash = window.location.hash.slice(1);

		try {
			const data = JSON.parse(LZString.decompressFromEncodedURIComponent(hash));
      code = data.code;
      title = data.title;
      compress = data.compress;
      uriEncoded = data.uriEncoded;

		} catch (e) {
      code = 'alert("Hello world!");'
      title = "Drag me to your bookmarks bar";
    }

    ready = true
	});
</script>

<svelte:head>
  <title>Bookmarklet generator | {title} </title>
</svelte:head>

<h1> Bookmarklet generator </h1>
<a class='github-link' href="https://github.com/ceritium/bookmarklet-generator"> Github </a>
<label>
  Title
  <input type="text" bind:value={title} />
</label>
<label for="code">
  Code
  <!--
   Disabled grammarly as described here: https://stackoverflow.com/a/46777787
   Disabled the browser spellcheck
  -->
  <textarea data-gramm="false" data-gramm_editor="false" data-enable-grammarly="false" spellcheck="false" bind:value={code} rows={10} />
  {#if (jsError) }
    <small style="color: red;">{jsError}</small>
  {/if}
</label>

<label>
  Compress js
  <input type="checkbox" bind:checked={compress} />
</label>

<label>
  URI encoded
  <input type="checkbox" bind:checked={uriEncoded} />
  <small>
    The bookmark may not work if the code is not URI encoded.
  </small>
</label>

<a class="link" href={encodedCode}>{title}</a>
<br/>
<small>
  Click to test your bookmarklet or drag it to your bookmarks bar
</small>

<h2> Code output </h2>
<h3> Bookmarklet code </h3>
<textarea readonly value={encodedCode} rows={5}/>
<small>
  { encodedCode.length } characters. 
  The bookmarklet probably will not work if it's longer than 2000 characters.
</small>

<h3> Link to bookmarklet code </h3>
<textarea readonly value={linkCode} rows={5}/>

<style>
  textarea {
    resize: vertical;
  }
  .github-link {
    position: absolute;
    top: 0;
    right: 0;
    margin: 10px;
    color: #333;
    text-decoration: none;
  }

  h1 {
    text-align: center;
    margin: 40px 0;
  }

  h2 {
    margin-top: 30px;
  }

  h3 {
    margin-top: 20px;
  }
  .link {
    display: inline-block;
    padding: 0.5em 1em;
    background-color: #eee;
    border: 1px solid #ccc;
    border-radius: 3px;
    text-decoration: none;
    color: #333;
    cursor: pointer;
  }
  .link:hover {
    background-color: #ddd;
  }
</style>
