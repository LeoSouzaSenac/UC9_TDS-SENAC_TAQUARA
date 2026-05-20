# ENUNCIADO DA ATIVIDADE

Você deve criar um sistema de lista de tarefas (ToDo App) usando:

* C# Windows Forms
* MySQL
* CRUD completo (Create, Read, Update, Delete)

O sistema deve permitir:

1. Adicionar tarefas
2. Listar tarefas
3. Marcar tarefas como concluídas
4. Excluir tarefas

---

# PASSO A PASSO PARA CRIAR O PROJETO

## 1. Criar projeto

* Abra o Visual Studio
* Clique em "Create a new project"
* Escolha: Windows Forms App (.NET)
* Nome: TodoApp

---

## 2. Instalar pacote NuGet

Instale:

* MySql.Data

---

## 3. Criar banco de dados

Execute no MySQL:

```sql id="sql1"
CREATE DATABASE todo_app;

USE todo_app;

CREATE TABLE tasks (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NOT NULL,
    title VARCHAR(100) NOT NULL,
    is_done BOOLEAN DEFAULT FALSE
);
```

---

## 4. Criar arquivos no projeto

Crie:

* Database.cs
* TaskItem.cs
* Form1.cs

---

# CÓDIGO COMPLETO

---

# Program.cs

```csharp id="prog1"
using System;
using System.Windows.Forms;

namespace TodoApp
{
    internal static class Program
    {
        [STAThread]
        static void Main()
        {
            // Inicializa configurações do Windows Forms
            ApplicationConfiguration.Initialize();

            // Abre a primeira tela do sistema (Form1)
            Application.Run(new Form1());
        }
    }
}
```

---

# Database.cs

```csharp id="db1"
using MySql.Data.MySqlClient;
using System;

namespace TodoApp
{
    public static class Database
    {
        // String que contém as informações para conectar no banco
        private static string connectionString =
            "server=localhost;database=todo_app;uid=root;pwd=sua_senha;";

        public static MySqlConnection GetConnection()
        {
            // Cria objeto de conexão com o banco de dados
            MySqlConnection conn = new MySqlConnection(connectionString);

            try
            {
                // Abre a conexão com o banco
                conn.Open();

                // Retorna a conexão já aberta para ser usada
                return conn;
            }
            catch (Exception ex)
            {
                // Caso dê erro na conexão, mostra mensagem explicando o erro
                throw new Exception("Erro ao conectar com o banco: " + ex.Message);
            }
        }
    }
}
```

---

# TaskItem.cs

```csharp id="model1"
namespace TodoApp
{
    public class TaskItem
    {
        // Identificador único da tarefa no banco de dados
        public int Id { get; set; }

        // Texto da tarefa (ex: "Estudar C#")
        public string Title { get; set; }

        // Indica se a tarefa está concluída ou não
        public bool IsDone { get; set; }
    }
}
```

---

# Form1.cs (TODA LÓGICA DO SISTEMA)

```csharp id="form1"
using System;
using System.Data;
using System.Windows.Forms;
using MySql.Data.MySqlClient;

namespace TodoApp
{
    public partial class Form1 : Form
    {
        public Form1()
        {
            // Cria todos os elementos da tela (botões, tabela, etc)
            InitializeComponent();
        }

        private void Form1_Load(object sender, EventArgs e)
        {
            // Quando o sistema abre, carregamos todas as tarefas
            LoadTasks();
        }

        private void btnAdd_Click(object sender, EventArgs e)
        {
            // Pega o texto digitado pelo usuário na caixa de texto
            string title = txtTask.Text.Trim();

            // Se o campo estiver vazio, não deixa continuar
            if (string.IsNullOrWhiteSpace(title))
            {
                MessageBox.Show("Digite uma tarefa");
                return;
            }

            // Abre conexão com o banco de dados
            using (var conn = Database.GetConnection())
            {
                // SQL com espaços chamados parâmetros (@user e @title)
                string sql = "INSERT INTO tasks (user_id, title) VALUES (@user, @title)";

                using (var cmd = new MySqlCommand(sql, conn))
                {
                    // Aqui estamos preenchendo o valor do parâmetro @user
                    cmd.Parameters.AddWithValue("@user", 1);

                    // Aqui estamos preenchendo o valor do parâmetro @title
                    cmd.Parameters.AddWithValue("@title", title);

                    // Executa o comando no banco (INSERT, UPDATE ou DELETE)
                    cmd.ExecuteNonQuery();
                }
            }

            // Limpa o campo de texto depois de adicionar
            txtTask.Clear();

            // Atualiza a lista na tela
            LoadTasks();
        }

        private void LoadTasks()
        {
            // Abre conexão com o banco de dados
            using (var conn = Database.GetConnection())
            {
                // Busca todas as tarefas do usuário 1
                string sql = "SELECT * FROM tasks WHERE user_id = 1";

                using (var cmd = new MySqlCommand(sql, conn))
                {
                    using (var reader = cmd.ExecuteReader())
                    {
                        // Cria uma tabela em memória para armazenar os dados
                        DataTable table = new DataTable();

                        // Copia os dados do banco para a tabela em memória
                        table.Load(reader);

                        // Mostra os dados na tabela da tela (DataGridView)
                        gridTasks.DataSource = table;
                    }
                }
            }
        }

        private void btnComplete_Click(object sender, EventArgs e)
        {
            // Verifica se o usuário selecionou alguma linha da tabela
            if (gridTasks.CurrentRow == null)
                return;

            // Pega o ID da tarefa selecionada na tabela
            int id = Convert.ToInt32(
                gridTasks.CurrentRow.Cells["id"].Value
            );

            // Explicação:
            // gridTasks -> tabela da tela
            // CurrentRow -> linha selecionada pelo usuário
            // Cells["id"] -> coluna chamada "id"
            // Value -> valor dentro dessa célula

            using (var conn = Database.GetConnection())
            {
                // Atualiza a tarefa para "concluída"
                string sql = "UPDATE tasks SET is_done = TRUE WHERE id = @id";

                using (var cmd = new MySqlCommand(sql, conn))
                {
                    // Preenche o parâmetro @id com o valor da tarefa selecionada
                    cmd.Parameters.AddWithValue("@id", id);

                    // Executa o UPDATE no banco
                    cmd.ExecuteNonQuery();
                }
            }

            // Atualiza a lista
            LoadTasks();
        }

        private void btnDelete_Click(object sender, EventArgs e)
        {
            // Verifica se alguma linha foi selecionada
            if (gridTasks.CurrentRow == null)
                return;

            // Pega o ID da tarefa selecionada
            int id = Convert.ToInt32(
                gridTasks.CurrentRow.Cells["id"].Value
            );

            using (var conn = Database.GetConnection())
            {
                // Remove a tarefa do banco de dados
                string sql = "DELETE FROM tasks WHERE id = @id";

                using (var cmd = new MySqlCommand(sql, conn))
                {
                    // Preenche o parâmetro @id com o ID selecionado
                    cmd.Parameters.AddWithValue("@id", id);

                    // Executa o DELETE no banco
                    cmd.ExecuteNonQuery();
                }
            }

            // Atualiza a lista na tela
            LoadTasks();
        }
    }
}
```
