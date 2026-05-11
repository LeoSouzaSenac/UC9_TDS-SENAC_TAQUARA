# Criando uma Tela Simples, Bonita e Moderna no WinForms

# Objetivo

Neste tutorial você vai aprender:

- Como criar uma interface moderna
- Como organizar componentes
- Como deixar o visual bonito
- Como criar um layout profissional
- Como programar os botões
- Como trabalhar com eventos
- Como usar painéis
- Como melhorar o design do sistema

---

# Resultado Final

Você irá criar uma tela parecida com isto:

```text
+------------------------------------------------+
| LOGO              Sistema Login                |
|------------------------------------------------|
|                                                |
| Usuário                                        |
| [______________________________]               |
|                                                |
| Senha                                          |
| [______________________________]               |
|                                                |
| [ Entrar ]                                     |
|                                                |
| [ ] Lembrar senha                              |
|                                                |
+------------------------------------------------+
````

---

# PASSO 1 — Criando o Projeto

No Visual Studio:

```text
Arquivo -> Novo Projeto
```

Escolha:

```text
Windows Forms App (.NET)
```

Nome:

```text
SistemaLogin
```

Clique em:

```text
Criar
```

---

# PASSO 2 — Configurando o Formulário

Clique no formulário.

Agora altere estas propriedades:

| Propriedade     | Valor         |
| --------------- | ------------- |
| Name            | FrmLogin      |
| Text            | Sistema Login |
| StartPosition   | CenterScreen  |
| FormBorderStyle | FixedSingle   |
| MaximizeBox     | False         |
| BackColor       | White         |
| Size            | 800, 500      |

---

# PASSO 3 — Criando o Painel Lateral

Arraste um componente:

```text
Panel
```

---

# Configurações do painel

| Propriedade | Valor        |
| ----------- | ------------ |
| Name        | panelLateral |
| Dock        | Left         |
| Width       | 250          |
| BackColor   | 45,45,48     |

---

# O que é um Panel?

O Panel serve para:

* Organizar elementos
* Criar menus laterais
* Separar áreas
* Melhorar layout

Muito usado em sistemas modernos.

---

# PASSO 4 — Adicionando Logo/Título

Dentro do painel lateral adicione um:

```text
Label
```

---

# Configuração

| Propriedade | Valor              |
| ----------- | ------------------ |
| Text        | LOGIN APP          |
| ForeColor   | White              |
| Font        | Segoe UI, 20, Bold |
| AutoSize    | True               |

---

# Ajuste posição

```text
Location = 40, 50
```

---

# PASSO 5 — Criando Área Principal

Agora vamos criar:

* Label usuário
* TextBox usuário
* Label senha
* TextBox senha
* CheckBox
* Button

---

# PASSO 6 — Label Usuário

Adicione um Label.

| Propriedade | Valor        |
| ----------- | ------------ |
| Text        | Usuário      |
| Font        | Segoe UI, 10 |
| ForeColor   | Black        |
| Location    | 320, 120     |

---

# PASSO 7 — TextBox Usuário

Adicione um TextBox.

| Propriedade | Valor        |
| ----------- | ------------ |
| Name        | txtUsuario   |
| BorderStyle | FixedSingle  |
| Font        | Segoe UI, 12 |
| Size        | 250, 30      |
| Location    | 320, 150     |

---

# PASSO 8 — Label Senha

Adicione outro Label.

| Propriedade | Valor        |
| ----------- | ------------ |
| Text        | Senha        |
| Font        | Segoe UI, 10 |
| Location    | 320, 210     |

---

# PASSO 9 — TextBox Senha

Adicione outro TextBox.

| Propriedade           | Valor        |
| --------------------- | ------------ |
| Name                  | txtSenha     |
| Font                  | Segoe UI, 12 |
| Size                  | 250, 30      |
| Location              | 320, 240     |
| UseSystemPasswordChar | True         |

---

# O que faz UseSystemPasswordChar?

Oculta a senha digitada.

Exemplo:

```text
******
```

---

# PASSO 10 — CheckBox

Adicione um CheckBox.

| Propriedade | Valor         |
| ----------- | ------------- |
| Name        | chkLembrar    |
| Text        | Lembrar senha |
| Location    | 320, 290      |

---

# PASSO 11 — Botão Entrar

Adicione um Button.

| Propriedade | Valor              |
| ----------- | ------------------ |
| Name        | btnEntrar          |
| Text        | Entrar             |
| Size        | 250, 40            |
| Location    | 320, 340           |
| BackColor   | 0,120,215          |
| ForeColor   | White              |
| FlatStyle   | Flat               |
| Font        | Segoe UI, 10, Bold |

---

# O que é FlatStyle?

Deixa o botão moderno.

Sem aparência antiga do Windows.

---

# PASSO 12 — Melhorando o Botão

Abra o código do formulário.

No construtor:

```csharp id="4u4xy8"
public FrmLogin()
{
    InitializeComponent();

    btnEntrar.FlatAppearance.BorderSize = 0;
}
```

---

# Resultado

O botão ficará:

* moderno
* sem borda
* mais profissional

---

# PASSO 13 — Criando Evento do Botão

Dê duplo clique no botão.

O Visual Studio cria:

```csharp id="bn2r9n"
private void btnEntrar_Click(object sender, EventArgs e)
{

}
```

---

# PASSO 14 — Código de Login

Agora coloque:

```csharp id="7k5m7v"
private void btnEntrar_Click(object sender, EventArgs e)
{
    string usuario = txtUsuario.Text;
    string senha = txtSenha.Text;

    if(usuario == "admin" && senha == "123")
    {
        MessageBox.Show(
            "Login realizado com sucesso!",
            "Sucesso",
            MessageBoxButtons.OK,
            MessageBoxIcon.Information
        );
    }
    else
    {
        MessageBox.Show(
            "Usuário ou senha inválidos!",
            "Erro",
            MessageBoxButtons.OK,
            MessageBoxIcon.Error
        );
    }
}
```

---

# Explicando o Código

---

# Pegando texto digitado

```csharp id="x5xy7q"
txtUsuario.Text
```

Retorna o conteúdo digitado.

---

# IF

```csharp id="6g6aq7"
if(usuario == "admin")
```

Verifica condição.

---

# MessageBox

Mostra uma janela.

---

# MessageBoxIcon

Pode ser:

| Tipo        | Ícone      |
| ----------- | ---------- |
| Information | Informação |
| Error       | Erro       |
| Warning     | Aviso      |

---

# PASSO 15 — Criando Botão Limpar

Adicione outro botão.

| Propriedade | Valor     |
| ----------- | --------- |
| Name        | btnLimpar |
| Text        | Limpar    |
| Size        | 250, 40   |
| Location    | 320, 390  |
| FlatStyle   | Flat      |

---

# Evento do botão limpar

```csharp id="fcgk4f"
private void btnLimpar_Click(object sender, EventArgs e)
{
    txtUsuario.Clear();
    txtSenha.Clear();

    txtUsuario.Focus();
}
```

---

# O que faz Focus()?

Move o cursor automaticamente.

---

# PASSO 16 — Adicionando PictureBox

Adicione:

```text
PictureBox
```

Dentro do painel lateral.

---

# Configuração

| Propriedade | Valor        |
| ----------- | ------------ |
| Size        | 100,100      |
| SizeMode    | StretchImage |
| Location    | 70,120       |

---

# Inserindo imagem

Na propriedade:

```text
Image
```

Escolha:

```text
Importar
```

---

# PASSO 17 — Criando Rodapé

Adicione Label.

| Propriedade | Valor                |
| ----------- | -------------------- |
| Text        | © 2025 Sistema Login |
| ForeColor   | Gray                 |
| Location    | 320, 450             |

---

# PASSO 18 — Melhorando Aparência

Use estas cores:

---

# Fundo escuro

```text
45,45,48
```

---

# Azul moderno

```text
0,120,215
```

---

# Fonte recomendada

```text
Segoe UI
```

---

# PASSO 19 — Organização Profissional

Use:

* Panels
* Margens
* Espaçamento
* Alinhamento

---

# ERROS COMUNS

---

# 1. Componentes desalinhados

Use:

```text
Formatar -> Alinhar
```

---

# 2. Nome errado

Evite:

```text
button1
textBox2
```

Prefira:

```text
btnEntrar
txtSenha
```

---

# 3. Código gigante

Crie métodos.

---

# Exemplo

```csharp id="t2wnhh"
private void LimparCampos()
{
    txtUsuario.Clear();
    txtSenha.Clear();
}
```

---

# PASSO 20 — Executando Projeto

Pressione:

```text
F5
```

ou clique:

```text
▶ Iniciar
```

