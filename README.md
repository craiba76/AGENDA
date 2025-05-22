<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Agenda de Licitações</title>
  <style>
    * {
      box-sizing: border-box;
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    }

    body {
      margin: 0;
      padding: 0;
      background: #f5f7fa;
      color: #333;
    }

    h2 {
      text-align: center;
      margin-top: 30px;
    }

    #loginForm, #agenda {
      max-width: 900px;
      margin: 20px auto;
      background: white;
      padding: 30px;
      border-radius: 12px;
      box-shadow: 0 4px 12px rgba(0,0,0,0.1);
    }

    input, select {
      width: 100%;
      padding: 10px;
      margin: 5px 0 15px;
      border: 1px solid #ccc;
      border-radius: 6px;
      font-size: 16px;
    }

    button {
      background-color: #007bff;
      color: white;
      padding: 12px 18px;
      border: none;
      border-radius: 6px;
      font-size: 16px;
      margin: 10px 5px;
      cursor: pointer;
      transition: background-color 0.3s ease;
    }

    button:hover {
      background-color: #0056b3;
    }

    .button-group {
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
    }

    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 20px;
    }

    th, td {
      padding: 10px;
      border: 1px solid #ccc;
      text-align: left;
    }

    th {
      background-color: #007bff;
      color: white;
      position: sticky;
      top: 0;
    }

    .destaque {
      background-color: #28a745 !important;
      color: white;
    }
  </style>
</head>
<body>
  <div id="login">
    <h2>Login</h2>
    <form id="loginForm">
      <input type="text" id="usuario" placeholder="Usuário" required />
      <input type="password" id="senha" placeholder="Senha" required />
      <button type="submit">Acessar Agenda</button>
    </form>
  </div>

  <div id="agenda" style="display: none;">
    <h2>Agenda de Licitações</h2>
    <audio id="alertaSom" src="https://www.soundjay.com/button/beep-07.wav"></audio>
    
    <div class="button-group">
      <button onclick="ordenarTabela()">📅 Ordenar por Data</button>
      <button onclick="adicionarNovaLinha()">➕ Adicionar Nova Linha</button>
      <button onclick="fecharPlanilha()">💾 Fechar e Salvar</button>
    </div>

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
  </div>

  <script>
    const usuarios = { "BH": "123", "HDF": "321" };

    document.getElementById("loginForm").addEventListener("submit", function (event) {
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
      const tabela = document.getElementById("tabela").getElementsByTagName("tbody")[0];
      const novaLinha = tabela.insertRow();
      const colunas = ["empresa", "pregao", "uasg", "orgao", "estado", "portal", "descricao", "horario", "edital"];
      colunas.forEach(coluna => {
        const celula = novaLinha.insertCell();
        const input = document.createElement("input");
        input.type = coluna === "horario" ? "datetime-local" : "text";
        input.name = coluna;
        input.value = dado[coluna] || "";
        celula.appendChild(input);
      });
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
      destacarPrimeiraLinha();
    }

    function fecharPlanilha() {
      salvarDados();
      alert("Planilha salva com sucesso!");
    }

    function ordenarTabela() {
      const tabela = document.getElementById("tabela").getElementsByTagName("tbody")[0];
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
      const tabela = document.getElementById("tabela").getElementsByTagName("tbody")[0];
      Array.from(tabela.rows).forEach(row => row.classList.remove("destaque"));
      let primeiraData = null;
      let linhaDestaque = null;
      Array.from(tabela.rows).forEach(row => {
        const dataStr = row.cells[7].querySelector("input").value;
        const data = new Date(dataStr);
        if (!primeiraData || data < primeiraData) {
          primeiraData = data;
          linhaDestaque = row;
        }
      });
      if (linhaDestaque) linhaDestaque.classList.add("destaque");
    }
  </script>
</body>
</html>
