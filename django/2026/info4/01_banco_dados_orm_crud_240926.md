# Material de Prof Bruno Gomes

# Assunto: Banco de Dados e Início do CRUD com Django

Nesta aula vamos criar um projeto Django do zero, configurar o banco de dados a partir de *models*, e dar o primeiro passo do CRUD: a **listagem** (o "R", de *Read*) de registros. Ao final, o projeto será enviado para o GitHub.

O sistema de exemplo, chamado **FIC**, cadastra cursos de Formação Inicial e Continuada, cada um com uma área, um público-alvo e outras informações.

> **CRUD** é a sigla para as quatro operações básicas sobre dados: **C**reate (cadastrar), **R**ead (listar/consultar), **U**pdate (editar) e **D**elete (excluir).

---

## 1. Preparando o ambiente

Abra o **Prompt de Comando** ou o **PowerShell** do Windows.

> Nos exemplos abaixo aparece a pasta `C:\Users\111111`. O valor `111111` corresponde a **sua matrícula** (o nome do seu usuário no computador).

### 1.1 Criando a pasta do projeto

```bat
C:\Users\111111> cd info4
C:\Users\111111\info4> mkdir fic
C:\Users\111111\info4> cd fic
```

- `cd` (*change directory*) entra em uma pasta.
- `mkdir` (*make directory*) cria uma pasta nova.

### 1.2 Criando e ativando o ambiente virtual

```bat
C:\Users\111111\info4\fic> python -m venv venv
C:\Users\111111\info4\fic> venv\Scripts\activate
(venv) C:\Users\111111\info4\fic>
```

O **ambiente virtual** (`venv`) é uma instalação isolada do Python só para este projeto. Assim, as bibliotecas que instalarmos aqui (como o Django) não interferem em outros projetos do computador, e cada projeto pode usar versões diferentes das mesmas bibliotecas.

Quando o ambiente está ativo, o prompt passa a começar com **`(venv)`**. Sempre confira isso antes de instalar pacotes ou rodar o projeto.

> **Problema comum no PowerShell:** se aparecer um erro dizendo que a execução de scripts está desabilitada, execute uma vez o comando abaixo e tente ativar de novo:
>
> ```powershell
> Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
> ```

### 1.3 Instalando o Django

```bat
(venv) C:\Users\111111\info4\fic> pip install django
```

O `pip` é o gerenciador de pacotes do Python. Ele baixa e instala o Django dentro do ambiente virtual.

### 1.4 Criando o projeto

```bat
(venv) C:\Users\111111\info4\fic> django-admin startproject fic .
```

> ⚠️ **Não esqueça o ponto (`.`) no final!**
>
> O ponto significa "a pasta atual". Ele diz ao Django para criar o projeto **aqui mesmo**, dentro da pasta `fic` onde já estamos.
>
> Sem o ponto, o Django criaria **mais uma** pasta `fic` dentro da atual, gerando uma estrutura duplicada e confusa (`fic\fic\fic\settings.py`), com o `manage.py` um nível abaixo de onde está o `venv`.

Estrutura correta, **com** o ponto:

```
fic/
├── venv/
├── manage.py
└── fic/
    ├── __init__.py
    ├── settings.py
    ├── urls.py
    ├── asgi.py
    └── wsgi.py
```

Estrutura **sem** o ponto (evitar):

```
fic/
├── venv/
└── fic/
    ├── manage.py
    └── fic/
        ├── settings.py
        └── ...
```

### 1.5 Criando o app `core`

```bat
(venv) C:\Users\111111\info4\fic> python manage.py startapp core
```

No Django existe uma diferença importante:

- **Projeto** (`fic`): a configuração geral do sistema (configurações, URLs principais).
- **App** (`core`): um módulo com uma funcionalidade. É nele que ficam os *models*, as *views* e os *templates*. Um projeto pode ter vários apps.

### 1.6 Abrindo o projeto no VS Code

```bat
(venv) C:\Users\111111\info4\fic> code .
```

Novamente o `.` significa "a pasta atual": o VS Code abre a pasta inteira do projeto.

### 1.7 Rodando o servidor

```bat
(venv) C:\Users\111111\info4\fic> python manage.py runserver
```

Abra o navegador em: **http://127.0.0.1:8000/**

Se aparecer a página do Django com um foguete, o projeto está funcionando. Para parar o servidor, pressione `Ctrl + C` no terminal.

> **Dica:** você pode usar o terminal integrado do VS Code (`Ctrl + '`). Lembre-se de ativar o `venv` nele também.

---

## 2. Configurações iniciais (`fic/settings.py`)

Abra `fic/settings.py` e faça três alterações:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'core',  # nosso app
]

LANGUAGE_CODE = 'pt-br'

TIME_ZONE = 'America/Recife'
```

- **`'core'` em `INSTALLED_APPS`**: registra o app no projeto. Sem isso, o Django não encontra os models nem os templates do `core`.
- **`LANGUAGE_CODE = 'pt-br'`**: coloca as mensagens do Django (incluindo o painel admin) em português.
- **`TIME_ZONE = 'America/Recife'`**: define o fuso horário usado pelo sistema.

---

## 3. Primeira página: template, view e URL

No Django, para exibir uma página precisamos de três peças:

| Peça | Arquivo | Papel |
|---|---|---|
| **Template** | `core/templates/*.html` | O HTML que será exibido |
| **View** | `core/views.py` | A função que processa a requisição e escolhe o template |
| **URL** | `fic/urls.py` | Liga um endereço (ex.: `/areas/`) a uma view |

O caminho de uma requisição é: **navegador → URL → view → template → navegador**.

### 3.1 Template

Dentro da pasta `core`, crie a pasta **`templates`**. Dentro dela, crie o arquivo **`index.html`** com um texto inicial:

```html
<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <title>FIC</title>
</head>
<body>
    <h1>Sistema de Cursos FIC</h1>
    <p>Bem-vindo!</p>
</body>
</html>
```

> O nome da pasta precisa ser exatamente `templates`: o Django procura automaticamente os templates nessa pasta dentro de cada app registrado em `INSTALLED_APPS`.

### 3.2 View

Em `core/views.py`:

```python
from django.shortcuts import render

def inicial(request):
    return render(request, 'index.html')
```

- `request` representa a requisição feita pelo navegador.
- `render` junta a requisição com o template e devolve a página HTML pronta.

### 3.3 URL

Em `fic/urls.py`:

```python
from django.contrib import admin
from django.urls import path
from core.views import *

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', inicial, name='inicial'),
]
```

- `''` (texto vazio) é a página raiz: `http://127.0.0.1:8000/`.
- `inicial` é a view que será chamada.
- `name='inicial'` dá um **nome** à rota. Com ele, podemos criar links nos templates sem escrever o endereço à mão (veremos adiante).

Acesse `http://127.0.0.1:8000/` e veja o texto do `index.html`.

---

## 4. Banco de dados: criando os Models

Um **model** é uma classe Python que representa uma **tabela** do banco de dados. Cada atributo da classe vira uma **coluna**. O Django usa um **ORM** (*Object-Relational Mapping*): escrevemos Python e ele gera o SQL por nós.

Por padrão, o Django usa o **SQLite**, um banco de dados guardado em um único arquivo (`db.sqlite3`) na pasta do projeto. Não é preciso instalar nada.

Em `core/models.py`:

```python
from django.db import models

class Area(models.Model):
    nome = models.CharField('Nome', max_length=100)

class Publico(models.Model):
    nome = models.CharField('Nome', max_length=100)

class Curso(models.Model):
    titulo = models.CharField('Título', max_length=200)
    descricao = models.TextField('Descrição')
    vagas = models.IntegerField('Vagas')
    carga_horaria = models.IntegerField('Carga Horária')
    data = models.DateField('Data de Início')
    area = models.ForeignKey(Area, on_delete=models.PROTECT)
    publico = models.ManyToManyField(Publico)
```

### 4.1 Tipos de campo usados

| Campo | Tipo no banco | Uso |
|---|---|---|
| `CharField` | Texto curto | Nomes, títulos. Exige `max_length` |
| `TextField` | Texto longo | Descrições sem limite definido |
| `IntegerField` | Número inteiro | Vagas, carga horária |
| `DateField` | Data | Data de início |
| `ForeignKey` | Chave estrangeira | Relacionamento **muitos-para-um** |
| `ManyToManyField` | Tabela intermediária | Relacionamento **muitos-para-muitos** |

O primeiro texto de cada campo (ex.: `'Título'`) é o **rótulo** que aparece para o usuário em formulários e no admin.

> O Django cria automaticamente, em toda tabela, uma coluna **`id`**: um número inteiro que identifica cada registro de forma única (a *chave primária*).

### 4.2 Os relacionamentos

**`area = models.ForeignKey(Area, on_delete=models.PROTECT)`**

Cada curso pertence a **uma** área, e uma área pode ter **vários** cursos (muitos-para-um). No banco, a tabela de cursos ganha uma coluna `area_id` com o `id` da área.

O `on_delete` diz o que fazer se alguém tentar excluir uma área que tem cursos:

- `PROTECT` (usado aqui): **impede** a exclusão enquanto houver cursos ligados à área. Protege contra perda acidental de dados.
- `CASCADE`: exclui a área **e todos os cursos** dela.
- `SET_NULL`: exclui a área e deixa o campo vazio nos cursos (exige `null=True`).

**`publico = models.ManyToManyField(Publico)`**

Um curso pode ser destinado a **vários** públicos (ex.: estudantes, servidores, comunidade externa), e um público pode estar em **vários** cursos. Para isso, o Django cria sozinho uma terceira tabela que guarda os pares curso–público.

### 4.3 Migrations: levando os models para o banco

Com os models escritos, precisamos criar as tabelas no banco. Isso é feito em dois passos.

**Passo 1 – gerar as migrations:**

```bat
python manage.py makemigrations core
```

Se aparecer o texto abaixo, está tudo certo:

```
Migrations for 'core':
  core\migrations\0001_initial.py
    + Create model Area
    + Create model Publico
    + Create model Curso
```

O `makemigrations` **compara** os models com o estado anterior e gera um arquivo (`0001_initial.py`) descrevendo as mudanças. Ele ainda **não** altera o banco.

**Passo 2 – aplicar as migrations:**

```bat
python manage.py migrate
```

O `migrate` **executa** as migrations pendentes e cria de fato as tabelas no `db.sqlite3`. Na primeira vez, ele também cria as tabelas internas do Django (usuários, sessões, admin etc.).

> **Regra prática:** sempre que alterar um `models.py`, rode `makemigrations` e depois `migrate`.

---

## 5. Página de Áreas

### 5.1 Template inicial

Em `core/templates`, crie **`areas.html`**:

```html
<h1>Áreas</h1>
<p>Cadastrar</p>
<p>Lista de Áreas</p>

<p><a href="{% url 'inicial' %}">Voltar</a></p>
```

A tag **`{% url 'inicial' %}`** gera o endereço da rota chamada `inicial` (definida com `name='inicial'` no `urls.py`). A vantagem: se um dia o endereço mudar, basta alterar o `urls.py`, e todos os links continuam funcionando.

### 5.2 View

Em `core/views.py`:

```python
def areas(request):
    return render(request, 'areas.html')
```

### 5.3 URL

Em `fic/urls.py`, dentro de `urlpatterns`:

```python
path('areas/', areas, name='areas'),
```

### 5.4 Link na página inicial

Em `core/templates/index.html`:

```html
<p><a href="{% url 'areas' %}">Áreas</a></p>
```

Agora é possível navegar da página inicial para a de áreas e voltar.

---

## 6. Listando as áreas do banco (o "R" do CRUD)

### 6.1 Buscando os dados na view

Atualize `core/views.py`:

```python
from django.shortcuts import render
from .models import Area, Publico, Curso

def inicial(request):
    return render(request, 'index.html')

def areas(request):
    lista_areas = Area.objects.all()
    context = {
        'lista_areas': lista_areas
    }
    return render(request, 'areas.html', context)
```

- **`from .models import ...`**: importa os models do próprio app (o `.` significa "deste mesmo pacote").
- **`Area.objects.all()`**: consulta todos os registros da tabela de áreas. É o equivalente a `SELECT * FROM core_area;` em SQL.
- **`context`**: um dicionário com os dados enviados ao template. A **chave** (`'lista_areas'`) é o nome pelo qual o template acessa os dados.

### 6.2 Exibindo no template

Atualize `core/templates/areas.html`:

```html
<h1>Áreas</h1>
<p>Cadastrar</p>
<p>Lista de Áreas</p>

{% for area in lista_areas %}
<p>{{ area.id }} {{ area.nome }}</p>
{% empty %}
<p>Nenhuma área cadastrada</p>
{% endfor %}

<p><a href="{% url 'inicial' %}">Voltar</a></p>
```

A linguagem de templates do Django tem dois tipos de marcação:

- **`{% ... %}`** (tags): executam lógica, como laços e condições.
- **`{{ ... }}`** (variáveis): exibem um valor na página.

O bloco `{% for %}` percorre a lista de áreas e exibe o `id` e o `nome` de cada uma. O **`{% empty %}`** é executado quando a lista está vazia, e o `{% endfor %}` fecha o laço.

### 6.3 Testando

1. Rode o servidor (`python manage.py runserver`).
2. Abra `http://127.0.0.1:8000/`.
3. Clique no link **Áreas**.
4. Deve aparecer: **Nenhuma área cadastrada**.

Isso é esperado: a tabela existe, mas ainda não tem registros. O cadastro (o "C" do CRUD) será feito na próxima etapa, no lugar do texto "Cadastrar".

---

## 7. Enviando o projeto para o GitHub

### 7.1 Criando o repositório

No GitHub, crie um **repositório novo e vazio** (sem README, sem `.gitignore` e sem licença, para evitar conflito no primeiro envio).

### 7.2 Criando o `.gitignore`

Na raiz do projeto (mesma pasta do `manage.py`), crie o arquivo **`.gitignore`**. Ele lista o que **não** deve ir para o repositório:

```gitignore
# Ambiente virtual
venv/

# Arquivos gerados pelo Python
__pycache__/
*.pyc

# Banco de dados local
db.sqlite3

# Configurações do editor
.vscode/
```

- **`venv/`** é pesado e específico de cada computador. Quem baixar o projeto cria o seu próprio.
- **`__pycache__/`** contém arquivos que o Python gera sozinho.
- **`db.sqlite3`** contém os dados locais. Como as migrations estão no repositório, qualquer pessoa recria o banco com `python manage.py migrate`.

Também é boa prática registrar as dependências do projeto:

```bat
pip freeze > requirements.txt
```

Quem clonar o projeto instala tudo com `pip install -r requirements.txt`.

### 7.3 Enviando

No terminal, na pasta do projeto (troque o endereço pelo do seu repositório):

```bat
git init
git add .
git commit -m "First commit"
git branch -M main
git remote add origin https://github.com/SEU-USUARIO/fic.git
git push -u origin main
```

| Comando | O que faz |
|---|---|
| `git init` | Transforma a pasta em um repositório Git |
| `git add .` | Seleciona todos os arquivos (exceto os do `.gitignore`) |
| `git commit -m "..."` | Salva uma versão com uma mensagem descritiva |
| `git branch -M main` | Nomeia a branch principal como `main` |
| `git remote add origin ...` | Liga o repositório local ao do GitHub |
| `git push -u origin main` | Envia os arquivos para o GitHub |

> Se for o primeiro uso do Git no computador, configure seu nome e e-mail antes do commit:
>
> ```bat
> git config --global user.name "Seu Nome"
> git config --global user.email "seu@email.com"
> ```

---

## Resumo da aula

1. Criamos um **ambiente virtual** e instalamos o Django.
2. Criamos o **projeto** `fic` (com o `.` no final!) e o **app** `core`.
3. Configuramos idioma, fuso horário e registramos o app em `settings.py`.
4. Montamos a primeira página com **template + view + URL**.
5. Criamos os **models** `Area`, `Publico` e `Curso`, com relacionamentos `ForeignKey` e `ManyToManyField`.
6. Geramos e aplicamos as **migrations** para criar as tabelas.
7. Implementamos a **listagem** de áreas, enviando dados da view para o template via `context`.
8. Versionamos o projeto no **GitHub** com um `.gitignore` adequado.

**Próximo passo:** implementar o **cadastro** de áreas (o "C" do CRUD) com formulários.
