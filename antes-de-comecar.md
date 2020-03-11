## 1. Crie um repositório na sua conta do GitHub chamado ``tdd-project``

Se necessário, leia mais informações [aqui](https://help.github.com/pt/github/getting-started-with-github/create-a-repo).


## 2. Abra o terminal Git Bash
Sempre utilize o terminal **Git Bash** para realizar as atividades das aulas práticas.

OBS: todas as máquinas devem ter instalado o Git e o Git Bash, mas caso a sua máquina não tenha,
baixe e instale no Windows (https://git-scm.com/download/win).

## 3. Crie uma pasta de trabalho na sua máquina chamada ``tdd-project``

```ShellSession
$ mkdir tdd-project
$ cd tdd-project
```

## 4. Configure seu repositório GitHub

Dentro da pasta `tdd-project`:

```ShellSession
$ echo "# tdd-project" >> README.md
$ git init
$ git add README.md
$ git commit -m "first commit"
$ git remote add origin git@github.com:SUA_CONTA/tdd-project.git
$ git push -u origin master
```

## 5. Crie um ambiente virtual Python (virtualenv)

Dentro da sua pasta de trabalho `tdd-project`, crie um ambiente virtual Python 3.7 (virtualenv):

```
$ py -3.7 -m venv virtualenv
```

## 6. Ative seu ambiente virtual

Dentro da sua pasta de trabalho `tdd-project`:

```ShellSession
$ source virtualenv/Scripts/activate
```

Note que a palavra **virtualenv** vai aparecer no seu terminal.

**IMPORTANTE:** O ambiente virtual deve estar ativado para realizar as atividades.

**OBS:** para desativar o ambiente vistual basta digital: `deactivate`.


## 7. Instale Django e Selenium

Com o ambiente virtual **ativado**, instale o Django 1.11 e o Selenium:

```
(virtualenv)
$ pip install "django<1.12" "selenium<4"
```

## 8. Baixe o Driver Geckodriver

O Geckodriver permite controlar o Firefox remotamente através do Selenium:

Baixe o driver, descompacte e coloque dentro da sua pasta de trabalho `tdd-project`:
- https://github.com/mozilla/geckodriver/releases/download/v0.26.0/geckodriver-v0.26.0-win64.zip

## 9. Ao concluir a aula prática, submeta via Moodle a dupla e o repositório GitHub utilizado


