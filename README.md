<!DOCTYPE html>
<html lang="pt">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Painel Administrativo - Pedidos</title>
    <!-- Importando Firebase -->
    <script type="module" src="https://www.gstatic.com/firebasejs/10.0.0/firebase-app.js"></script>
    <script type="module" src="https://www.gstatic.com/firebasejs/10.0.0/firebase-database.js"></script>
    <style>
        /* Estilo geral da página */
        body {
            font-family: Arial, sans-serif;
            background-color: #121212;
            color: #e0e0e0;
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
        }

        .container {
            width: 100%;
            max-width: 400px;
            background: #1f1f1f;
            padding: 30px;
            border-radius: 10px;
            box-shadow: 0 4px 10px rgba(0, 0, 0, 0.3);
        }

        h2 {
            color: #ff7f00; /* Cor laranja para título */
            margin-bottom: 20px;
        }

        input[type="password"] {
            width: 100%;
            padding: 10px;
            margin: 10px 0;
            border: 1px solid #333;
            border-radius: 5px;
            background-color: #2c2c2c;
            color: #e0e0e0;
            font-size: 16px;
        }

        .button {
            width: 100%;
            padding: 12px;
            border-radius: 5px;
            cursor: pointer;
            font-size: 16px;
            background-color: #ff7f00; /* Laranja */
            color: white;
            border: none;
        }

        .button:hover {
            background-color: #e76f00;
        }

        .error {
            color: red;
            text-align: center;
            margin-top: 20px;
        }

        /* Estilos do painel administrativo */
        .panel {
            display: none;
            width: 100%;
            max-width: 800px;
            background: #1f1f1f;
            padding: 30px;
            border-radius: 10px;
            box-shadow: 0 4px 10px rgba(0, 0, 0, 0.3);
            text-align: center;
        }

        table {
            width: 100%;
            margin-top: 20px;
            border-collapse: collapse;
        }

        th, td {
            padding: 12px;
            border: 1px solid #333;
            text-align: left;
        }

        th {
            background-color: #333;
        }

        tr:nth-child(even) {
            background-color: #2c2c2c;
        }

        /* Cor de fundo para status */
        .status {
            font-weight: bold;
        }

        .status.adicionado {
            color: green;
        }

        .status.nao-adicionado {
            color: red;
        }

        .status.pendente {
            color: orange;
        }

        .button.add {
            background-color: #4caf50; /* Verde */
            color: white;
        }

        .button.add:hover {
            background-color: #45a049;
        }

        .button.remove {
            background-color: #f44336; /* Vermelho */
            color: white;
        }

        .button.remove:hover {
            background-color: #e53935;
        }

        .button.delete {
            background-color: #ff7f00; /* Laranja */
            color: white;
        }

        .button.delete:hover {
            background-color: #e76f00;
        }
    </style>
</head>
<body>
    <!-- Tela de Login -->
    <div class="container" id="loginContainer">
        <h2>Login - Painel Administrativo</h2>
        <input type="password" id="passwordInput" placeholder="Digite a senha">
        <button class="button" id="entrarButton">Entrar</button>
        <div id="errorMessage" class="error"></div>
    </div>

    <!-- Painel Administrativo -->
    <div class="panel" id="adminPanel">
        <h2>Painel Administrativo - Pedidos</h2>
        <table id="pedidoTable">
            <thead>
                <tr>
                    <th>Filme/Série</th>
                    <th>Pessoa</th>
                    <th>Status</th>
                    <th>Ações</th>
                </tr>
            </thead>
            <tbody>
                <!-- Os pedidos serão carregados aqui -->
            </tbody>
        </table>
    </div>

    <script type="module">
        // Inicializa o Firebase
        import { initializeApp } from "https://www.gstatic.com/firebasejs/10.0.0/firebase-app.js";
        import { getDatabase, ref, onValue, remove, update } from "https://www.gstatic.com/firebasejs/10.0.0/firebase-database.js";

        const firebaseConfig = {
            apiKey: "AIzaSyAkmmFDwhkcY6Z4UJMYHPqHrTSPTfvop8s",
            authDomain: "blueplayweb.firebaseapp.com",
            databaseURL: "https://blueplayweb-default-rtdb.firebaseio.com",
            projectId: "blueplayweb",
            storageBucket: "blueplayweb.appspot.com",
            messagingSenderId: "1010274882829",
            appId: "1:1010274882829:web:8188c8a0eb7b02d4d4cc44",
            measurementId: "G-Q5SDTRY084"
        };

        // Inicializa o app Firebase
        const app = initializeApp(firebaseConfig);
        const database = getDatabase(app);

        const pedidoTable = document.getElementById("pedidoTable").getElementsByTagName("tbody")[0];
        const loginContainer = document.getElementById("loginContainer");
        const adminPanel = document.getElementById("adminPanel");
        const errorMessage = document.getElementById("errorMessage");

        const senhaCorreta = "Blue8486"; // Senha que dá acesso ao painel administrativo

        // Função para verificar a senha
        function verificarSenha() {
            const senhaInput = document.getElementById("passwordInput").value;

            if (senhaInput === senhaCorreta) {
                loginContainer.style.display = "none"; // Esconde a tela de login
                adminPanel.style.display = "block";   // Exibe o painel administrativo
                carregarPedidos(); // Carrega os pedidos
            } else {
                errorMessage.textContent = "Senha incorreta. Tente novamente.";
            }
        }

        // Função para carregar os pedidos da base de dados
        function carregarPedidos() {
            const pedidosRef = ref(database, 'pedidos');
            onValue(pedidosRef, (snapshot) => {
                pedidoTable.innerHTML = ''; // Limpa a tabela antes de carregar novos dados
                if (snapshot.exists()) {
                    snapshot.forEach((childSnapshot) => {
                        const pedido = childSnapshot.val();
                        const key = childSnapshot.key;

                        // Cria uma nova linha para o pedido
                        const row = pedidoTable.insertRow();
                        row.insertCell(0).textContent = pedido.filme;
                        row.insertCell(1).textContent = pedido.pessoa;

                        const statusCell = row.insertCell(2);
                        const statusSpan = document.createElement("span");
                        statusSpan.id = `status-${key}`;
                        statusSpan.className = `status ${pedido.status.toLowerCase().replace(' ', '-')}`;
                        statusSpan.textContent = pedido.status;
                        statusCell.appendChild(statusSpan);

                        const actionsCell = row.insertCell(3);

                        // Botão "Adicionar"
                        const addButton = document.createElement("button");
                        addButton.textContent = "Adicionar ao Add-on";
                        addButton.className = "button add";
                        addButton.addEventListener("click", () => alterarStatus(key, "Adicionado"));

                        // Botão "Não Adicionado"
                        const removeButton = document.createElement("button");
                        removeButton.textContent = "Não Adicionado";
                        removeButton.className = "button remove";
                        removeButton.addEventListener("click", () => alterarStatus(key, "Não Adicionado"));

                        // Botão "Deletar"
                        const deleteButton = document.createElement("button");
                        deleteButton.textContent = "Deletar";
                        deleteButton.className = "button delete";
                        deleteButton.addEventListener("click", () => deletarPedido(key));

                        actionsCell.appendChild(addButton);
                        actionsCell.appendChild(removeButton);
                        actionsCell.appendChild(deleteButton);
                    });
                } else {
                    console.log("Nenhum pedido encontrado.");
                }
            });
        }

        // Função para alterar o status de um pedido
        function alterarStatus(key, status) {
            const statusRef = ref(database, 'pedidos/' + key);
            update(statusRef, {
                status: status
            }).then(() => {
                const statusElement = document.getElementById(`status-${key}`);
                statusElement.innerText = status;
                statusElement.className = `status ${status.toLowerCase().replace(' ', '-')}`;
            }).catch(error => {
                alert('Erro ao alterar status: ' + error);
            });
        }

        // Função para deletar um pedido
        function deletarPedido(key) {
            const pedidoRef = ref(database, 'pedidos/' + key);
            remove(pedidoRef).then(() => {
                carregarPedidos(); // Recarrega a tabela após deletar
            }).catch((error) => {
                alert('Erro ao deletar pedido: ' + error);
            });
        }

        // Adicionando o evento de login
        document.getElementById("entrarButton").addEventListener("click", verificarSenha);
    </script>
</body>
</html>
