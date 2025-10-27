# Interatividade-e-Funcionalidades
Código JavaScript Modular Estrutura de pastas organizada (pastas, HTML, imagens, CSS e JS); No ar
quivo organizar os códigos por funcionalidade.

[meu-projeto.zip](https://github.com/user-attachments/files/23172660/meu-projeto.zip)

<meu-projeto>/

    
   <index.html>

   <css>
        </style.css>

   <js>
      <main.js        ← lógica principal (SPA, inicialização e tema)
      ui.js          ← manipulação do DOM, criação e edição de itens
      storage.js     ← funções de salvar, carregar, excluir e limpar itens
      utils.js       ← funções auxiliares (debounce, escapeHtml, etc.)
 
  imagens
     logo.png
 
  sons 
    click.mp3   (opcional — se quiser adicionar sons nos botões)


<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Lista Modular 6.0</title>
  <style>
    :root {
      --bg: #f5f5f5;
      --text: #222;
      --header: #4b4acb;
      --btn-bg: #4b4acb;
      --btn-text: #fff;
      --input-bg: #fff;
      --input-text: #222;
    }

    body {
      margin: 0;
      font-family: sans-serif;
      background-color: var(--bg);
      color: var(--text);
      transition: background 0.3s, color 0.3s;
    }

    .dark-mode {
      --bg: #222;
      --text: #f5f5f5;
      --header: #8888ff;
      --btn-bg: #8888ff;
      --btn-text: #222;
      --input-bg: #333;
      --input-text: #f5f5f5;
    }

    header {
      background-color: var(--header);
      color: var(--btn-text);
      padding: 1rem;
      text-align: center;
    }

    .header-top {
      display: flex;
      justify-content: space-between;
      align-items: center;
    }

    nav {
      display: flex;
      justify-content: center;
      gap: 1rem;
      margin: 1rem 0;
    }

    button {
      cursor: pointer;
      padding: 0.5rem 1rem;
      margin-left: 0.5rem;
      border: none;
      background-color: var(--btn-bg);
      color: var(--btn-text);
      border-radius: 4px;
    }

    main {
      padding: 1rem;
      max-width: 600px;
      margin: auto;
    }

    input[type="text"] {
      padding: 0.5rem;
      margin-bottom: 0.5rem;
      width: calc(100% - 1rem);
      border: 1px solid #ccc;
      border-radius: 4px;
      background-color: var(--input-bg);
      color: var(--input-text);
    }

    ul {
      list-style: none;
      padding: 0;
    }

    li {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 0.5rem;
      border-bottom: 1px solid #ccc;
      transition: background 0.3s;
    }

    li.editando input {
      width: 100%;
      padding: 0.25rem;
    }

    .actions button {
      margin-left: 0.5rem;
    }

    /* animações */
    @keyframes slideIn { from { opacity:0; transform: translateX(-20px);} to {opacity:1; transform: translateX(0);} }
    @keyframes slideOut { from { opacity:1; transform: translateX(0);} to {opacity:0; transform: translateX(20px);} }
    .animar-entrada { animation: slideIn 0.3s forwards; }
    .animar-saida { animation: slideOut 0.3s forwards; }
  </style>
</head>
<body>
  <header>
    <div class="header-top">
      <img src="https://via.placeholder.com/80x80?text=Logo" alt="Logo" width="80" />
      <button id="btnTema">🌙 Modo Escuro</button>
    </div>
    <h1>Minha Lista 6.0</h1>
  </header>

  <nav>
    <button data-page="home">Home</button>
    <button data-page="lista">Minha Lista</button>
    <button data-page="sobre">Sobre</button>
  </nav>

  <main></main>

  <script>
    // ======= STORAGE =======
    function salvarItem(item) {
      const lista = carregarItens();
      lista.push(item);
      localStorage.setItem('itens', JSON.stringify(lista));
    }
    function carregarItens() {
      return JSON.parse(localStorage.getItem('itens')) || [];
    }
    function excluirItem(item) {
      let lista = carregarItens();
      lista = lista.filter(i => i !== item);
      localStorage.setItem('itens', JSON.stringify(lista));
    }
    function editarItem(antigo, novo) {
      const lista = carregarItens();
      const index = lista.indexOf(antigo);
      if (index !== -1) {
        lista[index] = novo;
        localStorage.setItem('itens', JSON.stringify(lista));
      }
    }
    function limparTudo() {
      localStorage.removeItem('itens');
    }
    function salvarTema(tema) {
      localStorage.setItem('tema', tema);
    }
    function carregarTema() {
      return localStorage.getItem('tema') || 'light';
    }

    // ======= UTILS =======
    function escapeHtml(str) {
      return String(str).replace(/&/g,"&amp;").replace(/</g,"&lt;").replace(/>/g,"&gt;").replace(/"/g,"&quot;").replace(/'/g,"&#039;");
    }
    function debounce(fn, wait=250){
      let t;
      return (...args) => { clearTimeout(t); t = setTimeout(()=>fn(...args), wait); }
    }

    // ======= UI =======
    function criarItemNaTela(texto){
      const lista = document.querySelector('#listaItens');
      const li = document.createElement('li');
      li.classList.add('animar-entrada');
      li.innerHTML = `
        <span class="texto">${escapeHtml(texto)}</span>
        <div class="actions">
          <button class="btnEditar">Editar</button>
          <button class="btnExcluir">Excluir</button>
        </div>
      `;
      lista.appendChild(li);
      li.addEventListener('animationend', ev => { if(ev.animationName==='slideIn') li.classList.remove('animar-entrada'); });
      return li;
    }

    function removerItemComAnimacao(li, callback){
      li.classList.add('animar-saida');
      li.addEventListener('animationend', ev => {
        if(ev.animationName==='slideOut'){
          li.remove();
          if(callback) callback();
        }
      });
    }

    function limparListaNaTela(){
      const lista = document.querySelector('#listaItens');
      if (lista) lista.innerHTML='';
    }

    function atualizarItemNaTela(li, novoTexto){
      li.classList.remove('editando');
      li.querySelector('.texto').textContent = novoTexto;
    }

    function filtrarLista(termo){
      const itens = document.querySelectorAll('#listaItens li');
      const t = termo.trim().toLowerCase();
      itens.forEach(li=>{
        const texto = li.querySelector('.texto').textContent.toLowerCase();
        li.style.display = (!t || texto.includes(t)) ? 'flex' : 'none';
      });
    }

    function aplicarTema(tema){
      const body=document.body;
      const btnTema=document.querySelector('#btnTema');
      if(tema==='dark'){
        body.classList.add('dark-mode');
        btnTema.textContent='☀️ Modo Claro';
      }else{
        body.classList.remove('dark-mode');
        btnTema.textContent='🌙 Modo Escuro';
      }
    }

    // ======= LISTA =======
    function inicializarLista(){
      const input = document.querySelector('#inputTexto');
      const inputBusca = document.querySelector('#inputBusca');
      const btnSalvar = document.querySelector('#btnSalvar');
      const btnLimpar = document.querySelector('#btnLimpar');
      const lista = document.querySelector('#listaItens');

      carregarItens().forEach(t=>criarItemNaTela(t));

      btnSalvar.addEventListener('click', ()=>{
        const texto = input.value.trim();
        if(!texto) return alert('Digite algo!');
        salvarItem(texto);
        criarItemNaTela(texto);
        input.value='';
      });

      input.addEventListener('keydown', e=>{ if(e.key==='Enter') btnSalvar.click(); });

      lista.addEventListener('click', e=>{
        const li = e.target.closest('li');
        if(!li) return;

        if(e.target.classList.contains('btnExcluir')){
          const texto = li.querySelector('.texto').textContent;
          removerItemComAnimacao(li, ()=>excluirItem(texto));
        }

        if(e.target.classList.contains('btnEditar')){
          if(li.classList.contains('editando')) return;
          li.classList.add('editando');
          const textoAntigo = li.querySelector('.texto').textContent;
          const inputEdit = document.createElement('input');
          inputEdit.type='text';
          inputEdit.value=textoAntigo;
          inputEdit.className='inputEdit';
          li.querySelector('.texto').replaceWith(inputEdit);
          inputEdit.focus();

          const salvarEdicao = ()=>{
            const novoTexto=inputEdit.value.trim();
            if(!novoTexto) return alert('Texto vazio!');
            editarItem(textoAntigo, novoTexto);
            atualizarItemNaTela(li, novoTexto);
          };

          inputEdit.addEventListener('keydown', ev=>{
            if(ev.key==='Enter') salvarEdicao();
            if(ev.key==='Escape') atualizarItemNaTela(li, textoAntigo);
          });
          inputEdit.addEventListener('blur', salvarEdicao);
        }
      });

      btnLimpar.addEventListener('click', ()=>{
        if(!confirm('Tem certeza que deseja apagar tudo?')) return;
        limparTudo();
        limparListaNaTela();
      });

      const handleBusca = debounce(()=>filtrarLista(inputBusca.value), 180);
      inputBusca.addEventListener('input', handleBusca);
    }

    // ======= SPA / TEMPLATES =======
    const templates = {
      home: `
        <h2>Bem-vindo(a) à Lista 6.0</h2>
        <p>Use o menu acima pra navegar.</p>
      `,
      lista: `
        <div class="controls">
          <input type="text" id="inputTexto" placeholder="Digite algo..." />
          <button id="btnSalvar">Salvar</button>
          <button id="btnLimpar">Limpar Tudo</button>
        </div>
        <input type="text" id="inputBusca" placeholder="Pesquisar item..." />
        <ul id="listaItens"></ul>
      `,
      sobre: `
        <h2>Sobre</h2>
        <p>Aplicativo criado para testar manipulação do DOM e sistema SPA básico.</p>
      `
    };

    function carregarTemplate(nome) {
      const main = document.querySelector('main');
      main.innerHTML = templates[nome] || '<p>Página não encontrada</p>';
      if (nome === 'lista') inicializarLista();
    }

    // ======= MAIN =======
    document.addEventListener('DOMContentLoaded', ()=>{
      aplicarTema(carregarTema());
      carregarTemplate('home');

      document.querySelector('nav').addEventListener('click', e => {
        if (e.target.dataset.page) {
          carregarTemplate(e.target.dataset.page);
        }
      });

      document.querySelector('#btnTema').addEventListener('click', ()=>{
        const novoTema = document.body.classList.toggle('dark-mode')?'dark':'light';
        salvarTema(novoTema);
        aplicarTema(novoTema);
      });
    });
  </script>
</body>
</html>
