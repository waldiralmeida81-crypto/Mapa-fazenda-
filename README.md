# Mapa-fazenda-
Citro
html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />
  <title>AgroPonto GPS - Gestão de Serviços na Fazenda</title>
  
  <!-- Biblioteca de Mapas Leaflet -->
  <link rel="stylesheet" href="https://unpkg.com" />
  <script src="https://unpkg.com"></script>

  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; font-family: 'Segoe UI', Roboto, sans-serif; }
    body { display: flex; flex-direction: column; height: 100vh; background: #f4f6f4; overflow: hidden; }

    /* Cabeçalho */
    header {
      background: #1b5e20;
      color: white;
      padding: 12px 16px;
      display: flex;
      justify-content: space-between;
      align-items: center;
      box-shadow: 0 2px 6px rgba(0,0,0,0.2);
      z-index: 1000;
    }
    header h1 { font-size: 1.15rem; display: flex; align-items: center; gap: 8px; }
    .header-actions button {
      background: #2e7d32;
      color: white;
      border: 1px solid #81c784;
      padding: 6px 10px;
      border-radius: 6px;
      font-size: 0.85rem;
      cursor: pointer;
    }

    /* Mapa */
    #map { flex: 1; width: 100%; z-index: 1; }

    /* Botões Flutuantes */
    .fab-container {
      position: fixed;
      bottom: 24px;
      right: 16px;
      display: flex;
      flex-direction: column;
      gap: 12px;
      z-index: 1000;
    }
    .btn-gps {
      background: #ffffff;
      color: #1b5e20;
      border: 2px solid #1b5e20;
      padding: 12px 16px;
      border-radius: 50px;
      font-weight: bold;
      box-shadow: 0 4px 10px rgba(0,0,0,0.25);
      cursor: pointer;
    }
    .btn-add {
      background: #2e7d32;
      color: white;
      border: none;
      padding: 14px 20px;
      border-radius: 50px;
      font-size: 1rem;
      font-weight: bold;
      box-shadow: 0 4px 12px rgba(0,0,0,0.3);
      cursor: pointer;
    }
    .btn-add:active, .btn-gps:active { transform: scale(0.96); }

    /* Modal de Cadastro / Edição */
    .modal {
      display: none;
      position: fixed;
      inset: 0;
      background: rgba(0,0,0,0.6);
      z-index: 2000;
      justify-content: center;
      align-items: center;
      padding: 16px;
    }
    .modal.active { display: flex; }
    .modal-content {
      background: white;
      width: 100%;
      max-width: 420px;
      border-radius: 12px;
      padding: 20px;
      box-shadow: 0 8px 24px rgba(0,0,0,0.3);
      max-height: 90vh;
      overflow-y: auto;
    }
    .modal-content h2 { color: #1b5e20; margin-bottom: 14px; font-size: 1.2rem; }
    
    label { display: block; font-size: 0.9rem; font-weight: 600; margin: 10px 0 4px; color: #333; }
    input[type="text"], textarea, select {
      width: 100%;
      padding: 10px;
      border: 1px solid #ccc;
      border-radius: 8px;
      font-size: 0.95rem;
    }
    textarea { resize: vertical; min-height: 80px; }

    /* Seletor de Cores */
    .color-grid {
      display: grid;
      grid-template-columns: repeat(3, 1fr);
      gap: 8px;
      margin-top: 6px;
    }
    .color-option {
      display: flex;
      align-items: center;
      gap: 6px;
      padding: 8px;
      border: 2px solid #e0e0e0;
      border-radius: 8px;
      cursor: pointer;
      font-size: 0.8rem;
      font-weight: 600;
    }
    .color-option.selected { border-color: #111; background: #f0f7f0; }
    .color-dot { width: 16px; height: 16px; border-radius: 50%; flex-shrink: 0; }

    /* Botões do Modal */
    .modal-actions {
      display: flex;
      gap: 10px;
      margin-top: 18px;
    }
    .modal-actions button {
      flex: 1;
      padding: 12px;
      border: none;
      border-radius: 8px;
      font-weight: bold;
      cursor: pointer;
    }
    .btn-save { background: #1b5e20; color: white; }
    .btn-cancel { background: #e0e0e0; color: #333; }

    /* Lista Lateral / Gaveta de Pontos */
    #listaModal .point-item {
      border-left: 6px solid #ccc;
      background: #f9f9f9;
      padding: 10px;
      border-radius: 6px;
      margin-bottom: 10px;
      display: flex;
      justify-content: space-between;
      align-items: flex-start;
    }
    .point-info h4 { font-size: 1rem; color: #222; }
    .point-info p { font-size: 0.85rem; color: #555; margin-top: 4px; }
    .point-info small { font-size: 0.75rem; color: #888; display: block; margin-top: 4px; }
    .point-btns { display: flex; flex-direction: column; gap: 6px; }
    .point-btns button {
      padding: 5px 8px;
      font-size: 0.75rem;
      border: none;
      border-radius: 4px;
      cursor: pointer;
    }
    .btn-ir { background: #1976d2; color: white; }
    .btn-del { background: #d32f2f; color: white; }

    /* Ícone personalizado no mapa */
    .custom-pin {
      width: 28px;
      height: 28px;
      border-radius: 50% 50% 50% 0;
      transform: rotate(-45deg);
      border: 3px solid white;
      box-shadow: 0 2px 6px rgba(0,0,0,0.5);
    }
  </style>
</head>
<body>

  <header>
    <h1>🚜 AgroPonto GPS</h1>
    <div class="header-actions">
      <button onclick="abrirLista()">📋 Pontos (<span id="contador">0</span>)</button>
      <button onclick="exportarDados()">📤 Exportar</button>
    </div>
  </header>

  <div id="map"></div>

  <!-- Botões de Ação -->
  <div class="fab-container">
    <button class="btn-gps" onclick="centralizarGPS()">🎯 Minha Posição</button>
    <button class="btn-add" onclick="abrirModalNovoPonto()">📍 Marcar Ponto Atual</button>
  </div>

  <!-- Modal para Adicionar Ponto -->
  <div id="formModal" class="modal">
    <div class="modal-content">
      <h2>📌 Novo Ponto de Serviço</h2>
      
      <label for="titulo">Nome do Local / Serviço *</label>
      <input type="text" id="titulo" placeholder="Ex: Cerca rompida, Bebedouro 02, Praga na soja..." />

      <label>Cor e Categoria do Ícone *</label>
      <div class="color-grid" id="colorGrid"></div>

      <label for="descricao">Descrição Detalhada</label>
      <textarea id="descricao" placeholder="Descreva o problema, materiais necessários ou observações..."></textarea>

      <label>Coordenadas GPS</label>
      <input type="text" id="coordsDisplay" readonly style="background:#eee; font-size:0.85rem;" />

      <div class="modal-actions">
        <button class="btn-cancel" onclick="fecharModal('formModal')">Cancelar</button>
        <button class="btn-save" onclick="salvarPonto()">Salvar Ponto</button>
      </div>
    </div>
  </div>

  <!-- Modal de Lista de Pontos Salvos -->
  <div id="listaModal" class="modal">
    <div class="modal-content">
      <h2>📋 Pontos Marcados na Fazenda</h2>
      <div id="listaContainer" style="margin-top: 12px;"></div>
      <div class="modal-actions">
        <button class="btn-cancel" onclick="fecharModal('listaModal')">Fechar</button>
      </div>
    </div>
  </div>

  <script>
    // Categorias e Cores pré-definidas para a fazenda
    const CATEGORIAS = [
      { nome: 'Urgente / Reparo', cor: '#d32f2f' },   // Vermelho
      { nome: 'Água / Irrigação', cor: '#1976d2' },   // Azul
      { nome: 'Lavoura / Praga', cor: '#388e3c' },    // Verde
      { nome: 'Cerca / Infra', cor: '#f57c00' },      // Laranja
      { nome: 'Pecuária / Gado', cor: '#7b1fa2' },    // Roxo
      { nome: 'Maquinário', cor: '#fbc02d' }          // Amarelo
    ];

    let corSelecionada = CATEGORIAS[0];
    let pontos = JSON.parse(localStorage.getItem('fazenda_pontos')) || [];
    let marcadoresMapa = {};
    let posicaoAtual = null;
    let marcadorUsuario = null;

    // Inicializa o Mapa (Visão Satélite + Ruas)
    const map = L.map('map').setView([-15.7801, -47.9292], 4);

    // Camada de Satélite (Esri World Imagery - ótima para fazendas)
    const satelite = L.tileLayer('https://arcgisonline.com{z}/{y}/{x}', {
      attribution: 'Tiles &copy; Esri'
    }).addTo(map);

    // Camada de Rótulos / Estradas
    const estradas = L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      opacity: 0.35
    }).addTo(map);

    // Permite clicar em qualquer lugar do mapa para marcar um ponto manualmente também
    map.on('click', function(e) {
      if (confirm('Deseja marcar um ponto de serviço exatamente onde você clicou no mapa?')) {
        abrirModalComCoords(e.latlng.lat, e.latlng.lng);
      }
    });

    // Renderiza opções de cores no modal
    function montarSeletorCores() {
      const grid = document.getElementById('colorGrid');
      grid.innerHTML = '';
      CATEGORIAS.forEach((cat, idx) => {
        const div = document.createElement('div');
        div.className = 'color-option' + (idx === 0 ? ' selected' : '');
        div.innerHTML = `<span class="color-dot" style="background:${cat.cor}"></span><span>${cat.nome.split(' ')[0]}</span>`;
        div.onclick = () => {
          document.querySelectorAll('.color-option').forEach(el => el.classList.remove('selected'));
          div.classList.add('selected');
          corSelecionada = cat;
        };
        grid.appendChild(div);
      });
    }

    // Cria ícone colorido customizado
    function criarIconeColorido(cor) {
      return L.divIcon({
        className: '',
        html: `<div class="custom-pin" style="background-color: ${cor};"></div>`,
        iconSize:,
        iconAnchor:,
        popupAnchor: [0, -28]
      });
    }

    // Rastreamento contínuo de GPS
    function iniciarGPS() {
      if (!navigator.geolocation) {
        alert('Seu navegador não suporta GPS.');
        return;
      }
      navigator.geolocation.watchPosition(
        (pos) => {
          const lat = pos.coords.latitude;
          const lng = pos.coords.longitude;
          posicaoAtual = { lat, lng, precisao: Math.round(pos.coords.accuracy) };

          if (!marcadorUsuario) {

