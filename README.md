<!doctype html>
<html lang="pt-BR">
<head>
  <meta charset="utf-8" />
  <title>Envio de Arquivos</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <style>
    :root {
      --bg: #ffffff;
      --card: #000000;
      --text: rgb(255, 255, 255);
      --accent: #000000;
      --muted: #9ca3af;
    }

    body {
      background: var(--bg);
      color: var(--text);
      font-family: system-ui, -apple-system, Segoe UI, Roboto, sans-serif;
      margin: 0;
      padding: 2rem;
    }

    .container { max-width: 760px; margin: 0 auto; }
    .card { background: var(--card); border: 1px solid #1f2937; border-radius: 12px; padding: 1.25rem; }
    h1 { margin: 0 0 1rem; font-size: 1.5rem; }

    .row {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 0.75rem;
      margin-bottom: 0.75rem;
    }

    label {
      font-size: 0.9rem;
      color: var(--muted);
      display: block;
      margin-bottom: 0.25rem;
    }

    input[type="text"], input[type="number"] {
      width: 100%;
      padding: 0.6rem 0.7rem;
      border-radius: 8px;
      border: 1px solid #374151;
      background: #0b1220;
      color: var(--text);
    }

    input[type="file"] { width: 100%; }

    button {
      background: var(--accent);
      color: #0b1220;
      border: none;
      border-radius: 10px;
      padding: 0.7rem 1rem;
      font-weight: 600;
      cursor: pointer;
    }

    button:disabled { opacity: 0.6; cursor: not-allowed; }

    .progress-wrap { margin-top: 1rem; }
    .progress-bar { height: 12px; background: #1f2937; border-radius: 999px; overflow: hidden; }
    .progress { height: 100%; width: 0%; background: linear-gradient(90deg, #22d3ee, #38bdf8); transition: width 0.15s ease; }

    .status { margin-top: 0.5rem; font-size: 0.9rem; color: var(--muted); }
    .log { margin-top: 1rem; font-size: 0.9rem; white-space: pre-wrap; background: #0b1220; padding: 0.75rem; border-radius: 8px; border: 1px solid #1f2937; }
  </style>
</head>
<body>
  <div class="container">
    <div class="card">
      <h1>Envio de arquivos</h1>

      <div class="row">
        <div>
          <label for="server">Servidor (ex.: http://123.456.0.78:5000)</label>
          <input id="server" type="text" placeholder="http://localhost:5000" value="http://localhost:5000" />
        </div>
        <div>
          <label for="endpoint">Endpoint</label>
          <input id="endpoint" type="text" placeholder="/upload" value="/upload" />
        </div>
      </div>

      <div class="row">
        <div>
          <label for="file">Arquivo</label>
          <input id="file" type="file" />
        </div>
        <div>
          <label for="token">Token (opcional)</label>
          <input id="token" type="text" placeholder="Chave de acesso" />
        </div>
      </div>

      <button id="sendBtn">Enviar arquivo</button>

      <div class="progress-wrap">
        <div class="progress-bar">
          <div class="progress" id="progress"></div>
        </div>
        <div class="status" id="status">Aguardando envio...</div>
      </div>

      <div class="log" id="log"></div>
    </div>
  </div>

  <script>
    const sendBtn = document.getElementById('sendBtn');
    const fileInput = document.getElementById('file');
    const serverInput = document.getElementById('server');
    const endpointInput = document.getElementById('endpoint');
    const tokenInput = document.getElementById('token');
    const progressEl = document.getElementById('progress');
    const statusEl = document.getElementById('status');
    const logEl = document.getElementById('log');

    function setProgress(pct) {
      progressEl.style.width = `${pct}%`;
      statusEl.textContent = `Progresso: ${pct.toFixed(1)}%`;
    }

    function log(msg) {
      const ts = new Date().toLocaleTimeString();
      logEl.textContent += `[${ts}] ${msg}\n`;
    }

    async function upload() {
      const file = fileInput.files[0];
      const base = serverInput.value.trim().replace(/\/+$/, '');
      const endpoint = endpointInput.value.trim().startsWith('/')
        ? endpointInput.value.trim()
        : '/' + endpointInput.value.trim();
      const url = `${base}${endpoint}`;

      if (!file) {
        alert('Selecione um arquivo.');
        return;
      }

      sendBtn.disabled = true;
      setProgress(0);
      statusEl.textContent = 'Iniciando...';
      log('Preparando envio.');

      const form = new FormData();
      form.append('file', file, file.name);

      const xhr = new XMLHttpRequest();
      xhr.open('POST', url, true);

      if (tokenInput.value.trim()) {
        xhr.setRequestHeader('X-Auth-Token', tokenInput.value.trim());
      }

      xhr.upload.onprogress = (e) => {
        if (e.lengthComputable) {
          const pct = (e.loaded / e.total) * 100;
          setProgress(pct);
        }
      };

      xhr.onload = () => {
        sendBtn.disabled = false;
        if (xhr.status >= 200 && xhr.status < 300) {
          setProgress(100);
          statusEl.textContent = 'Concluído.';
          log(`Sucesso: ${xhr.responseText || 'OK'}`);
        } else {
          statusEl.textContent = `Falha (${xhr.status})`;
          log(`Erro: ${xhr.status} - ${xhr.responseText}`);
        }
      };

      xhr.onerror = () => {
        sendBtn.disabled = false;
        statusEl.textContent = 'Erro de rede.';
        log('Erro de rede ao enviar.');
      };

      xhr.send(form);
    }

    sendBtn.addEventListener('click', upload);
  </script>
</body>
</html>
