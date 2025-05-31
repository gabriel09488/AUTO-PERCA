<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>AUTO PEÇA SM - Controle de Estoque e Vendas</title>
<style>
  body {
    font-family: Arial, sans-serif;
    background: #e3f2fd;
    margin: 0; padding: 0;
    color: #222;
  }
  header {
    background: #1565c0;
    color: white;
    padding: 20px;
    text-align: center;
    font-size: 1.8em;
    font-weight: bold;
  }
  main {
    max-width: 900px;
    margin: 30px auto;
    background: white;
    padding: 20px 30px;
    border-radius: 8px;
    box-shadow: 0 0 15px #999;
  }
  h2 {
    color: #1565c0;
  }
  input, button, select {
    font-size: 1em;
    padding: 8px 12px;
    margin: 6px 0 12px 0;
    border-radius: 5px;
    border: 1px solid #ccc;
  }
  button {
    background: #1565c0;
    color: white;
    border: none;
    cursor: pointer;
    transition: background-color 0.3s ease;
  }
  button:hover {
    background: #0d47a1;
  }
  table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 15px;
  }
  th, td {
    border: 1px solid #aaa;
    padding: 10px;
    text-align: left;
  }
  th {
    background: #1565c0;
    color: white;
  }
  .flex-row {
    display: flex;
    gap: 20px;
    flex-wrap: wrap;
  }
  .flex-col {
    flex: 1;
    min-width: 280px;
  }
  .hidden {
    display: none;
  }
</style>
</head>
<body>

<header>AUTO PEÇA SM - Controle de Estoque e Vendas</header>

<main>

  <section id="cadastroPecas">
    <h2>Cadastro de Peças</h2>
    <div class="flex-row">
      <div class="flex-col">
        <input type="text" id="nomePeca" placeholder="Nome da Peça" />
        <input type="text" id="codigoPeca" placeholder="Código da Peça" />
        <input type="number" id="quantidadePeca" placeholder="Quantidade" min="0" />
      </div>
      <div class="flex-col">
        <input type="number" step="0.01" id="precoPeca" placeholder="Preço (R$)" min="0" />
        <input type="text" id="fornecedorPeca" placeholder="Fornecedor" />
        <button onclick="adicionarPeca()">Adicionar Peça</button>
      </div>
    </div>
  </section>

  <section id="estoqueAtual">
    <h2>Estoque Atual</h2>
    <table id="tabelaEstoque">
      <thead>
        <tr>
          <th>Nome</th>
          <th>Código</th>
          <th>Quantidade</th>
          <th>Preço (R$)</th>
          <th>Fornecedor</th>
          <th>Ações</th>
        </tr>
      </thead>
      <tbody></tbody>
    </table>
  </section>

  <section id="vendas">
    <h2>Registrar Venda</h2>
    <input type="text" id="codigoVenda" placeholder="Código da Peça" />
    <input type="number" id="quantidadeVenda" placeholder="Quantidade" min="1" value="1" />
    <input type="text" id="clienteVenda" placeholder="Cliente" />
    <button onclick="registrarVenda()">Registrar Venda</button>
  </section>

  <section id="historicoVendas">
    <h2>Histórico de Vendas</h2>
    <table id="tabelaHistorico">
      <thead>
        <tr>
          <th>Nome</th>
          <th>Código</th>
          <th>Quantidade</th>
          <th>Cliente</th>
          <th>Data/Hora</th>
        </tr>
      </thead>
      <tbody></tbody>
    </table>
  </section>

</main>

<script>
  let estoque = [];
  let vendas = [];

  // Carregar dados do localStorage
  function carregarDados() {
    const est = localStorage.getItem('estoqueAutoPecaSM');
    const ven = localStorage.getItem('vendasAutoPecaSM');
    if (est) estoque = JSON.parse(est);
    if (ven) vendas = JSON.parse(ven);
  }

  // Salvar dados no localStorage
  function salvarDados() {
    localStorage.setItem('estoqueAutoPecaSM', JSON.stringify(estoque));
    localStorage.setItem('vendasAutoPecaSM', JSON.stringify(vendas));
  }

  // Atualiza a tabela do estoque
  function atualizarEstoque() {
    const tbody = document.querySelector('#tabelaEstoque tbody');
    tbody.innerHTML = '';
    estoque.forEach((p, i) => {
      const tr = document.createElement('tr');
      tr.innerHTML = `
        <td>${p.nome}</td>
        <td>${p.codigo}</td>
        <td>${p.quantidade}</td>
        <td>${p.preco.toFixed(2)}</td>
        <td>${p.fornecedor}</td>
        <td>
          <button onclick="editarPeca(${i})">Editar</button>
          <button onclick="excluirPeca(${i})">Excluir</button>
        </td>
      `;
      tbody.appendChild(tr);
    });
  }

  // Atualiza a tabela de histórico de vendas
  function atualizarHistorico() {
    const tbody = document.querySelector('#tabelaHistorico tbody');
    tbody.innerHTML = '';
    vendas.slice().reverse().forEach(v => {
      const tr = document.createElement('tr');
      tr.innerHTML = `
        <td>${v.nome}</td>
        <td>${v.codigo}</td>
        <td>${v.quantidade}</td>
        <td>${v.cliente}</td>
        <td>${v.dataHora}</td>
      `;
      tbody.appendChild(tr);
    });
  }

  // Adiciona uma peça nova
  function adicionarPeca() {
    const nome = document.getElementById('nomePeca').value.trim();
    const codigo = document.getElementById('codigoPeca').value.trim();
    const quantidade = parseInt(document.getElementById('quantidadePeca').value);
    const preco = parseFloat(document.getElementById('precoPeca').value);
    const fornecedor = document.getElementById('fornecedorPeca').value.trim();

    if (!nome || !codigo || isNaN(quantidade) || isNaN(preco) || !fornecedor) {
      alert('Por favor, preencha todos os campos corretamente.');
      return;
    }

    if (estoque.some(p => p.codigo === codigo)) {
      alert('Já existe uma peça com esse código.');
      return;
    }

    estoque.push({ nome, codigo, quantidade, preco, fornecedor });
    salvarDados();
    atualizarEstoque();
    limparCamposCadastro();
  }

  // Limpa os campos do cadastro
  function limparCamposCadastro() {
    document.getElementById('nomePeca').value = '';
    document.getElementById('codigoPeca').value = '';
    document.getElementById('quantidadePeca').value = '';
    document.getElementById('precoPeca').value = '';
    document.getElementById('fornecedorPeca').value = '';
  }

  // Edita uma peça (carrega dados nos campos e exclui a peça antiga)
  function editarPeca(index) {
    const p = estoque[index];
    document.getElementById('nomePeca').value = p.nome;
    document.getElementById('codigoPeca').value = p.codigo;
    document.getElementById('quantidadePeca').value = p.quantidade;
    document.getElementById('precoPeca').value = p.preco;
    document.getElementById('fornecedorPeca').value = p.fornecedor;
    estoque.splice(index, 1);
    salvarDados();
    atualizarEstoque();
  }

  // Excluir peça do estoque
  function excluirPeca(index) {
    if (confirm('Deseja realmente excluir esta peça?')) {
      estoque.splice(index, 1);
      salvarDados();
      atualizarEstoque();
    }
  }

  // Registrar venda e diminuir estoque
  function registrarVenda() {
    const codigo = document.getElementById('codigoVenda').value.trim();
    const quantidadeVenda = parseInt(document.getElementById('quantidadeVenda').value);
    const cliente = document.getElementById('clienteVenda').value.trim();

    if (!codigo || isNaN(quantidadeVenda) || quantidadeVenda < 1 || !cliente) {
      alert('Preencha todos os campos da venda corretamente.');
      return;
    }

    const peca = estoque.find(p => p.codigo === codigo);
    if (!peca) {
      alert('Peça não encontrada no estoque.');
      return;
    }

    if (peca.quantidade < quantidadeVenda) {
      alert(`Estoque insuficiente. Temos apenas ${peca.quantidade} unidades.`);
      return;
    }

    peca.quantidade -= quantidadeVenda;
    const dataHora = new Date().toLocaleString();

    vendas.push({
      nome: peca.nome,
      codigo,
      quantidade:
