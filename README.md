<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>AGENDA BHDF - Licitações</title>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600&display=swap" rel="stylesheet">
  <style>
    * {
      box-sizing: border-box;
      font-family: 'Inter', sans-serif;
    }

    body {
      margin: 0;
      background: #f9fafb;
      color: #333;
    }

    .container {
      max-width: 1200px;
      margin: 2rem auto;
      padding: 1rem;
    }

    h1, h2 {
      text-align: center;
      color: #2c3e50;
    }

    #loginForm, #agenda {
      background: white;
      padding: 2rem;
      border-radius: 12px;
      box-shadow: 0 4px 12px rgba(0,0,0,0.1);
      max-width: 400px;
      margin: 2rem auto;
    }

    #loginForm input, #loginForm button {
      margin: 0.5rem 0;
      padding: 0.75rem;
      border-radius: 8px;
      border: 1px solid #ccc;
      font-size: 1rem;
    }

    #loginForm button {
      background-color: #007bff;
      color: white;
      border: none;
      cursor: pointer;
      transition: background 0.3s;
    }

    #loginForm button:hover {
      background-color: #0056b3;
    }

    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 2rem;
    }

    th, td {
      padding: 10px;
      border: 1px solid #ddd;
    }

    th {
      background-color: #007bff;
      color: white;
    }

    td input {
      width: 100%;
      padding: 5px;
      border: none;
      background: transparent;
    }

    .destaque {
      background-color: #28a745 !important;
      color: white;
    }

    .actions {
      display: flex;
      justify-content: center;
      gap: 1rem;
      margin-top: 1rem;
    }

    .actions button {
      padding: 0.75rem 1.5rem;
      border: none;
      border-radius: 8px;
      background-color: #6c757d;
      color: white;
      cursor: pointer;
      transition: background 0.3s;
    }

    .actions button:hover {
      background-color: #5a6268;
    }

    @media (max-width: 768px) {
      table, thead, tbody, th, td, tr {
        display: block;
      }
      th, td {
        padding: 1rem;
        text-align: right;
      }
      td::before {
        content: attr(data-label);
        float: left;
        font-weight: bold;
        color: #555;
      }
    }
  </style>
</head>
<body>
  <div class="container">
    <h1>AGENDA BHDF</h1>

    <div id="login">
      <h2>Login</h2>
      <form id="loginForm">
        <input type="text" id="usuario" placeholder="Usuário" required />
        <input type="password" id="senha" placeholder="Senha" required />
        <button type="submit">Acessar</button>
      </form>
    </div>

    <div id="agenda" style="display:none;">
      <h2>Agenda de Licitações</h2>
      <audio id="alertaSom" src="https://www.soundjay.com/button/beep-07.wav"></audio>

      <table id="tabela">
        <thead>
          <tr>
            <th>Empresa</th>
            <th>Pregão/Dispensa</th>
            <th>UASG</th>
            <th>Órgão</th>
            <th>Estado</th>
            <th>Portal</th>
            <th>Descrição</th>
            <th>Horário</th>
            <th>Edital</th>
          </tr>
        </thead>
        <tbody></tbody>
      </table>

      <div class="actions">
        <button onclick="ordenarTabela()">Ordenar por Data</button>
        <button onclick="adicionarNovaLinha()">Adicionar Linha</button>
        <button onclick="fecharPlanilha()">Salvar</button>
      </div>
    </div>
  </div>

  <script>
    const usuarios = { "BH": "123", "HDF": "321" };

    document.getElementById("loginForm").addEventListener("submit", function(event) {
      event.preventDefault();
      const usuario = document.getElementById("usuario").value.trim();
      const senha = document.getElementById("senha").value.trim();

      if (usuarios[usuario] === senha) {
        document.getElementById("login").style.display = "none";
        document.getElementById("agenda").style.display = "block";
        carregarDados();
        alert("Login bem-sucedido!");
      } else {
        alert("Usuário ou senha incorretos!");
      }
    });

    function adicionarNovaLinha(dado = {}) {
      const tabela = document.querySelector("#tabela tbody");
      const novaLinha = tabela.insertRow();
      const campos = ["empresa", "pregao", "uasg", "orgao", "estado", "portal", "descricao", "horario", "edital"];

      campos.forEach(campo => {
        const celula = novaLinha.insertCell();
        const input = document.createElement("input");
        input.name = campo;
        input.type = campo === "horario" ? "datetime-local" : "text";
        input.value = dado[campo] || "";
        celula.appendChild(input);
      });
    }

    function ordenarTabela() {
      const tabela = document.querySelector("#tabela tbody");
      const linhas = Array.from(tabela.rows);

      linhas.sort((a, b) => {
        const dataA = new Date(a.cells[7].querySelector("input").value);
        const dataB = new Date(b.cells[7].querySelector("input").value);
        return dataA - dataB;
      });

      linhas.forEach(linha => tabela.appendChild(linha));
      destacarPrimeiraLinha();
    }

    function destacarPrimeiraLinha() {
      const linhas = document.querySelectorAll("#tabela tbody tr");
      linhas.forEach(linha => linha.classList.remove("destaque"));

      let menorData = Infinity;
      let destaqueLinha = null;

      linhas.forEach(linha => {
        const dataStr = linha.cells[7].querySelector("input").value;
        const data = new Date(dataStr);
        if (data && data < menorData) {
          menorData = data;
          destaqueLinha = linha;
        }
      });

      if (destaqueLinha) destaqueLinha.classList.add("destaque");
    }

    function salvarDados() {
      const dados = [];
      document.querySelectorAll("#tabela tbody tr").forEach(linha => {
        const obj = {};
        linha.querySelectorAll("input").forEach(input => {
          obj[input.name] = input.value;
        });
        dados.push(obj);
      });
      localStorage.setItem("agendaLicitacoes", JSON.stringify(dados));
    }

    function carregarDados() {
      const dados = JSON.parse(localStorage.getItem("agendaLicitacoes")) || [];
      dados.forEach(dado => adicionarNovaLinha(dado));
    }

    function fecharPlanilha() {
      salvarDados();
      alert("Planilha salva com sucesso!");
    }
  </script>
</body>
</html>
