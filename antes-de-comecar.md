## Requisitos
- Python 3.6+ (https://www.python.org)
- Git (http://git-scm.com)
- Firefox (https://www.mozilla.org/firefox)


## 1. Crie um repositório na sua conta do GitHub chamado ``tdd-project``

Se necessário, leia mais informações [sobe como criar um repositório](https://help.github.com/pt/github/getting-started-with-github/create-a-repo).

Caso não tenha uma conta no GitHub, [faça o registro](https://help.github.com/pt/github/getting-started-with-github/signing-up-for-github).

## 2. Abra o terminal

Windows: Sempre utilize o terminal **Git Bash** para realizar as atividades das aulas práticas.

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
$ git remote add origin https://github.com/SUA_CONTA/tdd-project.git
$ git push -u origin master
```

## 5. Crie um ambiente virtual Python (virtualenv)

Dentro da sua pasta de trabalho `tdd-project`, crie um ambiente virtual Python 3.6+.
Nesse caso, estamos criando um ambiente com Python 3.7:

Windows:
```
$ py -3.7 -m venv virtualenv
```

Mac/Linux:
```
$ python3.7 -m venv virtualenv
```

## 6. Ative seu ambiente virtual

Dentro da sua pasta de trabalho `tdd-project`, ative o ambiente virtual Python:

Ativando no Windows:

```ShellSession
$ source virtualenv/Scripts/activate
```

Ativando no Mac/Linux:

```ShellSession
$ source virtualenv/bin/activate
```

Note que a palavra **virtualenv** vai aparecer no seu terminal, mostrando que o ambiente virtual está ativo.
Para desativar o ambiente virtual, basta digitar: `deactivate`.

### IMPORTANTE: O ambiente virtual deve estar sempre ativado para realizar as atividades.

## 7. Instale Django e Selenium (com ambiente virtual ativado)

Com o ambiente virtual **ativado**, instale o Django 1.11 e o Selenium:

```
(virtualenv)
$ pip install "django<1.12" "selenium<4"
```

## 8. Baixe o Driver Geckodriver

O Geckodriver permite controlar o Firefox remotamente através do Selenium.

- Download: https://github.com/mozilla/geckodriver/releases
- Windows: descompacte e coloque dentro da sua pasta de trabalho `tdd-project`
- Mac/Linux: descompacte e coloque na pasta `/usr/local/bin` (você vai precisar usar o `sudo`) 
