<script>
  import { fabric } from 'fabric'
  import ImageLoader from './ImageLoader.svelte'
  import AddHeadButton from './AddHeadButton.svelte'
  import Canvas from './Canvas.svelte'
  import Image from './Image.svelte'
  import Head from './Head.svelte'
  import Footer from './Footer.svelte'
  import images from './images'

  let canvas, image, scale, heads = []
  let result

  function handleKeydown (e) {
    if (e.which === 8 || e.which === 46) { // backspace or delete key
      deleteSelected()
    }
  }

  function deleteSelected () {
    const activeObjects = canvas.getActiveObjects()
    activeObjects.forEach(activeObject => {
      if (activeObject.head) {
        heads = heads.filter(head => head.id !== activeObject.head.id)
      }
    })
  }

  function replaceImage (e) {
    image = e.detail.image
    heads = []
  }

  function addHead (e) {
    heads = [...heads, e.detail.head]
  }

  function output () {
    const width = image ? image.width : canvas.width
    const height = image ? image.height : canvas.height

    result = canvas.toDataURL({
      format: 'png',
      multiplier: 1 / scale,
      width: width * scale,
      height: height * scale
    })
  }

  function clearResult () {
    result = null
  }
</script>

<svelte:head>
  <link rel='stylesheet' href='https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/css/bootstrap.min.css'
      integrity='sha384-Vkoo8x4CGsO3+Hhxv8T/Q5PaXtkKtu6ug5TOeNV6gBiFeWPGFN9MuhOf23Q9Ifjh'
      crossorigin='anonymous'>
</svelte:head>

<svelte:body on:keydown={handleKeydown} />

<style>
  .result {
    position: fixed;
    left: 0;
    right: 0;
    top: 0;
    bottom: 0;
    background-color: white;
    overflow-y: auto;
    z-index: 10;
  }

  .images {
    width: 250px;
  }
</style>

<div class='container'>
  <h1>接頭霸王 <small class='text-muted'>v0.9</small></h1>
  <div class='form-row'>
    <div class='col'>
      <div>第一步, 選擇原圖:</div>
      <ImageLoader on:load={replaceImage} />
    </div>
    <div class='col'>
      <div>第三步, 把頭拖到喜歡的地方,</div>
      <div class='text-muted'>可拖拉控制點翻轉圖片!</div>
      <div class='text-muted'>按 delete 刪除多餘的頭.</div>
    </div>
    <div class='col'>
      <div>最後, 下載並分享:</div>
      <button class='btn btn-outline-secondary' on:click={output}>
        下載圖片
      </button>
    </div>
  </div>

  <div class='d-flex'>
    <div class='images'>
      <div>第二步, 加頭:</div>
      <div class='d-flex flex-wrap'>
        {#each images as image}
          <AddHeadButton source={image} on:add={addHead} />
        {/each}
      </div>
    </div>

    <div class='flex-fill'>
      <Canvas bind:canvas />
    </div>
  </div>

  <Footer />

  {#if canvas}
    {#if image}
      {#each [image] as image (image.src)}
        <Image {canvas} {image} bind:scale />
      {/each}
    {/if}

    {#each heads as head (head.id)}
      <Head {canvas} {head} />
    {/each}
  {/if}
</div>

{#if result}
  <div class='result'>
    <div class='container'>
      <div class='d-flex'>
        <div class='flex-fill'>完成了! 請對圖片按右鍵另存:</div>
        <button class='btn btn-primary' on:click={clearResult}>繼續編輯</button>
      </div>
      <img src={result} style='max-width: 100%;' />
    </div>
  </div>
{/if}
