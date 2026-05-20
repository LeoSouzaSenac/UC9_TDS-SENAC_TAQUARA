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
* Clique em “Create a new project”
* Escolha: Windows Forms App (.NET)
* Nome: TodoApp

---

## 2. Instalar pacote NuGet

Instale:

* MySql.Data

---

## 3. Criar banco de dados

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

# 4. ELEMENTOS DA TELA (IMPORTANTE)

Crie no Form1 os seguintes componentes:

---

## TEXTBOX

* Name: `txtTask`
* Função: onde o usuário digita a tarefa

---

## BUTTON 1

* Name: `btnAdd`
* Text: `Adicionar`
* Função: inserir nova tarefa no banco

---

## BUTTON 2

* Name: `btnComplete`
* Text: `Concluir`
* Função: marcar tarefa como feita

---

## BUTTON 3

* Name: `btnDelete`
* Text: `Excluir`
* Função: apagar tarefa

---

## DATAGRIDVIEW

* Name: `gridTasks`
* Função: mostrar todas as tarefas do banco

---

## EVENTOS IMPORTANTES

No Form1:

* Form Load → `Form1_Load`
* btnAdd → `btnAdd_Click`
* btnComplete → `btnComplete_Click`
* btnDelete → `btnDelete_Click`

---

# 5. CÓDIGO COMPLETO

---

# Database.cs

```csharp id="db1"
using MySql.Data.MySqlClient;
using System;

namespace TodoApp
{
    public static class Database
    {
        // string de conexão com o banco de dados
        // aqui você coloca servidor, banco, usuário e senha
        private static string connectionString =
            "server=localhost;database=todo_app;uid=root;pwd=sua_senha;";

        public static MySqlConnection GetConnection()
        {
            // cria objeto de conexão com o MySQL
            MySqlConnection conn = new MySqlConnection(connectionString);

            try
            {
                // abre a conexão com o banco
                conn.Open();

                // retorna conexão aberta para uso
                return conn;
            }
            catch (Exception ex)
            {
                // mostra erro caso conexão falhe
                throw new Exception("Erro ao conectar: " + ex.Message);
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
        // id da tarefa no banco
        public int Id { get; set; }

        // texto da tarefa digitada pelo usuário
        public string Title { get; set; }

        // indica se a tarefa foi concluída ou não
        public bool IsDone { get; set; }
    }
}
```

---

# Form1.cs (TODA LÓGICA)

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
            // cria todos os componentes da tela
            InitializeComponent();
        }

        private void Form1_Load(object sender, EventArgs e)
        {
            // quando o sistema abre, carrega tarefas do banco
            LoadTasks();
        }

        private void btnAdd_Click(object sender, EventArgs e)
        {
            // pega texto digitado pelo usuário
            string title = txtTask.Text.Trim();

            // se estiver vazio, cancela operação
            if (string.IsNullOrWhiteSpace(title))
            {
                MessageBox.Show("Digite uma tarefa");
                return;
            }

            // abre conexão com banco
            using (var conn = Database.GetConnection())
            {
                // SQL com parâmetros (evita erro e ataque)
                string sql = "INSERT INTO tasks (user_id, title) VALUES (@user, @title)";

                using (var cmd = new MySqlCommand(sql, conn))
                {
                    // @user representa o id do usuário
                    cmd.Parameters.AddWithValue("@user", 1);

                    // @title recebe o texto digitado
                    cmd.Parameters.AddWithValue("@title", title);

                    // executa INSERT no banco
                    cmd.ExecuteNonQuery();
                }
            }

            // limpa campo de texto
            txtTask.Clear();

            // recarrega lista
            LoadTasks();
        }

        private void LoadTasks()
        {
            // abre conexão com banco
            using (var conn = Database.GetConnection())
            {
                // busca tarefas do usuário 1
                string sql = "SELECT * FROM tasks WHERE user_id = 1";

                using (var cmd = new MySqlCommand(sql, conn))
                {
                    using (var reader = cmd.ExecuteReader())
                    {
                        // cria tabela em memória
                        DataTable table = new DataTable();

                        // joga dados do banco dentro da tabela
                        table.Load(reader);

                        // mostra dados na tela
                        gridTasks.DataSource = table;
                    }
                }
            }
        }

        private void btnComplete_Click(object sender, EventArgs e)
        {
            // verifica se alguma linha foi selecionada
            if (gridTasks.CurrentRow == null)
                return;

            // pega id da linha selecionada
            int id = Convert.ToInt32(
                gridTasks.CurrentRow.Cells["id"].Value
            );

            // CurrentRow = linha selecionada
            // Cells["id"] = coluna id da tabela
            // Value = valor dentro da célula

            using (var conn = Database.GetConnection())
            {
                string sql = "UPDATE tasks SET is_done = TRUE WHERE id = @id";

                using (var cmd = new MySqlCommand(sql, conn))
                {
                    // substitui @id pelo id da tarefa
                    cmd.Parameters.AddWithValue("@id", id);

                    // executa UPDATE
                    cmd.ExecuteNonQuery();
                }
            }

            // atualiza lista
            LoadTasks();
        }

        private void btnDelete_Click(object sender, EventArgs e)
        {
            // verifica se linha foi selecionada
            if (gridTasks.CurrentRow == null)
                return;

            // pega id da tarefa selecionada
            int id = Convert.ToInt32(
                gridTasks.CurrentRow.Cells["id"].Value
            );

            using (var conn = Database.GetConnection())
            {
                string sql = "DELETE FROM tasks WHERE id = @id";

                using (var cmd = new MySqlCommand(sql, conn))
                {
                    // substitui @id pelo valor real
                    cmd.Parameters.AddWithValue("@id", id);

                    // executa DELETE
                    cmd.ExecuteNonQuery();
                }
            }

            // atualiza lista
            LoadTasks();
        }
    }
}
```

Só me fala o nível da turma.
