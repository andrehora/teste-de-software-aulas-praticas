# Teste de Software

## Aula Prática 2: Testando Interações do Usuário, Regressão e Banco de Dados

Prof. André Hora

Objetivo: Reproduzir uma sessão de desenvolvimento de software usando TDD (Test-Driven Development), com base no livro *Test-Driven-Development with Python*.
Especificamente, iremos reproduzir os exemplos do Cap. 4 e 5 do livro. 
Faça o exercício com o mindset de TDD.
Se necessário, estude antes o material visto em sala de aula.

Instruções:

- Siga o roteiro, passo a passo.
- Após cada passo, rode os testes.
- Nos passos marcados com COMMIT & PUSH, faça um commit e um push no seu repositório. 
Isso será usado no momento da correção, para garantir que seguiu a sequência do roteiro, passo-a-passo.

## 1. Utilizando Selenium para Testar Interações do Usuário

Finalizamos a Aula Prática 1 rodando com sucesso nosso Teste Funcional **functional_tests.py**:

```ShellSession
$ python functional_tests.py 
.
----------------------------------------------------------------------
Ran 1 test in 4.568s

OK
```

No entanto, o Teste Funcional possui diversas funcionalidades ainda não implementadas.

Vamos estender nosso Teste Funcional.
Modifique **functional_tests.py** conforme segue:

```Python
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
import time
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
        header_text = self.browser.find_element_by_tag_name('h1').text  
        self.assertIn('To-Do', header_text)

        # Ela é convidada a entrar com um item TODO imediatamente
        inputbox = self.browser.find_element_by_id('id_new_item')  
        self.assertEqual(inputbox.get_attribute('placeholder'), 'Enter a to-do item')

        # Ela digita "Estudar testes funcionais" em uma caixa de texto
        inputbox.send_keys('Estudar testes funcionais')

        # Quando ela aperta enter, a página atualiza, e mostra a lista
        # "1: Estudar testes funcionais" como um item da lista TODO
        inputbox.send_keys(Keys.ENTER)
        time.sleep(1)
        
        table = self.browser.find_element_by_id('id_list_table')
        rows = table.find_elements_by_tag_name('tr')  
        self.assertTrue(
            any(row.text == '1: Estudar testes funcionais' for row in rows)
        )
        
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

Estamos utilizando diversos métodos do Selenium para examinar páginas web: `find_element_by_tag_name`, `find_element_by_id` e `find⁠_ele⁠ment⁠s⁠_by⁠_tag_name`.
Também usamos `send_keys`, que é a forma do Selenium "digitar" elementos de entrada (`Keys.ENTER` simula o Enter).
Após o Enter, a página deve atualizar.
O `time.sleep(1)` garante que atualização termine antes antes de acessarmos a nova página (isso é chamado de "explicit wait").

Rode o Teste Funcional:

```ShellSession
$ python functional_tests.py
[...]
selenium.common.exceptions.NoSuchElementException: Message: Unable to locate element: h1
```

#### COMMIT & PUSH com a mensagem: Utilizando Selenium para Testar Interações do Usuário

## 2. Refatorando a aplicação para utilizar templates

No momento, nossa view (**lists/views.py**) está misturando conteúdo e apresentação HTML, o que não é uma boa prática em desenvolvimento web.
Além disso, nossa view está gerando conteúdo HTML de forma estática, o que também não é uma boa prática em desenvolvimento web.

**Templates** permitem que o conteúdo de páginas HTML sejam geradas dinamicamente.
**Templates** também permitem separar conteúdo (qual dado você vê) da apresentação (como você vê).

Nesta etapa, queremos que a nossa view retorne exatamente o mesmo HTML, mas utilizando outro processo mais elegante, isto é, **templates**.
Isso é uma [refatoração](https://refactoring.com): melhorar o código sem modificar seu comportamento.

A primeira regra da refatoração é que não podemos refatorar sem testes!
Testes ajudam a garantir que o comportamento da aplicação não será modificado
([muita atenção ao refatorar](http://www.obeythetestinggoat.com/static/images/refactoring_cat.gif)).

Então, vamos rodar o nosso Teste de Unidade para garantir que estão passando:

```ShellSession
$ python manage.py test
[...]
OK
```

Em seguida, crie um diretório **lists/templates** e adicione o seguinte arquivo **lists/templates/home.html**, conforme segue:

```html
<html>
    <title>To-Do lists</title>
</html>
```

Após, altere a nossa view (**lists/views.py**) de:

```Python
from django.http import HttpResponse

# Create your views here.
def home_page(request):
    return HttpResponse('<html><title>To-Do lists</title></html>')
```

Para:

```Python
from django.shortcuts import render

def home_page(request):
    return render(request, 'home.html')
```

Note que, ao invés de construirmos `HttpResponse`, agora utilizamos a função `render` do Django.
O framework Django irá procurar pastas chamadas **templates** e construir uma `HttpResponse` com base no conteúdo do template.

Rode novamente o Teste de Unidade:

```ShellSession
$ python manage.py test
[...]
======================================================================
ERROR: test_home_page_returns_correct_html (lists.tests.HomePageTest)
 ---------------------------------------------------------------------
Traceback (most recent call last):
  File "...python-tdd-book/lists/tests.py", line 17, in
test_home_page_returns_correct_html
    response = home_page(request)
  File "...python-tdd-book/lists/views.py", line 5, in home_page
    return render(request, 'home.html')
  File "/usr/local/lib/python3.6/dist-packages/django/shortcuts.py", line 48,
in render
    return HttpResponse(loader.render_to_string(*args, **kwargs),
  File "/usr/local/lib/python3.6/dist-packages/django/template/loader.py", line
170, in render_to_string
    t = get_template(template_name, dirs)
  File "/usr/local/lib/python3.6/dist-packages/django/template/loader.py", line
144, in get_template
    template, origin = find_template(template_name, dirs)
  File "/usr/local/lib/python3.6/dist-packages/django/template/loader.py", line
136, in find_template
    raise TemplateDoesNotExist(name)
django.template.base.TemplateDoesNotExist: home.html

 ---------------------------------------------------------------------
Ran 2 tests in 0.004s

```

Ao ler a traceback, podemos verificar que o template não pode ser encontrado (TemplateDoesNotExist).

Devemos realizar uma pequena configuração no Django para resolver esse problema.
Abra o arquivo **superlists/settings.py**, e adicione `lists` na variável `INSTALLED_APPS`, conforme abaixo:

```Python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'lists',
]
```

Rode o Teste de Unidade novamente:

```Python
$ python manage.py test
[...]
OK
```

*OBS: Dependendo do editor de texto utilizado, uma quebra de linha pode ser adicionada no final do arquivo.
Logo, caso o seu Teste de Unidade (**lists/tests.py**) falhe, atualize-o conforme abaixo:*

```Python
self.assertTrue(html.strip().endswith('</html>'))
```

Nossa refatoração está completa.
Iniciamos essa etapa com o Teste de Unidade passando, alteramos o código da aplicação e ao rodar o Teste de Unidade novamente, garantimos que o comportamento foi preservado.

#### COMMIT & PUSH com a mensagem: Refatorando a aplicação para utilizar templates

## 3. Utilizando o Django Test Client

Vamos melhorar nosso Teste de Unidade para utilizar uma ferramenta do framework Django chamada [Django Test Client](https://docs.djangoproject.com/en/1.11/topics/testing/tools/#the-test-client).
Essa ferramenta simula um Web browser, permitindo testar views e interagir com a aplicação Django programaticamente.
Além disso, essa ferramenta não requer que o servidor esteja rodando, logo ela ajuda na criação dos testes de unidade.

Mude o método `test_home_page_returns_correct_html` do Teste de Unidade (**lists/tests.py**), de:

```Python
def test_home_page_returns_correct_html(self):
    request = HttpRequest()
    response = home_page(request)
    html = response.content.decode('utf8')
    self.assertTrue(html.startswith('<html>'))
    self.assertIn('<title>To-Do lists</title>', html)
    self.assertTrue(html.strip().endswith('</html>'))
```
 
Para:

```Python
def test_home_page_returns_correct_html(self):
    response = self.client.get('/')  

    html = response.content.decode('utf8')  
    self.assertTrue(html.startswith('<html>'))
    self.assertIn('<title>To-Do lists</title>', html)
    self.assertTrue(html.strip().endswith('</html>'))

    self.assertTemplateUsed(response, 'home.html') 
 ```
 
 Note que, estamos utilizando `self.client.get`, ao invés de criar `HttpRequest`.
 Além disso, estamos verificando se o template correto foi chamado através de `assertTemplateUsed`.
 
 Rode o Teste de Unidade para garantir que o comportamento foi mantido:
 
 ```ShellSession
$ python manage.py test
...
Ran 2 tests in 0.047s

OK
 ```
 
Apenas para verificar que o Django Test Client está funcionando corretamente, faça a seguinte alteração no Teste de Unidade e rode-o novamente:

```Python
self.assertTemplateUsed(response, 'wrong.html')
```

```ShellSession
AssertionError: False is not true : Template 'wrong.html' was not a template
used to render the response. Actual template(s) used: home.html
```

Conforme esperado, o Teste de Unidade falhou, ou seja, a ferramenta de testes está funcionando conforme o esperado.
Altere novamente o Teste de Unidade para utilizar a view home: `self.assertTemplateUsed(response, 'home.html')`.

Vamos agora simplificar nosso Teste de Unidade (**lists/tests.py**).
Podemos remover `test_root_url_resolves`, pois isso é testado implicitamente pelo Django Test Client.
Podemos também remover os asserts do método principal, testando somente se o template correto está sendo utilizado:

```Python
from django.test import TestCase

class HomePageTest(TestCase):

    def test_uses_home_template(self):
        response = self.client.get('/')
        self.assertTemplateUsed(response, 'home.html')
```

#### COMMIT & PUSH com a mensagem: Utilizando o Django Test Client

## 4. Melhorando a Página Inicial

No momento, o Teste Funcional está falhando.
Vamos alterar a página inicial da aplicação para que o teste possa passar.

Modifique nosso template home (**lists/templates/home.html**) para:

```html
<html>
    <head>
        <title>To-Do lists</title>
    </head>
    <body>
        <h1>Your To-Do list</h1>
    </body>
</html>
```

E rode o Teste Funcional:

```ShellSession
$ python functional_tests.py
...
selenium.common.exceptions.NoSuchElementException: Message: Unable to locate element: [id="id_new_item"]
----------------------------------------------------------------------
Ran 1 test in 5.842s

FAILED (errors=1)
```

Modifique nosso template home (**lists/templates/home.html**) para:

```
<html>
    <head>
        <title>To-Do lists</title>
    </head>
    <body>
        <h1>Your To-Do list</h1>
        <input id="id_new_item" />
    </body>
</html>
```

E rode o Teste Funcional novamente:

```ShellSession
$ python functional_tests.py
...
AssertionError: '' != 'Enter a to-do item'
+ Enter a to-do item
----------------------------------------------------------------------
Ran 1 test in 4.707s

FAILED (failures=1)
```

Modifique nosso template home (**lists/templates/home.html**) para:

```html
<html>
    <head>
        <title>To-Do lists</title>
    </head>
    <body>
        <h1>Your To-Do list</h1>
        <input id="id_new_item" placeholder="Enter a to-do item"/>
    </body>
</html>
```

E rode o Teste Funcional mais uma vez:

```ShellSession
$ python functional_tests.py
...
selenium.common.exceptions.NoSuchElementException: Message: Unable to locate element: [id="id_list_table"]
----------------------------------------------------------------------
Ran 1 test in 5.825s

FAILED (errors=1)
```

Modifique nosso template home (**lists/templates/home.html**) para:

```html
<html>
    <head>
        <title>To-Do lists</title>
    </head>
    <body>
        <h1>Your To-Do list</h1>
        <input id="id_new_item" placeholder="Enter a to-do item"/>
        <table id="id_list_table">
    </body>
</html>
```

Rode o Teste Funcional:

```ShellSession
$ python functional_tests.py
...
File "functional_tests.py", line 39, in test_can_start_a_list_and_retrieve_it_later
    any(row.text == '1: Estudar testes funcionais' for row in rows)
AssertionError: False is not true
----------------------------------------------------------------------
Ran 1 test in 6.782s

FAILED (failures=1)
```

A mensagem de erro `AssertionError: False is not true` não ajuda muito.
Vamos atualizar nosso Teste Funcional (**functional_tests.py**) para incluir uma mensagem de erro melhor:

```Python
self.assertTrue(
    any(row.text == '1: Estudar testes funcionais' for row in rows),
    "New to-do item did not appear in table"
)
```

Rode o Teste Funcional novamente para obter a nova mensagem:

```ShellSession
$ python functional_tests.py
...
File "functional_tests.py", line 39, in test_can_start_a_list_and_retrieve_it_later
    any(row.text == '1: Estudar testes funcionais' for row in rows)
AssertionError: False is not true : New to-do item did not appear in table
----------------------------------------------------------------------
Ran 1 test in 6.782s

FAILED (failures=1)
```

Para o teste passar, precisamos processar a requisição do usuário.
Vamos fazer isso nos próximos passos.

#### COMMIT & PUSH com a mensagem: Melhorando a Página Inicial

## 5. Reforçando o Processo TDD

Até o momento vimos os principais aspectos do nosso processo TDD:

- Testes Funcionais
- Testes de Unidade
- Ciclo: Teste de Unidade/Desenvolvimento
- Refatoração

De forma geral, o processo TDD pode ser visto na figura abaixo: escrever o teste, rodar o teste e verificar se falha, escrever algum código da aplicação, rodar o teste novamente e repetir até passar.
Adicionalmente, podemos refatorar nosso código, utilizando sempre o teste para garantir que não quebramos nada.

![tdd-process](http://www.obeythetestinggoat.com/book/images/twp2_0403.png)

Mas note que estamos utilizando dois tipos de teste: Testes Funcionais e Testes de Unidade.
Então, nosso processo é um pouco mais complicado, conforme apresentado na figura abaixo.

![tdd-process2](http://www.obeythetestinggoat.com/book/images/twp2_0404.png)

#### COMMIT & PUSH com a mensagem: Reforçando o Processo TDD

Crie um arquivo chamado **processo-tdd.txt**, descreva brevemente o processo apresentado na figura acima, e faça o commit desse arquivo.

## 6. Criando Form para Enviar Requisição

No momento, vamos escrever o código mais simples possível para enviar uma requisição.
Altere o template home (**lists/templates/home.html**), dando um nome para a tag `<input>` e adicionando a tag `<form>`:

```html
<html>
    <head>
        <title>To-Do lists</title>
    </head>
    <body>
        <h1>Your To-Do list</h1>
        <form method="POST">
            <input name="item_text" id="id_new_item" placeholder="Enter a to-do item" />
            {% csrf_token %}
        </form>
        <table id="id_list_table">
    </body>
</html>
```

Rode o Teste Funcional:

```ShellSession
$ python functional_tests.py
[...]
AssertionError: False is not true : New to-do item did not appear in table
```

#### COMMIT & PUSH com a mensagem: Criando Form para Enviar Requisição

## 7. Processando Requisição POST

Vamos iniciar a implementação do processamento da requisição POST no lado do servidor.
Isso significa que temos que escrever um Teste de Unidade.

Adicione o seguinte o método no nosso Teste de Unidade **lists/tests.py**:

```Python
def test_can_save_a_POST_request(self):
    response = self.client.post('/', data={'item_text': 'A new list item'})
    self.assertIn('A new list item', response.content.decode())
```

Para fazer um POST, devemos chamar o método `self.client.post`.
Esse método recebe um argumento `data`, que contém o dado a ser enviado.

Rode o Teste de Unidade e observe a falha esperada:

```ShellSession
$ python manage.py test
[...]
AssertionError: 'A new list item' not found in '<html>\n    <head>\n
<title>To-Do lists</title>\n    </head>\n    <body>\n        <h1>Your To-Do
list</h1>\n        <form method="POST">\n            <input name="item_text"
[...]
</body>\n</html>\n'
```

Podemos fazer o Teste de Unidade passar alterando a nossa view.
Faça a seguinte alteração em **lists/views.py** e, sem seguida, rode o Teste de Unidade novamente:

```Python
from django.http import HttpResponse
from django.shortcuts import render

def home_page(request):
    if request.method == 'POST':
        return HttpResponse(request.POST['item_text'])
    return render(request, 'home.html')
```

```ShellSession
$ python manage.py test
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
..
----------------------------------------------------------------------
Ran 2 tests in 0.013s

OK

```

Nosso Teste de Unidade passou, mas, claro, isso não é exatamente a implementação desejada.
Queremos adicionar a requisição POST em uma tabela na página inicial.

#### COMMIT & PUSH com a mensagem: Processando Requisição POST

## 8. Passando Variáveis no Template

Finalmente, vamos começar a conhecer o poder dos Templates Django, que é passar variáveis da view para o template HTML.
A sintaxe dos templates permite incluir objetos Python no template através da notação `{{ ... }}`.

Altere a nosso template **lists/templates/home.html** para:

```html
<html>
    <head>
        <title>To-Do lists</title>
    </head>
    <body>
        <h1>Your To-Do list</h1>
        <form method="POST">
            <input name="item_text" id="id_new_item" placeholder="Enter a to-do item" />
            {% csrf_token %}
        </form>

        <table id="id_list_table">
            <tr><td>{{ new_item_text }}</td></tr>  
        </table>
    </body>
</html>
```

Vamos ajustar nosso Teste de Unidade para garantir que o template correto está sendo utilizado:

```Python
def test_can_save_a_POST_request(self):
   response = self.client.post('/', data={'item_text': 'A new list item'})
   self.assertIn('A new list item', response.content.decode())
   self.assertTemplateUsed(response, 'home.html')
```

E rode o Teste de Unidade:

```ShellSession
$ python manage.py test
...
AssertionError: No templates used to render the response
```

Conforme esperado, o teste falhou.
Isso ocorreu pois o `if request.method == 'POST':` foi adicionado na view **lists/views.py** apenas para o teste passar, mas essa implementação não está servindo mais.

#### COMMIT & PUSH com a mensagem: Passando Variáveis no Template

## 9. Regressão

Vamos agora implementar a view da forma correta.
Altere a view **lists/views.py** de:

```Python
from django.http import HttpResponse
from django.shortcuts import render

def home_page(request):
    if request.method == 'POST':
        return HttpResponse(request.POST['item_text'])
    return render(request, 'home.html')
```

Para:

```Python
from django.http import HttpResponse
from django.shortcuts import render

def home_page(request):
    return render(request, 'home.html', {
        'new_item_text': request.POST['item_text'],
    })
```

Note que a função `render` recebe como terceiro argumento um dicionário que mapeia váriaveis do template para seus valores.
Nesse caso, a váriavel do template é `new_item_text` e o seu valor é `request.POST['item_text']`, ou seja, a requisição POST.

Em teoria, está tudo ok.
Rode o Teste de Unidade novamente:

```ShellSession
$ python manage.py test
...
ERROR: test_uses_home_template (lists.tests.HomePageTest)
[...]
  File "...python-tdd-book/lists/views.py", line 5, in home_page
    'new_item_text': request.POST['item_text'],
[...]
django.utils.datastructures.MultiValueDictKeyError: "'item_text'"
```

Veja que a falha inesperada ocorreu em outro teste: `ERROR: test_uses_home_template (lists.tests.HomePageTest)`.
Ou seja, o teste que falhou foi o nosso primeiro teste criado (isto é, `test_uses_home_template`) e não o teste que estamos trabalhando atualmente (isto é, `test_can_save_a_POST_request`).

Trata-se de uma consequência inesperada, ou seja, uma **regressão**: quebramos uma parte da aplicação que estava funcionando corretamente (o teste que quebrou, `test_uses_home_template`, trata dos casos onde não existem requisições POST, mas a nossa implementação assume sempre que um valor deve ser passado via `new_item_text`).
Essa é uma das razões principais para se ter testes: nosso teste evitou que a aplicação fosse acidentalmente quebrada.
Além disso, como estamos utilizando TDD, descobrimos o problema imediatamente.

Vamos corrigir a nossa view **lists/views.py** para os lidar com os casos que não estamos fazendo uma requisição POST, e, consequentemente, o dicionário deve estar vazio.
Devemos tratar esse caso apenas passando um valor default `''` (string vazia) para `new_item_text` caso não exista requisição POST, conforme segue:

```Python
def home_page(request):
    return render(request, 'home.html', {
        'new_item_text': request.POST.get('item_text', ''),
    })
```

O Teste de Unidade deve passar:

```ShellSession
$ python manage.py test
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
..
----------------------------------------------------------------------
Ran 2 tests in 0.018s

OK
```

#### COMMIT & PUSH com a mensagem: Regressão

## 10. Melhorando e Expandindo o Teste Funcional

Vamos voltar para o nosso Teste Funcional.
Rode-o:

```ShellSession
$ python functional_tests.py
...
AssertionError: False is not true : New to-do item did not appear in table
```

Veja que o teste falhou: não foi possível passar no último assert.

Antes de resolver o problema, vamos realizar uma pequena melhoria no Teste Funcional.
Torne o último assert do Teste Funcional um pouco mais fácil de entender, alterando de `assertTrue` para `assertIn`.
Para isso, altere **functional_tests.py** de:

```Python
self.assertTrue(
    any(row.text == '1: Estudar testes funcionais' for row in rows),
    "New to-do item did not appear in table"
)
```

Para:

```Python
self.assertIn('1: Estudar testes funcionais', [row.text for row in rows])
```

Rode o Teste Funcional novamente e verifique uma mensagem de erro mais legível:

```ShellSession
$ python functional_tests.py
...
self.assertIn('1: Estudar testes funcionais', [row.text for row in rows])
AssertionError: '1: Estudar testes funcionais' not found in ['Estudar testes funcionais']
```

O Teste Funcional requer uma enumeração na listagem com um `1:` no início do primeiro item.
A melhor forma de passar rapidamente é alterando o nosso template **lists/templates/home.html**:

```html
<tr><td>1: {{ new_item_text }}</td></tr>  
```

Rode o Teste Funcional novamente, que deverá passar:

```ShellSession
$ python functional_tests.py 
.
----------------------------------------------------------------------
Ran 1 test in 5.920s
OK
```

Vamos expandir nosso Teste Funcional para adicionar um segundo item na tabela (isto é, `2. Estudar testes de unidade`), conforme apresentado abaixo:


```Python
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
import time
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
        header_text = self.browser.find_element_by_tag_name('h1').text  
        self.assertIn('To-Do', header_text)

        # Ela é convidada a entrar com um item TODO imediatamente
        inputbox = self.browser.find_element_by_id('id_new_item')  
        self.assertEqual(inputbox.get_attribute('placeholder'), 'Enter a to-do item')

        # Ela digita "Estudar testes funcionais" em uma caixa de texto
        inputbox.send_keys('Estudar testes funcionais')

        # Quando ela aperta enter, a página atualiza, e mostra a lista
        # "1: Estudar testes funcionais" como um item da lista TODO
        inputbox.send_keys(Keys.ENTER)
        time.sleep(1)
        
        table = self.browser.find_element_by_id('id_list_table')
        rows = table.find_elements_by_tag_name('tr')  
        self.assertIn('1: Estudar testes funcionais', [row.text for row in rows])
        
        # Ainda existe uma caixa de texto convidando para adicionar outro item
        # Ela digita: "Estudar testes de unidade"
        inputbox = self.browser.find_element_by_id('id_new_item')  
        inputbox.send_keys('Estudar testes de unidade')
        inputbox.send_keys(Keys.ENTER)
        time.sleep(1)

        # A página atualiza novamente, e agora mostra ambos os itens na sua lista
        table = self.browser.find_element_by_id('id_list_table')
        rows = table.find_elements_by_tag_name('tr')
        self.assertIn('1: Estudar testes funcionais', [row.text for row in rows])
        self.assertIn('2: Estudar testes de unidade', [row.text for row in rows])

        # Maria se pergunta se o site vai lembrar da sua lista. Então, ela verifica que
        # o site gerou uma URL única para ela -- existe uma explicação sobre essa feature

        # Ela visita a URL: a sua lista TODO ainda está armazenada

        # Satisfeita, ela vai dormir

if __name__ == '__main__':
    unittest.main()
```

Rode o Teste Funcional:

```ShellSession
$ python functional_tests.py
...
self.assertIn('1: Estudar testes funcionais', [row.text for row in rows])
AssertionError: '1: Estudar testes funcionais' not found in ['1: Estudar testes de unidade']
```

Falha esperada?

#### COMMIT & PUSH com a mensagem: Melhorando e Expandindo o Teste Funcional

## 11. Refatorando o Teste Funcional

Na última modificação, introduzimos um [bad smell](https://en.wikipedia.org/wiki/Code_smell) no Teste Funcional.
Antes de continuar a leitura, analise o Teste Funcional acima e pense por um momento: qual seria esse bad smell?

O problema foi que inserimos duplicação de código, que está relacionado ao princípio
[Don’t Repeat Yourself (DRY)](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself).
Vamos refatorar o Teste Funcional para evitar duplicação.

Adicione o seguinte método em **functional_tests.py**:

```Python
def check_for_row_in_list_table(self, row_text):
   table = self.browser.find_element_by_id('id_list_table')
   rows = table.find_elements_by_tag_name('tr')
   self.assertIn(row_text, [row.text for row in rows])
```

E atualize o método `test_can_start_a_list_and_retrieve_it_later` com seguinte trecho:

```Python
    def test_can_start_a_list_and_retrieve_it_later(self): 
        ...
        # Quando ela aperta enter, a página atualiza, e mostra a lista
        # "1: Estudar testes funcionais" como um item da lista TODO
        inputbox.send_keys(Keys.ENTER)
        time.sleep(1)
        self.check_for_row_in_list_table('1: Estudar testes funcionais')
        
        # Ainda existe uma caixa de texto convidando para adicionar outro item
        # Ela digita: "Estudar testes de unidade"
        inputbox = self.browser.find_element_by_id('id_new_item')  
        inputbox.send_keys('Estudar testes de unidade')
        inputbox.send_keys(Keys.ENTER)
        time.sleep(1)

        # A página atualiza novamente, e agora mostra ambos os itens na sua lista
        self.check_for_row_in_list_table('1: Estudar testes funcionais')
        self.check_for_row_in_list_table('2: Estudar testes de unidade')
        ...
```

Rode o Teste Funcional:

```ShellSession
$ python functional_tests.py 
...
AssertionError: '1: Estudar testes funcionais' not found in ['1: Estudar testes de unidade']
----------------------------------------------------------------------
Ran 1 test in 8.223s
FAILED (failures=1)
```

Ocorreu a falha esperada, pois nossa funcionalidade ainda não está implementada.

#### COMMIT & PUSH com a mensagem: Refatorando o Teste Funcional

## 12. Persistindo os Dados

Precisamos persistir os dados inseridos pelo usuário.
Django vem com um excelente Object-Relational Mapper (ORM), uma camada de abstração para armazenar dados em um banco de dados com tabelas, colunas e registros (classes são mapeadas para tabelas, atributos são mapeados para colunas e instâncias da classe são mapeadas para registros no banco).

Como sempre, iniciamos pelos testes.
Adicione uma nova classe no Teste de Unidade **lists/tests.py**:

```Python
from lists.models import Item
...

class ItemModelTest(TestCase):

    def test_saving_and_retrieving_items(self):
        first_item = Item()
        first_item.text = 'O primeiro item'
        first_item.save()

        second_item = Item()
        second_item.text = 'O segundo item'
        second_item.save()

        saved_items = Item.objects.all()
        self.assertEqual(saved_items.count(), 2)

        first_saved_item = saved_items[0]
        second_saved_item = saved_items[1]
        self.assertEqual(first_saved_item.text, 'O primeiro item')
        self.assertEqual(second_saved_item.text, 'O segundo item')
```

Note que para criar um novo registro no banco de dados basta criar um objeto, atribuir atributos e chamar a função `.save()`.
Django também fornece uma API para consultar o banco através do atributo de classe `.objects`.
Estamos utilizando uma consulta bem simples, `.all()`, que recupera todos os registros da tabela.
Os resultados das consultas são do tipo `QuerySet` (parecido com uma lista), que possui funções como `.count()`.

Nosso teste possui agora uma dependência externa, ou seja, o banco de dados.
Logo, na verdade, o último teste que escrevemos pode ser considerado um **Teste de Integração**.
Vamos voltar a esse tópico quando falarmos de Isolamento de Testes e Mocks, mais a frente no curso.

Vamos iniciar mais um Ciclo: Teste de Unidade/Desenvolvimento.

Rode o Teste de Unidade:

```ShellSession
$ python manage.py test
...
ImportError: cannot import name 'Item'
```

Crie a classe `Item` em **lists/models.py**:

```Python
from django.db import models

class Item(object):
    pass
```

Rode o Teste de Unidade novamente:

```ShellSession
$ python manage.py test
AttributeError: 'Item' object has no attribute 'save'
```

Atualize a classe `Item` em **lists/models.py**:

```Python
from django.db import models

class Item(models.Model):
    pass
```

Rode o Teste de Unidade e note um erro de banco de dados:

```ShellSession
$ python manage.py test
...
django.db.utils.OperationalError: no such table: lists_item
```

Precisamos realizar uma pequena configuração no Django para que os dados possam ser persistidos.
Já vimos que o ORM modela o banco de dados: classes->tabelas, atributos->colunas, instâncias->registros.
Existe um outro sistema no Django encarregado de criar/alterar o banco de dados: **migrations**.

Rode o seguinte comando `makemigrations` para configurar o BD, conforme segue:

```ShellSession
$ python manage.py makemigrations
Migrations for 'lists':
  lists/migrations/0001_initial.py
    - Create model Item
```

Note que o arquivo **0001_initial.py** foi criado em **lists/migrations**, que representa as alterações realizadas em **models.py**.

Rode o Teste de Unidade novamente:

```ShellSession
$ python manage.py test
AttributeError: 'Item' object has no attribute 'text'
```

Atualize a classe **lists/models.py** para incluir o campo `text` do tipo `TextField` (OBS: Django possui outros tipos, tais como `IntegerField`, `CharField`, `DateField`, etc):

```Python
from django.db import models

class Item(models.Model):
    text = models.TextField(default='')
```

Ao atualizar o modelo, devemos rodar a migração novamente através do comando `makemigrations`,
para que o esquema do banco de dados seja atualizado:

```ShellSession
$ python manage.py makemigrations
Migrations for 'lists':
  lists/migrations/0002_item_text.py
    - Add field text to item
```

Note que um novo arquivo (**0002_item_text.py**) foi gerado em **lists/migrations**, representando a adição do campo `text` adiciondo em `Item`.

Por fim, rode o Teste de Unidade para ver o teste passar:

```ShellSession
$ python manage.py test
...
Ran 3 tests in 0.054s
OK
```

#### COMMIT & PUSH com a mensagem: Persistindo os Dados

## 13. Salvando o POST no Banco de Dados

Atualize o método `test_can_save_a_POST_request` do Teste de Unidade **lists/tests.py** de

```Python
def test_can_save_a_POST_request(self):
    response = self.client.post('/', data={'item_text': 'A new list item'})

    self.assertIn('A new list item', response.content.decode())
    self.assertTemplateUsed(response, 'home.html')
```

Para:

```Python
def test_can_save_a_POST_request(self):
    response = self.client.post('/', data={'item_text': 'A new list item'})

    self.assertEqual(Item.objects.count(), 1)
    new_item = Item.objects.first()
    self.assertEqual(new_item.text, 'A new list item')

    self.assertIn('A new list item', response.content.decode())
    self.assertTemplateUsed(response, 'home.html')
```

Esse teste verifica se um novo item foi salvo no BD e se texto do item está correto.
Note que estamos testando diversas coisas em um único método de teste, logo temos outro **code smell**.
Um teste de unidade longo deve ser dividido em outros testes (ou pode ser uma indicação de que o código testado está muito complicado).

Rode o Teste de Unidade:

```ShellSession
$ python manage.py test
....
    self.assertEqual(Item.objects.count(), 1)
AssertionError: 0 != 1
```

Como esperado o teste falhou pois a funcionalidade ainda não foi implementada na view.

Atualize a view **lists/views.py** de

```Python
from django.shortcuts import render

def home_page(request):
    return render(request, 'home.html', {
        'new_item_text': request.POST.get('item_text', ''),
    })
```

Para (essa implementação é bem problemática, pois salva itens em branco a cada requisição):

```Python
from django.shortcuts import render
from lists.models import Item

def home_page(request):

    item = Item()
    item.text = request.POST.get('item_text', '')
    item.save()

    return render(request, 'home.html', {
        'new_item_text': item.text
    })
```

Rode o Teste de Unidade:

```ShellSession
$ python manage.py test
...
Ran 3 tests in 0.018s
OK
```

#### COMMIT & PUSH com a mensagem: Salvando o POST no Banco de Dados

## 14. Não Salvar Items em Branco a cada Requisição

A view implementada em **lists/views.py** salva items em branco a cada requisição.
Vamos corrigir esse problema.

Primeiro criamos um novo Teste de Unidade `test_only_saves_items_when_necessary` na classe `HomePageTest` de **lists/tests.py**:

```Python
class HomePageTest(TestCase):
    
    ...
    
    def test_only_saves_items_when_necessary(self):
        self.client.get('/')
        self.assertEqual(Item.objects.count(), 0)
```

Rode o Teste de Unidade e veja a falha esperada: `AssertionError: 1 != 0`.

Atualize a view **lists/views.py** para salvar apenas as requisições POST:

```Python
from django.shortcuts import render
from lists.models import Item

def home_page(request):
    new_item_text = ''
    if request.method == 'POST':
        new_item_text = request.POST['item_text']
        Item.objects.create(text=new_item_text)

    return render(request, 'home.html', {
        'new_item_text': new_item_text
    })
```

OBS: `.objects.create` é um shorthand para criar o Item sem precisar utilizar o `.save`.

Rode o Teste de Unidade:

```ShellSession
$ python manage.py test
...
Ran 4 tests in 0.019s
OK
```

#### COMMIT & PUSH com a mensagem: Não Salvar Items em Branco a cada Requisição

## 15. Redirecionando após o POST

Uma view possui duas funções principais: (1) processar a entrada do usuário e (2) retornar a resposta apropriada.
Estamos fazendo a primeira parte, que é salvar a entrada do usuário no BD, mas precisamos trabalhar na segunda parte.

Altere o método `test_can_save_a_POST_request` do Teste de Unidade **lists/tests.py** para: 

```Python
    def test_can_save_a_POST_request(self):
        response = self.client.post('/', data={'item_text': 'A new list item'})

        self.assertEqual(Item.objects.count(), 1)
        new_item = Item.objects.first()
        self.assertEqual(new_item.text, 'A new list item')

        self.assertEqual(response.status_code, 302)
        self.assertEqual(response['location'], '/')
```

Ao rodar o Teste de Unidade, você verá o seguinte erro: `AssertionError: 200 != 302`.

Atualize a view **lists/views.py** para:

```Python
from django.shortcuts import redirect, render
from lists.models import Item

def home_page(request):
    if request.method == 'POST':
        new_item_text = request.POST['item_text']
        Item.objects.create(text=new_item_text)
        return redirect('/')

    return render(request, 'home.html')
```

#### COMMIT & PUSH com a mensagem: Redirecionando após o POST

## 16. Refatorando o Teste de Unidade (boa prática: método de teste deve testar apenas uma coisa)

As boas práticas de Teste de Unidade ditam que um método de teste deve testar apenas uma coisa, pois isso torna mais fácil rastrear bugs.
Ter múltiplos asserts em um único teste significa que se um teste falhou em um assert inicial, não sabemos o estado dos asserts seguintes.
Por exemplo, se um bug é introduzido na nossa view (levando seu teste a falhar), queremos saber se problema ocorreu (1) na parte de salvar objetos no banco ou (2) na parte de tratar a resposta.

Verifique o método `test_can_save_a_POST_request` e note que ele está atualmente testando ambos (1) salvar e (2) tratar resposta, ou seja, duas coisas distintas.

Refatore o método `test_can_save_a_POST_request` para extrair o novo método de teste `test_redirects_after_POST`, conforme segue:

```Python
    def test_can_save_a_POST_request(self):
        self.client.post('/', data={'item_text': 'A new list item'})

        self.assertEqual(Item.objects.count(), 1)
        new_item = Item.objects.first()
        self.assertEqual(new_item.text, 'A new list item')

    def test_redirects_after_POST(self):
        response = self.client.post('/', data={'item_text': 'A new list item'})

        self.assertEqual(response.status_code, 302)
        self.assertEqual(response['location'], '/')
```

Rode o Teste de Unidade:

```ShellSession
$ python manage.py test
...
Ran 5 tests in 0.019s
OK
```

#### COMMIT & PUSH com a mensagem: Refatorando o Teste de Unidade (boa prática: método de teste deve testar apenas uma coisa)

## 17. Apresentar Múltiplos Itens na Listagem

Nossa listagem TODO ainda está muito simples, e não é capaz de mostrar múltiplos itens para o usuário.
Vamos implementar essa funcionalidade, começando com a adição de um novo Tesde de Unidade na classe `HomePageTest`:

```Python
...
    def test_displays_all_list_items:
        Item.objects.create(text='itemey 1')
        Item.objects.create(text='itemey 2')

        response = self.client.get('/')

        self.assertIn('itemey 1', response.content.decode())
        self.assertIn('itemey 2', response.content.decode())
...
```

Rode o Teste de Unidade e veja a falha esperada:

```ShellSession
$ python manage.py test
...
AssertionError: 'itemey 1' not found in '<html>\n    <head>\n
```

O template do Django possui uma sintaxe para iterar em listas `{% for .. in .. %}`.
Atualize o template **lists/templates/home.html** de:

```html
...
        <table id="id_list_table">
            <tr><td>1: {{ new_item_text }}</td></tr>  
        </table>
...
```

Para:

```html
...
        <table id="id_list_table">
            {% for item in items %}
                <tr><td>1: {{ item.text }}</td></tr>
            {% endfor %}
        </table>
...
```

Nosso Teste de Unidade continua falhando, conforme esperado.
Devemos implementar a funcionalidade na view.

Modifique a view **lists/views.py** de:

```Python
from django.shortcuts import redirect, render
from lists.models import Item

def home_page(request):
    if request.method == 'POST':
        new_item_text = request.POST['item_text']
        Item.objects.create(text=new_item_text)
        return redirect('/')
    
    return render(request, 'home.html')
```

Para: 

```Python
from django.shortcuts import redirect, render
from lists.models import Item

def home_page(request):
    if request.method == 'POST':
        new_item_text = request.POST['item_text']
        Item.objects.create(text=new_item_text)
        return redirect('/')
    
    items = Item.objects.all()
    return render(request, 'home.html', {'items': items})
```

O Teste de Unidade continua falhando!

Abra o browser em http://localhost:8000 e verifique o que está ocorrendo.

#### COMMIT & PUSH com a mensagem: Apresentar Múltiplos Itens na Listagem

## 18. Criando BD de Produção

O erro em http://localhost:8000 ocorre pois ainda não configuramos o banco de dados de produção corretamente.
Por qual razão algumas operações no BD estavam funcionando no Teste de Unidade? 
Pois o Django cria um *BD especial de testes* para Testes de Unidade (isso é gerenciado pela classe `TestCase`).

Para configurar o BD real (ou seja, de produção), precisamos criá-lo.
Vamos utilizar o SQLite; por default o Django coloca esse BD no arquivo **db.sqlite3** no diretório raiz do projeto.

Já fizemos algumas tarefas anteriormente para configurar o BD: utilizar o **models.py** e criar os arquivos de migração.
A última etapa é de fato criar o BD, através do comando `python manage.py migrate`:

```ShellSession
$ python manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, lists, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying lists.0001_initial... OK
  Applying lists.0002_item_text... OK
  Applying sessions.0001_initial... OK
```

Abra o browser em http://localhost:8000 novamente e verifique que o erro não aparece mais.

Ok, rode o Teste de Unidade novamente, e veja ele passar:

```ShellSession
$ python manage.py test
...
Ran 6 tests in 0.024s
OK
```

Rode o Teste Funcional, e veja que ele vai falhar:

```ShellSession
$ python functional_tests.py
...
AssertionError: '2: Estudar testes de unidade' not found in [
```

Vamos para a última implementação para que o Teste Funcional possa passar.
Altere o template **lists/templates/home.html** de:

```html
...
        <table id="id_list_table">
            {% for item in items %}
                <tr><td>1: {{ item.text }}</td></tr>
            {% endfor %}
        </table>
...
```

Para utilizar `{{ forloop.counter }}`:

```html
...
        <table id="id_list_table">
            {% for item in items %}
                <tr><td>{ forloop.counter }}: {{ item.text }}</td></tr>
            {% endfor %}
        </table>
...
```

Por fim, rode o Teste Funcional novamente com sucesso:

```ShellSession
$ python functional_tests.py 
...
Ran 1 test in 7.057s
OK
```

OBS 1: caso o último teste falhe, tente limpar o BD:

```ShellSession
$ rm db.sqlite3
$ python manage.py migrate --noinput
```

OBS 2: Note que o Teste Funcional ainda não está funcionando corretamente: cada vez que rodamos os testes, estamos deixando dados no BD.
Tente rodar Teste Funcional novamente e veja o problema em http://localhost:8000.
Assunto para as próximas aulas práticas.

#### COMMIT & PUSH com a mensagem: Criando BD de Produção
