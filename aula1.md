# Teste de Software

## Aula Prática 1: Introdução a TDD, Testes Funcionais e Testes de Unidade

Prof. André Hora

Objetivo: Reproduzir uma sessão de desenvolvimento de software usando TDD (Test-Driven Development), com base no livro *Test-Driven-Development with Python*.
Especificamente, iremos reproduzir os exemplos do Cap. 1 até o Cap. 3 do livro.
**Faça o exercício com o mindset de TDD.**
Se necessário, estude antes o material visto em sala de aula.

Aplicação a ser desenvolvida: Site para listar tarefas a fazer (TODO list).
Mais informações ao longo do roteiro.

Instruções:

- Siga o roteiro, passo a passo.
- Após cada passo, rode os testes.
- Nos passos marcados com **COMMIT & PUSH**, faça um commit e um push no seu repositório.
Isso será usado no momento da correção, para garantir que seguiu a sequência do roteiro, passo-a-passo.

## 1. Iniciando com TDD e Testes Funcionais

No seu diretório de trabalho `tdd-project`, crie um arquivo chamado **functional_tests.py**, com o seguinte código:

```Python
from selenium import webdriver

browser = webdriver.Firefox()
browser.get('http://localhost:8000')

assert 'Django' in browser.title
```
Esse é nosso primeiro teste funcional. Esse código:

1. Inicia o driver Selenium para abrir um janela do Firefox
2. Abre uma página local (http://localhost:8000)
3. Verifica se a página tem a palavra Django no seu título

Tente rodar o teste:

```ShellSession
$ python functional_tests.py
Traceback (most recent call last):
  File "functional_tests.py", line 4, in <module>
    browser.get('http://localhost:8000')
  ...
```

Conforme esperado, o teste vai falhar (estado vermelho do ciclo TDD).

#### COMMIT & PUSH com a mensagem: Iniciando com Testes Funcionais

```ShellSession
$ git add functional_tests.py
$ git commit -m "Iniciando com Testes Funcionais"
$ git push -u origin master
```

## 2. Criando projeto Django e subindo o servidor

O próximo passo é criar um projeto Django na pasta do projeto:

```ShellSession
$ django-admin.py startproject superlists .
```

Isso irá criar um arquivo **manage.py** na pasta do projeto e uma subpasta **superlists**, conforme a seguir:

```
├── functional_tests.py
├── geckodriver.log
├── manage.py
├── superlists
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
└── virtualenv
    ├── [...]
```

Nosso diretório de trabalho é sempre a pasta que contém **manage.py**, um importante arquivo para o framework Django. Uma das funcionalidades do arquivo **manage.py** é subir o servidor, conforme segue:

```ShellSession
$ python manage.py runserver
Performing system checks...

System check identified no issues (0 silenced).
...
```

Nosso servidor de desenvolvimento está no ar. Tente entrar em: http://localhost:8000

Abra outro terminal (**e lembre de ativar novamente o ambiente virtual do Python**), navegue para a pasta do projeto, ative o virtualenv e rode o teste novamente:

```ShellSession
$ python functional_tests.py
$
```

Nada acontece no terminal, mas note dois pontos: 

1. O teste passou (sem mensagem de erro) 
2. A janela do Firefox aberta mostrou o servidor no ar (com a mensagem: **It worked! Congratulations on your first Django-powered page**)

Antes de fazermos o commit, note que não queremos versionar alguns arquivos e pastas: db.sqlite3 (banco de dados), geckodriver.log, virtualenv, \_\_pycache\_\_ e \*.pyc. Então, adicionamos eles em **.gitignore** para o git ignorar tais arquivos:

```ShellSession
$ echo "db.sqlite3" >> .gitignore
$ echo "geckodriver.log" >> .gitignore
$ echo "virtualenv" >> .gitignore
$ echo "__pycache__" >> .gitignore
$ echo "*.pyc" >> .gitignore
```

#### COMMIT & PUSH com a mensagem: Criando projeto Django e subindo o servidor

Lembre de adicionar o arquivo .gitignore:

```ShellSession
$ git add .gitignore
$ git commit -m "Criando projeto Django e subindo o servidor"
$ git push -u origin master
```

Parabéns! Você implementou (e versionou) o seu primeiro teste funcional!

## 3. Utilizando Testes Funcionais para planejar um MVP

Vamos alterar e expandir nosso teste funcional para que funcione de forma mais realista.
Logo, esse teste deve verificar algo real sobre a aplicação a ser desenvolvida.
A aplicação desenvolvida ao longo das aulas práticas será um **site para listar tarefas a fazer** (TODO list).

Uma aplicação TODO list é bem simples (apenas uma listagem de tarefas), então é fácil criarmos um [produto viável mínimo](https://pt.wikipedia.org/wiki/Produto_vi%C3%A1vel_m%C3%ADnimo) (MVP, de Minimum Viable Product). Além disso, essa aplicação pode ser estendida de diversas formas: diferentes modelos de persistência, deadlines, lembretes, compartilhamento, melhoria de UI, etc. Mas o ponto principal é que ela permite apresentar diversos aspectos de testes de software de forma aplicada.

Testes que usam Selenium (ou seja, os Testes Funcionais) permitem que aplicações sejam testadas do ponto de vista do usuário. Em outras palavras, os Testes Funcionais podem ser vistos como uma especificação do sistema. Podemos mapear esse teste para uma [História do Usuário](https://pt.wikipedia.org/wiki/Hist%C3%B3ria_de_usu%C3%A1rio) (User Story), ou seja, como o usuário vai utilizar uma determinada funcionalidade e como o sistema deve responder.

Lembrando: Testes Funcionais == Testes de Aceitação == Testes End-to-End

Testes Funcionais (assim como Histórias do Usuário) devem contar uma história legível por humanos. Podemos fazer isso explícito através de comentários no próprio Teste Funcional.

Abra **functional_tests.py** e escreva a seguinte história:

```Python
from selenium import webdriver

browser = webdriver.Firefox()

# Maria decidiu utilizar o novo app TODO. Ela entra em sua página principal:
browser.get('http://localhost:8000')

# Ela nota que o título da página menciona TODO
assert 'To-Do' in browser.title

# Ela é convidada a entrar com um item TODO imediatamente

# Ela digita "Estudar testes funcionais" em uma caixa de texto

# Quando ela aperta enter, a página atualiza, e mostra a lista
# "1: Estudar testes funcionais" como um item da lista TODO

# Ainda existe uma caixa de texto convidando para adicionar outro item
# Ela digita: "Estudar testes de unidade"

# A página atualiza novamente, e agora mostra ambos os itens na sua lista

# Maria se pergunta se o site vai lembrar da sua lista. Então, ela verifica que
# o site gerou uma URL única para ela -- existe uma explicação sobre essa feature

# Ela visita a URL: a sua lista TODO ainda está armazenada

# Satisfeita, ela vai dormir

browser.quit()
```

Note que mudamos o assert de 'Django' para 'TODO', logo esse teste deve falhar.
Rode o teste funcional e verifique o resultado:

```ShellSession
$ python functional_tests.py
Traceback (most recent call last):
  File "functional_tests.py", line 9, in <module>
    assert 'to-do' in browser.title
AssertionError
```

Essa falha é esperada (expected fail), mas trata-se de uma pequena vitória: desenvolvedor possui uma especificação do que ele precisará implementar.

#### COMMIT & PUSH com a mensagem: Utilizando Testes Funcionais para planejar um MVP

## 4. Utilizando o módulo unittest

Para melhorar nosso teste funcional, vamos utilizar o módulo **unittest** da biblioteca padrão de Python chamado.
Atualize **functional_tests.py** para:

```Python
from selenium import webdriver
import unittest

class NewVisitorTest(unittest.TestCase):

    def test_can_start_a_list_and_retrieve_it_later(self): 
    
        self.browser = webdriver.Firefox()
    
        # Maria decidiu utilizar o novo app TODO. Ela entra em sua página principal:
        self.browser.get('http://localhost:8000')

        # Ela nota que o título da página menciona TODO
        self.assertIn('To-Do', self.browser.title)

        # Ela é convidada a entrar com um item TODO imediatamente

        # Ela digita "Estudar testes funcionais" em uma caixa de texto

        # Quando ela aperta enter, a página atualiza, e mostra a lista
        # "1: Estudar testes funcionais" como um item da lista TODO
        
        # Ainda existe uma caixa de texto convidando para adicionar outro item
        # Ela digita: "Estudar testes de unidade"

        # A página atualiza novamente, e agora mostra ambos os itens na sua lista

        # Maria se pergunta se o site vai lembrar da sua lista. Então, ela verifica que
        # o site gerou uma URL única para ela -- existe uma explicação sobre essa feature

        # Ela visita a URL: a sua lista TODO ainda está armazenada

        # Satisfeita, ela vai dormir
        
        self.browser.quit()

if __name__ == '__main__':
    unittest.main()
```

Note alguns pontos importantes:

1. O teste está organizado em uma classe, `NewVisitorTest`, que herda de `unittest.TestCase`.
2. O corpo do teste se encontra no método `test_can_start_a_list_and_retrieve_it_later()`. Lembrando que todo método de teste começa com **test** e será rodado pela biblioteca de testes. Podemos ter um ou mais métodos de teste por classe. Nomes descritivos são uma boa prática.
3. Usamos `self.assertIn` ao invés de `assert`. O módulo **unittest** possui diversos métodos utilitários, tais como `assertEqual`, `assertTrue`, `assertFalse`, etc. Mais informações na sua [documentação](https://docs.python.org/3.6/library/unittest.html).
4. Por fim, rodamos classes e métodos de testes através de `unittest.main()`.

Rode o novo teste funcional e verifique o resultado:

```ShellSession
$ python functional_tests.py
F
======================================================================
FAIL: test_can_start_a_list_and_retrieve_it_later (__main__.NewVisitorTest)
 ---------------------------------------------------------------------
Traceback (most recent call last):
  File "functional_tests.py", line 18, in
test_can_start_a_list_and_retrieve_it_later
    self.assertIn('To-Do', self.browser.title)
AssertionError: 'To-Do' not found in 'Welcome to Django'

 ---------------------------------------------------------------------
Ran 1 test in 1.747s

FAILED (failures=1)
```

O resultado fornecido possui agora melhor formatação, detalhando: quantos testes rodaram, quantos falharam e o resultado de `assertIn`: 'To-Do' not found in 'Welcome to Django'.

#### COMMIT & PUSH com a mensagem: Utilizando o módulo unittest

## 5. Melhorando a classe de teste com setUp e tearDown

Modifique o teste **functional_tests.py** para incluir os métodos `setUp` e `tearDown`:

```Python
from selenium import webdriver
import unittest

class NewVisitorTest(unittest.TestCase):  

    def setUp(self):  
        self.browser = webdriver.Firefox()

    def tearDown(self):  
        self.browser.quit()

    def test_can_start_a_list_and_retrieve_it_later(self):  
        # Maria decidiu utilizar o novo app TODO. Ela entra em sua página principal:
        self.browser.get('http://localhost:8000')

        # Ela nota que o título da página menciona TODO
        self.assertIn('To-Do', self.browser.title)

        # Ela é convidada a entrar com um item TODO imediatamente

        # Ela digita "Estudar testes funcionais" em uma caixa de texto

        # Quando ela aperta enter, a página atualiza, e mostra a lista
        # "1: Estudar testes funcionais" como um item da lista TODO
        
        # Ainda existe uma caixa de texto convidando para adicionar outro item
        # Ela digita: "Estudar testes de unidade"

        # A página atualiza novamente, e agora mostra ambos os itens na sua lista

        # Maria se pergunta se o site vai lembrar da sua lista. Então, ela verifica que
        # o site gerou uma URL única para ela -- existe uma explicação sobre essa feature

        # Ela visita a URL: a sua lista TODO ainda está armazenada

        # Satisfeita, ela vai dormir

if __name__ == '__main__':  
    unittest.main()
```

`setUp` e `tearDown` são métodos especiais que rodam antes e depois de cada método de teste.
Com eles, evitamos duplicação de código e garantimos que certas pré e pós-condições sejam satisfeitas para cada teste.

#### COMMIT & PUSH com a mensagem: Melhorando a classe de teste com setUp e tearDown

## 6. Criando um app Django

Vamos organizar nosso projeto em um app Django.
Rode o seguinte comando na pasta do projeto:

```ShellSession
$ python manage.py startapp lists
```

Note que a pasta `lists` foi criada. Confira os arquivos gerados!

Você deve ter agora a seguinte estrutura de arquivos na pasta do seu projeto:

```
.
├── db.sqlite3
├── functional_tests.py
├── lists
│   ├── admin.py
│   ├── apps.py
│   ├── __init__.py
│   ├── migrations
│   │   └── __init__.py
│   ├── models.py
│   ├── tests.py
│   └── views.py
├── manage.py
├── superlists
│   ├── __init__.py
│   ├── __pycache__
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
└── virtualenv
    ├── [...]
```

#### COMMIT & PUSH com a mensagem: Criando um app Django

Lembre-se de incluir os arquivos que foram gerados (isto é, **lists**) através do comando acima.

## 7. Testes Funcionais vs. Testes de Unidade

Até o momento, criamos apenas Testes Funcionais, isto é, testes do ponto de vista do usuário (externo).
A partir de agora passamos a criar outro tipo de teste: **Testes de Unidades**, isto é, testes automatizados de pequenas unidades de código (ex: classes, métodos) de forma isolada. Diferentemente dos Testes Funcionais, os Testes de Unidade focam no ponto de vista do desenvolvedor (interno). 

Nossa abordagem de TDD cobre tantos os Testes Funcionais quanto os Testes de Unidade.
Portanto, nosso fluxo de trabalho será o seguinte:

1. Começamos escrevendo um **Teste Funcional** sobre uma nova funcionalidade do ponto de vista do usuário.
2. Uma vez que temos um Teste Funcional que falha, começamos a pensar em como escrever o código da aplicação para passar no teste. Nesse momento, utilizamos um ou mais **Testes de Unidade** para definir o comportamento do nosso código. A ideia é que cada linha de código da nossa aplicação seja coberta por pelo menos um Teste de Unidade.
3. Uma vez que temos um Teste de Unidade que falha, escremos o **código da aplicação** mínimo para o teste de unidade passar (podemos iterar nos passos 2 e 3 algumas vezes).
4. Por fim, rodamos novamento nosso **Teste Funcional** e verificamos se ele passa ou avançou um pouco (podemos voltar para os passos 2 e 3, se necessário).

Algumas diferenças:
Testes Funcionais dirigem o desenvolvimento em alto nível, enquanto Testes de Unidade focam em um nível mais baixo.
Testes Funcionais ajudam a construir uma aplicação com funcionalidades corretas, enquanto Testes de Unidade ajudam a escrever código limpo e sem bug.

Quais outras diferenças existem entre Testes Funcionais e Testes de Unidade?

#### COMMIT & PUSH com a mensagem: Testes Funcionais vs. Testes de Unidade

Crie um arquivo chamado **funcionais-vs-unidade.txt**, escreva duas diferenças simples entre Testes Funcionais e Testes de Unidade, e faça o commit desse arquivo.

## 8. Iniciando com Testes de Unidade

Abra o arquivo **lists/tests.py**:

```Python
from django.test import TestCase

# Create your tests here.
```

Veja que Django sugere utilizar uma versão especial de `TestCase`, que é uma versão melhorada de `unittest.TestCase`.

Vamos fazer um teste bem simples apenas para aprender a rodar um Teste de Unidade.
Altere o arquivo **lists/tests.py** conforme abaixo:

```Python
from django.test import TestCase

class SmokeTest(TestCase):

    def test_bad_maths(self):
        self.assertEqual(1 + 1, 3)
 ```

E agora rode o Teste de Unidade com o comando `python manage.py test`:

```ShellSession
$ python manage.py test
Creating test database for alias 'default'...
F
======================================================================
FAIL: test_bad_maths (lists.tests.SmokeTest)
 ---------------------------------------------------------------------
Traceback (most recent call last):
  File "...python-tdd-book/lists/tests.py", line 6, in test_bad_maths
    self.assertEqual(1 + 1, 3)
AssertionError: 2 != 3

 ---------------------------------------------------------------------
Ran 1 test in 0.001s

FAILED (failures=1)
System check identified no issues (0 silenced).
Destroying test database for alias 'default'...
```

#### COMMIT & PUSH com a mensagem: Iniciando com Testes de Unidade

## 9. Um pouco de arquitetura web: MVC, URLs e View

Django é estruturado no padrão clássico [MVC](https://pt.wikipedia.org/wiki/MVC) (Model-View-Controller).
Em suma, a principal função do framework Django é decidir o que fazer quando um usuário solicita uma URL no nosso site.

O fluxo de trabalho do Django é o seguinte:

1. Requisição: Uma requisição HTTP chega para uma URL (por exemplo, um usuário do app TODO clicou em *adicionar tarefa*).
2. Mapeamento: Django utiliza algumas regras para decidir qual view vai lidar com a requisição (passo conhecido como resolvendo URL).
3. Resposta: A view processa a requisição e retorna uma resposta HTTP.

Então, queremos testar duas coisas:

- Podemos *resolver* (ou seja, mapear) a URL do página inicial (isto é, "/") do nosso site para uma view em particular?
- Podemos fazer essa view retornar algum HTML para que nosso Teste Funcional passe?

Altere o arquivo **lists/tests.py** para:

```Python
from django.urls import resolve
from django.test import TestCase
from lists.views import home_page

class HomePageTest(TestCase):

    def test_root_url_resolves_to_home_page_view(self):
        found = resolve('/')  
        self.assertEqual(found.func, home_page)
```

Note alguns pontos importantes:

1. `resolve` é função que o Django utiliza internamente para resolver URLs e mapear para *views*. Estamos verificando se `resolve`, quando chamado com "/" (isto é, a página inicial), encontra uma função chamada `home_page`.
2. Que função é essa `home_page`? Ela ainda não existe! Estamos aplicando TDD, logo o código da nossa aplicação ainda será escrito.

Rode o Teste de Unidade novamente:

```ShellSession
$ python manage.py test
ImportError: cannot import name 'home_page'
```

Erro esperado: estamos tentando importar algo que ainda nem existe.
Note que agora temos ambos os testes (Funcional e de Unidade) falhando.
Isso representa boas novas no TDD.

#### COMMIT & PUSH com a mensagem: Um pouco de arquitetura web: MVC, URLs e View

## 10. Escrevendo código da aplicação

Finalmente, vamos escrever algum código da aplicação. Lembre-se, com TDD, o código é desenvolvido em pequenos passos (baby steps). Como estamos aprendendo TDD, esses passos são ainda menores.

Para fazer o nosso Teste de Unidade passar, podemos atualizar o código de **lists/views.py** como segue:

```Python
from django.shortcuts import render

# Create your views here.
home_page = None
```

E atualizar o mapeamento das URLs em **superlists/urls.py**:

```Python
from django.conf.urls import url
from lists import views

urlpatterns = [
    url(r'^$', views.home_page, name='home'),
]
```

A URL acima mapeia origem e destino da requisição. O destino pode ser uma URL ou uma view.
A expressão regular `^$` representa uma string vazia, isto é, a nossa página inicial "/".

Rode o Teste de Unidade novamente:

```ShellSession
$ python manage.py test
...
TypeError: view must be a callable or a list/tuple in the case of include().
```

Importante: o Teste de Unidade fez o link entre "/" (presente no arquivo **superlists/urls.py**) e `home_page = None` (presente no arquivo **lists/views.py**).
A mensagem de erro fala que a view não é *chamável*.

Logo, em **lists/views.py**, mudamos a definição de `home_page` para função:

```Python
from django.shortcuts import render

# Create your views here.
def home_page():
    pass
```

Finalmente, rodamos nosso Teste de Unidade novamente:

```ShellSession
$ python manage.py test
Creating test database for alias 'default'...
.
 ---------------------------------------------------------------------
Ran 1 test in 0.003s

OK
System check identified no issues (0 silenced).
Destroying test database for alias 'default'...
```

Enfim, temos um Teste de Unidade passando!

#### COMMIT & PUSH com a mensagem: Escrevendo código da aplicação

## 11. Melhorando o Teste de Unidade

Abra o arquivo **lists/tests.py** e adicione o método de teste `test_home_page_returns_correct_html`:

```Python
from django.urls import resolve
from django.test import TestCase
from django.http import HttpRequest

from lists.views import home_page

class HomePageTest(TestCase):

    def test_root_url_resolves_to_home_page_view(self):
        found = resolve('/')
        self.assertEqual(found.func, home_page)

    def test_home_page_returns_correct_html(self):
        request = HttpRequest()
        response = home_page(request)
        html = response.content.decode('utf8')
        self.assertTrue(html.startswith('<html>'))
        self.assertIn('<title>To-Do lists</title>', html)
        self.assertTrue(html.endswith('</html>'))
```

Explicando:

1. Criamos um objeto `HttpRequest`, que representa uma requisição do usuário para uma página.
2. Passamos essa requisição para a view `home_page`, que nos fornece uma resposta (objeto `HttpResponse`).
3. Depois, extraímos o conteúdo da resposta e convertemos em HTML através de `decode`.
4. Queremos que nossa página comece com `<html>` e termine com `</html>`, como qualquer página web.
5. Queremos uma tag `<title>` em algum lugar com as palavras "To-Do lists" (isso está especificado em nosso Teste Funcional).

Note que, de certa forma, o Teste de Unidade é guiado pelo Teste Funcional (por exemplo, a verificação de "To-Do" no Teste de Unidade é motivada pelo Teste Funcional), mas o Teste de Unidade é muito mais próximo do código da aplicação.

Rode o Teste de Unidade para obter a falha esperada:

```ShellSession
$ python manage.py test
...
TypeError: home_page() takes 0 positional arguments but 1 was given
```

#### COMMIT & PUSH com a mensagem: Melhorando o Teste de Unidade

## 12. Ciclo: Teste de Unidade/Desenvolvimento

Podemos estabelecer um ciclo de trabalho entre Testes de Unidade e código da aplicação:

1. No terminal, rode os Testes de Unidade e veja como eles falham.
2. Desenvolva o código mínimo da aplicação para endereçar a falha (volte para o passo 1).

Ideia central: cada trecho de código criado é justificado por um Teste de Unidade.

Isso pode parecer trabalhoso e cansativo em um primeiro momento (e, de fato, será).
Entretanto, de acordo com a filosofia TDD, ao se acostumar a esse "ritmo" de trabalho, você irá programar mais rapidamente, mesmo que com pequenos passos. Isso é como o código da aplicação é completamente implementado: teste -> codificação.

Vamos exercitar esse ciclo.

2. Mudança mínima na aplicação (**lists/views.py**, adição de parâmetro):

```Python
def home_page(request):
    pass
```

1. Teste de Unidade:

```ShellSession
$ python manage.py test
...
AttributeError: 'NoneType' object has no attribute 'content'
```

2. Mudança mínima na aplicação (**lists/views.py**, retornar `django.http.HttpResponse`):

```Python
from django.http import HttpResponse

# Create your views here.
def home_page(request):
    return HttpResponse()
```

1. Teste de Unidade novamente:

```ShellSession
$ python manage.py test
   ...
   self.assertTrue(html.startswith('<html>'))
AssertionError: False is not true
```

2. Mudança mínima na aplicação (**lists/views.py**, adicionando tag `<html>`):

```Python
from django.http import HttpResponse

# Create your views here.
def home_page(request):
    return HttpResponse('<html>')
```

1. Mais Teste de Unidade:

```ShellSession
$ python manage.py test
   ...
    self.assertIn('<title>To-Do lists</title>', html)
AssertionError: '<title>To-Do lists</title>' not found in '<html>'
```

2. Mudança mínima na aplicação (**lists/views.py**):

```Python
from django.http import HttpResponse

# Create your views here.
def home_page(request):
    return HttpResponse('<html><title>To-Do lists</title>')
```

1. Teste de Unidade novamente:

```ShellSession
$ python manage.py test
   ...
    self.assertTrue(html.endswith('</html>'))
AssertionError: False is not true
```

Quase lá!

2. Mudança mínima na aplicação (**lists/views.py**):

```Python
from django.http import HttpResponse

# Create your views here.
def home_page(request):
    return HttpResponse('<html><title>To-Do lists</title></html>')
```

1. Teste de Unidade novamente:

```ShellSession
$ python manage.py test
Creating test database for alias 'default'...
..
 ---------------------------------------------------------------------
Ran 2 tests in 0.001s

OK
System check identified no issues (0 silenced).
Destroying test database for alias 'default'...
```

O Teste de Unidade passou.

Por fim, vamos rodar o Teste Funcional (que estava quebrado desde a etapa 5).

```ShellSession
$ python functional_tests.py 
.
----------------------------------------------------------------------
Ran 1 test in 4.568s

OK
```

O Teste Funcional também passou.

Palavra final: TDD é uma *disciplina* (ou seja, possui regras e você deve segui-las), logo essa prática não surge naturalmente.
O custo-benefício não é imediato, mas a médio e longo prazo: a aplicação terá um projeto melhor, alta testabilidade e alta cobertura de teste.

#### COMMIT & PUSH com a mensagem: Ciclo: Teste de Unidade/Desenvolvimento
