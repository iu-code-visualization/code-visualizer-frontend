<script lang="ts">

  let fileInput: HTMLInputElement;
  let responseHtml = '';
  let requestSent = false;

  function uploadFile() {
      const file = fileInput.files?.[0];

      if (!file) {
          console.error('Please select a file');
          return;
      }
      const formData = new FormData();
      formData.append('project-archive', file);

      fetch('http://127.0.0.1:8000/visualize', {
          headers: {
          'Access-Control-Allow-Origin':'*'
          },
          method: 'POST',
          signal: AbortSignal.timeout(80000),
          keepalive: true,
          body: formData,
      }).then((response) => {
          if (response.ok) {
              response.text().then((responseText) => {
                  responseHtml = responseText
              });
          } else {
              console.error('Server error:', response.status, response.statusText);
          }
      }).catch((error) => {
          console.error(error)
      });
      requestSent = true;
  }

  const downloadHtml = () => {
      if (responseHtml) {
          const blob = new Blob([responseHtml], { type: 'text/html' });
          const url = URL.createObjectURL(blob);

          const a = document.createElement('a');
          a.href = url;
          a.download = 'downloaded-file.html'; // Установите желаемое имя файла
          a.style.display = 'none';

          document.body.appendChild(a);
          a.click();

          document.body.removeChild(a);
          URL.revokeObjectURL(url);
      }
  };

</script>

<style>
    iframe {
        width: 100%;
        height: 100vh;
    }
</style>

<main>
    <h1>Загрузите zip-архив</h1>

    <input type="file" accept=".zip" bind:this={fileInput} />
    <button on:click={uploadFile}>Отправить файл</button>

    {#if responseHtml}
        <button on:click={downloadHtml}>Скачать HTML</button>
        <h2>Ответ от сервера:</h2>
        <iframe srcdoc="{responseHtml}"></iframe>
    {:else if requestSent}
        <h2>Ждем ответ от сервера</h2>
    {/if}

</main>
