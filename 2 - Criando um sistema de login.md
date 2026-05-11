# **Tutorial simples: Login/Logout em C# Windows Forms (.NET) com MySQL e BCrypt**

## **🎯 Objetivo**

Criar um sistema simples de:

* Cadastro de usuário
* Login
* Logout
* Senhas criptografadas com BCrypt

---

# **1️⃣ Criar Projeto**

1. Abra o Visual Studio
2. Clique em **Create a new project**
3. Escolha:
   👉 Windows Forms App (.NET)
4. Nome: `LoginApp`
5. Criar

---

# **2️⃣ Instalar pacotes NuGet**

Clique com botão direito no projeto:

**Manage NuGet Packages → Install:**

* `MySql.Data`
* `BCrypt.Net-Next`

---

# **3️⃣ Criar Banco de Dados MySQL**

Execute no MySQL:

```sql
CREATE DATABASE login_system;

USE login_system;

CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL
);
```

---

# **4️⃣ Criar classe de conexão com banco**

Crie um arquivo: **Database.cs**

```csharp
using MySql.Data.MySqlClient;
using System;

namespace LoginApp
{
    public static class Database
    {
        // string de conexão com o banco
        private static string connectionString =
            "server=localhost;database=login_system;uid=root;pwd=sua_senha;";

        public static MySqlConnection GetConnection()
        {
            // cria conexão
            MySqlConnection conn = new MySqlConnection(connectionString);

            try
            {
                conn.Open(); // abre conexão com o banco
                return conn;
            }
            catch (Exception ex)
            {
                // erro de conexão
                throw new Exception("Erro ao conectar: " + ex.Message);
            }
        }
    }
}
```

---

# **5️⃣ Criar FormLogin**

No FormLogin adicione:

* TextBox: `txtUsername`
* TextBox: `txtPassword` (PasswordChar = '*')
* Button: `btnLogin`
* Button: `btnRegister`

---

## **📌 Código do FormLogin**

```csharp
using System;
using System.Windows.Forms;
using MySql.Data.MySqlClient;
using BCrypt.Net;

namespace LoginApp
{
    public partial class FormLogin : Form
    {
        public FormLogin()
        {
            InitializeComponent();
        }

        // =========================
        // BOTÃO DE LOGIN
        // =========================
        private void btnLogin_Click(object sender, EventArgs e)
        {
            string username = txtUsername.Text.Trim();
            string password = txtPassword.Text;

            // pega conexão com banco
            // O "using" garante que a conexão com o banco será fechada automaticamente
// assim que o bloco terminar, mesmo que aconteça um erro.
using (var conn = Database.GetConnection())
{
    // SQL que busca a senha (hash) do usuário no banco de dados
    // O @user é um parâmetro para evitar SQL Injection (ataques no banco)
    string sql = "SELECT password FROM users WHERE username=@user";

    // "using" também aqui garante que o comando será liberado depois de usado
    using (var cmd = new MySqlCommand(sql, conn))
    {
        // Aqui estamos passando o valor do username para o parâmetro @user
        // Isso evita concatenação direta de strings e aumenta a segurança
        cmd.Parameters.AddWithValue("@user", username);

        // Executa o comando SQL e retorna apenas o primeiro valor encontrado
        // Nesse caso, será a senha (hash) do usuário
        var result = cmd.ExecuteScalar();

        // Se "result" não for nulo, significa que o usuário foi encontrado no banco
        if (result != null)
        {
            // Converte o resultado para string (o hash da senha)
            string hash = result.ToString();

            // Compara a senha digitada com o hash armazenado no banco
            // BCrypt faz a comparação segura de senha (não compara texto puro)
            if (BCrypt.Net.BCrypt.Verify(password, hash))
            {
                // Se a senha estiver correta, mostra mensagem de sucesso
                MessageBox.Show("Login realizado!");

                // Cria e abre a tela principal do sistema
                FormMain main = new FormMain(username);
                main.Show();

                // Esconde a tela de login atual
                this.Hide();
            }
            else
            {
                // Se o hash não bater com a senha digitada
                MessageBox.Show("Senha incorreta!");
            }
        }
        else
        {
            // Se não encontrou nenhum usuário com esse username
            MessageBox.Show("Usuário não encontrado!");
        }
    }
}

        // =========================
        // BOTÃO DE CADASTRO
        // =========================
        private void btnRegister_Click(object sender, EventArgs e)
        {
            string username = txtUsername.Text.Trim();
            string password = txtPassword.Text;

            // cria hash da senha (não salva senha pura!)
            string hash = BCrypt.Net.BCrypt.HashPassword(password);

            using (var conn = Database.GetConnection())
            {
                string sql = "INSERT INTO users (username, password) VALUES (@user, @pass)";

                using (var cmd = new MySqlCommand(sql, conn))
                {
                    cmd.Parameters.AddWithValue("@user", username);
                    cmd.Parameters.AddWithValue("@pass", hash);

                    try
                    {
                        cmd.ExecuteNonQuery();
                        MessageBox.Show("Usuário criado com sucesso!");
                    }
                    catch
                    {
                        MessageBox.Show("Erro: usuário já existe!");
                    }
                }
            }
        }
    }
}
```

---

# **6️⃣ Criar FormMain (tela após login)**

Adicione:

* Label: `lblWelcome`
* Button: `btnLogout`

---

## **📌 Código FormMain**

```csharp
using System;
using System.Windows.Forms;

namespace LoginApp
{
    public partial class FormMain : Form
    {
        public FormMain(string username)
        {
            InitializeComponent();

            // mostra nome do usuário logado
            lblWelcome.Text = "Bem-vindo, " + username;
        }

        // =========================
        // LOGOUT
        // =========================
        private void btnLogout_Click(object sender, EventArgs e)
        {
            // volta para login
            FormLogin login = new FormLogin();
            login.Show();

            // fecha tela atual
            this.Close();
        }
    }
}
```



Se quiser, posso fazer a **versão ainda mais simples (sem banco, só memória)** ou uma **versão com nível intermediário depois dessa**.
