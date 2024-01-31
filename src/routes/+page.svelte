<script lang="ts">

  let fileInput: HTMLInputElement;
  let responseHtml = '';


  const uploadFile = async () => {
    const file = fileInput.files?.[0];

    if (!file) {
      console.error('Please select a file');
      return;
    }
    const formData = new FormData();
    formData.append('project-archive', file);
    try {
      const response = await Promise.race([
        fetch('http://127.0.0.1:8000/visualize', {
          mode: 'no-cors',
          //headers: {
          // 'Access-Control-Allow-Origin':'*'
          //},
          method: 'POST',
          signal: AbortSignal.timeout(80000),
          keepalive: true,
          body: formData,
        }),
        new Promise<Response>((_, reject) =>
            setTimeout(() => reject(new Error('Request timed out')), 30000)
        ),
      ]);

      if (response.ok) {
        responseHtml = await response.text();
      } else {
        console.error('Server error:', response.status, response.statusText);
      }
    } catch (error : any) {
      console.error('Error:', error.message);
    }
  };
</script>

<main>
    <h1>Загрузите zip-архив</h1>

    <input type="file" accept=".zip" bind:this={fileInput} />
    <button on:click={uploadFile}>Отправить файл</button>

    {#if responseHtml}
        <div>
            <h2>Ответ от сервера:</h2>
            {@html responseHtml}
        </div>
    {/if}
</main>
