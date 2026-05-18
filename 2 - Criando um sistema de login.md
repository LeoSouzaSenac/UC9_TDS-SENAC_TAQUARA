# Tutorial Simples: Login/Logout em C# Windows Forms (.NET) com MySQL e BCrypt

## Objetivo

Neste projeto vamos aprender:

* Como criar um sistema de login
* Como conectar C# com MySQL
* Como cadastrar usuários
* Como funciona criptografia de senha
* Como usar BCrypt
* Como funciona comunicação com banco de dados

Tudo explicado de forma simples e comentada.

---

# 1. Criando o Projeto

1. Abra o Visual Studio
2. Clique em:

```text id="qf4vud"
Create a new project
```

3. Escolha:

```text id="2f2ovl"
Windows Forms App (.NET)
```

4. Nome do projeto:

```text id="55xb1g"
LoginApp
```

5. Clique em Create

---

# 2. Instalando Pacotes NuGet

## O que é NuGet?

NuGet é o sistema de bibliotecas do C#.

Ele permite instalar funcionalidades prontas.

---

## Instale:

* `MySql.Data`
* `BCrypt.Net-Next`

---

## Como instalar

Clique com botão direito no projeto:

```text id="v5z4ej"
Manage NuGet Packages
```

Depois procure e instale os pacotes.

---

# 3. Criando o Banco de Dados

Execute esse SQL no MySQL:

```sql id="vb81h9"
CREATE DATABASE login_system;

USE login_system;

CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL
);
```

---

# Explicando o Banco

## `CREATE DATABASE`

Cria o banco.

---

## `CREATE TABLE`

Cria uma tabela.

---

## `AUTO_INCREMENT`

O número aumenta sozinho.

---

## `PRIMARY KEY`

Identificador único.

---

## `VARCHAR(50)`

Texto de até 50 caracteres.

---

## `UNIQUE`

Impede usuários repetidos.

---

# 4. Criando Classe de Conexão

Crie um arquivo chamado:

```text id="m8ev08"
Database.cs
```

---

# Código Completo do Database.cs

```csharp id="w7mg0h"
using MySql.Data.MySqlClient;
using System;

namespace LoginApp
{
    // Classe responsável pela conexão com o banco
    // "static" significa que podemos usar ela sem criar objeto
    public static class Database
    {
        // String de conexão
        // Aqui informamos:
        // servidor, banco, usuário e senha
        private static string connectionString =
            "server=localhost;database=login_system;uid=root;pwd=sua_senha;";

        // Método responsável por abrir conexão
        public static MySqlConnection GetConnection()
        {
            // Cria objeto de conexão
            MySqlConnection conn = new MySqlConnection(connectionString);

            try
            {
                // Tenta abrir conexão com o banco
                conn.Open();

                // Retorna conexão aberta
                return conn;
            }
            catch (Exception ex)
            {
                // Caso dê erro, mostra mensagem
                throw new Exception("Erro ao conectar: " + ex.Message);
            }
        }
    }
}
```

---

# Explicando as partes importantes

## `using`

```csharp id="0d1jdf"
using MySql.Data.MySqlClient;
```

Importa ferramentas do MySQL.

---

## `namespace`

```csharp id="b5fqyl"
namespace LoginApp
```

Organiza o projeto.

É como uma pasta lógica.

---

## `public static class`

```csharp id="84i8b8"
public static class Database
```

Criamos uma classe chamada Database.

---

## O que é classe?

Classe é um molde.

Exemplo:

* Classe Carro
* Classe Pessoa
* Classe Database

---

## O que é objeto?

Objeto é algo criado a partir da classe.

Exemplo:

```csharp id="6dd8b4"
MySqlConnection conn
```

`conn` é um objeto.

---

## String de conexão

```csharp id="6nvf6z"
"server=localhost;database=login_system;uid=root;pwd=sua_senha;"
```

Explicando:

| Parte                 | Significado          |
| --------------------- | -------------------- |
| server=localhost      | banco está no seu PC |
| database=login_system | nome do banco        |
| uid=root              | usuário do MySQL     |
| pwd=sua_senha         | senha do MySQL       |

---

# 5. Criando o FormLogin

No formulário adicione:

## TextBox

* `txtUsername`
* `txtPassword`

---

## PasswordChar

No `txtPassword` coloque:

```text id="ozsyl2"
PasswordChar = *
```

Isso esconde a senha.

---

## Buttons

* `btnLogin`
* `btnRegister`

---

# Código Completo do FormLogin

```csharp id="8f4z9u"
using System;
using System.Windows.Forms;
using MySql.Data.MySqlClient;
using BCrypt.Net;

namespace LoginApp
{
    public partial class FormLogin : Form
    {
        // Construtor da tela
        // Executa quando o formulário abre
        public FormLogin()
        {
            InitializeComponent();
        }

        // =====================================================
        // BOTÃO LOGIN
        // =====================================================
        private void btnLogin_Click(object sender, EventArgs e)
        {
            // Pega texto digitado nas caixas
            // Trim() remove espaços extras
            string username = txtUsername.Text.Trim();
            string password = txtPassword.Text;

            // "using" fecha conexão automaticamente
            // mesmo se ocorrer erro
            using (var conn = Database.GetConnection())
            {
                // SQL que busca senha do usuário
                string sql =
                    "SELECT password FROM users WHERE username=@user";

                // Objeto responsável por executar SQL
                using (var cmd = new MySqlCommand(sql, conn))
                {
                    // Passa valor para parâmetro @user
                    // Isso evita SQL Injection
                    cmd.Parameters.AddWithValue("@user", username);

                    // ExecuteScalar retorna apenas UM valor
                    // Nesse caso: a senha(hash)
                    var result = cmd.ExecuteScalar();

                    // Se encontrou usuário
                    if (result != null)
                    {
                        // Converte resultado para string
                        string hash = result.ToString();

                        // BCrypt compara senha digitada
                        // com hash salvo no banco
                        bool senhaCorreta =
                            BCrypt.Net.BCrypt.Verify(password, hash);

                        // Se senha estiver correta
                        if (senhaCorreta)
                        {
                            MessageBox.Show("Login realizado!");

                            // Abre tela principal
                            FormMain main = new FormMain(username);

                            main.Show();

                            // Esconde tela atual
                            this.Hide();
                        }
                        else
                        {
                            MessageBox.Show("Senha incorreta!");
                        }
                    }
                    else
                    {
                        MessageBox.Show("Usuário não encontrado!");
                    }
                }
            }
        }

        // =====================================================
        // BOTÃO CADASTRO
        // =====================================================
        private void btnRegister_Click(object sender, EventArgs e)
        {
            // Pega valores digitados
            string username = txtUsername.Text.Trim();
            string password = txtPassword.Text;

            // Cria hash da senha
            // Nunca salvamos senha pura
            string hash =
                BCrypt.Net.BCrypt.HashPassword(password);

            // Abre conexão
            using (var conn = Database.GetConnection())
            {
                // SQL de INSERT
                string sql =
                    "INSERT INTO users (username, password) VALUES (@user, @pass)";

                using (var cmd = new MySqlCommand(sql, conn))
                {
                    // Passa valores para parâmetros
                    cmd.Parameters.AddWithValue("@user", username);
                    cmd.Parameters.AddWithValue("@pass", hash);

                    try
                    {
                        // Executa INSERT
                        cmd.ExecuteNonQuery();

                        MessageBox.Show("Usuário criado com sucesso!");
                    }
                    catch
                    {
                        // Caso usuário já exista
                        MessageBox.Show("Erro: usuário já existe!");
                    }
                }
            }
        }
    }
}
```

---

# Explicando Conceitos do FormLogin

# Evento Click

```csharp id="3j9nzt"
private void btnLogin_Click(object sender, EventArgs e)
```

Esse método executa quando clicamos no botão.

---

# `private`

Só a própria classe pode usar.

---

# `void`

O método não retorna valor.

---

# `.Text`

```csharp id="55m4mx"
txtUsername.Text
```

Pega o texto digitado.

---

# `.Trim()`

Remove espaços extras.

Exemplo:

```text id="ykhqkh"
" joao "
```

vira:

```text id="vzh7f6"
"joao"
```

---

# `using`

```csharp id="6p0pwm"
using (var conn = Database.GetConnection())
```

Libera memória automaticamente.

Muito usado com:

* banco
* arquivos
* conexões

---

# SQL Injection

Usamos parâmetros:

```csharp id="8i7p8w"
@user
```

para evitar ataques.

---

# ExecuteScalar

```csharp id="tdr1yj"
cmd.ExecuteScalar();
```

Retorna apenas um valor.

---

# BCrypt

## Hash da senha

```csharp id="q3gl31"
BCrypt.Net.BCrypt.HashPassword(password);
```

Transforma senha em hash.

---

## Verificação

```csharp id="q3hxv2"
BCrypt.Net.BCrypt.Verify(password, hash);
```

Compara senha digitada com hash.

---

# Por que usar hash?

Nunca devemos salvar senha real.

ERRADO:

```text id="u1j5f9"
123456
```

CERTO:

```text id="5oqr3z"
$2a$11$asd89as7d...
```

---

# 6. Criando FormMain

Adicione:

* Label → `lblWelcome`
* Button → `btnLogout`

---

# Código Completo do FormMain

```csharp id="2vhq4k"
using System;
using System.Windows.Forms;

namespace LoginApp
{
    public partial class FormMain : Form
    {
        // Construtor
        // Recebe nome do usuário
        public FormMain(string username)
        {
            InitializeComponent();

            // Mostra usuário logado
            lblWelcome.Text =
                "Bem-vindo, " + username;
        }

        // =====================================================
        // LOGOUT
        // =====================================================
        private void btnLogout_Click(object sender, EventArgs e)
        {
            // Cria tela de login
            FormLogin login = new FormLogin();

            // Mostra tela
            login.Show();

            // Fecha tela atual
            this.Close();
        }
    }
}
```

---

# Fluxo do Sistema

# Cadastro

```text id="mpbf3g"
Usuário digita senha
↓
BCrypt cria hash
↓
Hash vai para banco
```

---

# Login

```text id="p1l2o5"
Usuário digita senha
↓
Sistema busca hash
↓
BCrypt compara
↓
Acesso liberado
```


