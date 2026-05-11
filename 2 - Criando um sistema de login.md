# **Tutorial: Login/Logout em C# Windows Forms com MVC, Services, MySQL e BCrypt**

## **1️⃣ Criar Projeto no Visual Studio 2026**

1. Abra o Visual Studio 2026.
2. Clique em **Create a new project**.
3. Escolha **Windows Forms App (.NET 10)**.
4. Nome do projeto: `LoginApp`.
5. Local: escolha sua pasta.
6. Clique em **Create**.

---

## **2️⃣ Instalar Pacotes NuGet**

No **Solution Explorer**:

1. Clique com o botão direito em **Dependencies → Manage NuGet Packages**.
2. Procure e instale:

   * `MySql.Data` → para MySQL.
   * `BCrypt.Net-Next` → para criptografia de senha.

---

## **3️⃣ Estrutura de Pastas (MVC + Services)**

Organize o projeto assim:

```
LoginApp/
├── Models/
│   └── User.cs
├── Views/
│   ├── FormLogin.cs
│   └── FormMain.cs
├── Controllers/
│   └── UserController.cs
├── Services/
│   └── UserService.cs
└── Database.cs
```

Essa estrutura mantém **separação de responsabilidades**:

* **Models** → representação dos dados.
* **Views** → telas (Forms).
* **Controllers** → lógica de interface e fluxo.
* **Services** → regras de negócio, acesso ao banco, hashing.

---

## **4️⃣ Criar Model: User**

```csharp
// Models/User.cs
namespace LoginApp.Models
{
    public class User
    {
        public int Id { get; set; }
        public string Username { get; set; }
        public string Password { get; set; } // hash
    }
}
```

---

## **5️⃣ Criar Conexão com MySQL**

```csharp
// Database.cs
using MySql.Data.MySqlClient;
using System;

namespace LoginApp
{
    public static class Database
    {
        private static string connectionString = "server=localhost;database=login_system;uid=root;pwd=senha;";

        public static MySqlConnection GetConnection()
        {
            var conn = new MySqlConnection(connectionString);
            try
            {
                conn.Open();
                return conn;
            }
            catch (Exception ex)
            {
                throw new Exception("Erro ao conectar ao banco: " + ex.Message);
            }
        }
    }
}
```

> [!NOTE]
> Troque `senha` pela sua senha do MySQL.

---

## **6️⃣ Criar Camada Service: UserService**

```csharp
// Services/UserService.cs
using LoginApp.Models;
using MySql.Data.MySqlClient;
using BCrypt.Net;

namespace LoginApp.Services
{
    public class UserService
    {
        public bool RegisterUser(User user)
        {
            string hashedPassword = BCrypt.Net.BCrypt.HashPassword(user.Password);

            using (var conn = Database.GetConnection())
            {
                string query = "INSERT INTO users (username, password) VALUES (@username, @password)";
                using (var cmd = new MySqlCommand(query, conn))
                {
                    cmd.Parameters.AddWithValue("@username", user.Username);
                    cmd.Parameters.AddWithValue("@password", hashedPassword);

                    try
                    {
                        cmd.ExecuteNonQuery();
                        return true;
                    }
                    catch
                    {
                        return false; // Usuário já existe
                    }
                }
            }
        }

        public bool ValidateUser(string username, string password)
        {
            using (var conn = Database.GetConnection())
            {
                string query = "SELECT password FROM users WHERE username=@username";
                using (var cmd = new MySqlCommand(query, conn))
                {
                    cmd.Parameters.AddWithValue("@username", username);
                    var result = cmd.ExecuteScalar();

                    if (result != null)
                    {
                        string hashedPassword = result.ToString();
                        return BCrypt.Net.BCrypt.Verify(password, hashedPassword);
                    }
                    else
                    {
                        return false; // Usuário não encontrado
                    }
                }
            }
        }
    }
}
```

---

## **7️⃣ Criar Controller: UserController**

```csharp
// Controllers/UserController.cs
using LoginApp.Models;
using LoginApp.Services;

namespace LoginApp.Controllers
{
    public class UserController
    {
        private UserService _userService;

        public UserController()
        {
            _userService = new UserService();
        }

        public bool Register(string username, string password)
        {
            User user = new User { Username = username, Password = password };
            return _userService.RegisterUser(user);
        }

        public bool Login(string username, string password)
        {
            return _userService.ValidateUser(username, password);
        }
    }
}
```

---

## **8️⃣ Criar Formulário Login (View)**

No `FormLogin.cs`, adicione:

* `TextBox txtUsername`
* `TextBox txtPassword` (PasswordChar = '*')
* `Button btnLogin`
* `Button btnRegister` (opcional)

```csharp
// Views/FormLogin.cs
using System;
using System.Windows.Forms;
using LoginApp.Controllers;

namespace LoginApp.Views
{
    public partial class FormLogin : Form
    {
        private UserController _controller;

        public FormLogin()
        {
            InitializeComponent();
            _controller = new UserController();
        }

        private void btnLogin_Click(object sender, EventArgs e)
        {
            string username = txtUsername.Text.Trim();
            string password = txtPassword.Text;

            if (_controller.Login(username, password))
            {
                FormMain mainForm = new FormMain(username);
                mainForm.Show();
                this.Hide();
            }
            else
            {
                MessageBox.Show("Usuário ou senha incorretos!");
            }
        }

        private void btnRegister_Click(object sender, EventArgs e)
        {
            string username = txtUsername.Text.Trim();
            string password = txtPassword.Text;

            if (_controller.Register(username, password))
            {
                MessageBox.Show("Usuário registrado com sucesso!");
            }
            else
            {
                MessageBox.Show("Falha ao registrar usuário (já existe?)");
            }
        }
    }
}
```

---

## **9️⃣ Criar Formulário Main (View) com Logout**

No `FormMain.cs`:

* Label `lblWelcome`
* Button `btnLogout`

```csharp
// Views/FormMain.cs
using System;
using System.Windows.Forms;

namespace LoginApp.Views
{
    public partial class FormMain : Form
    {
        private string _username;

        public FormMain(string username)
        {
            InitializeComponent();
            _username = username;
            lblWelcome.Text = $"Bem-vindo, {_username}!";
        }

        private void btnLogout_Click(object sender, EventArgs e)
        {
            FormLogin loginForm = new FormLogin();
            loginForm.Show();
            this.Close();
        }
    }
}
```

---

## **🔟 Criar Banco de Dados e Tabela**

```sql
CREATE DATABASE login_system;

USE login_system;

CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL
);
```

> [!TIP]
> Lembre-se de substituir a senha no `Database.cs`.

---

## **11️⃣ Executar**

1. Compile e rode o projeto.
2. Teste o registro de usuário.
3. Teste login/logout.
4. Senhas ficam criptografadas com **bcrypt** no MySQL.

---
