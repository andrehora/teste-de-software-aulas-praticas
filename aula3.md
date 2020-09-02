# Teste de Software

## Aula Prática 3: Melhorando Testes Funcionais e Expandindo a Aplicação

Prof. André Hora

Objetivo: Reproduzir uma sessão de desenvolvimento de software usando TDD (Test-Driven Development), com base no livro *Test-Driven-Development with Python*.
Especificamente, iremos reproduzir os exemplos do Cap. 6 e 7 do livro. 
Faça o exercício com o mindset de TDD.
Se necessário, estude antes o material visto em sala de aula.

Instruções:

- Siga o roteiro, passo a passo.
- Após cada passo, rode os testes.
- Nos passos marcados com COMMIT & PUSH, faça um commit e um push no seu repositório. 
Isso será usado no momento da correção, para garantir que seguiu a sequência do roteiro, passo-a-passo.

## 1. Melhoria 1: Isolando Testes Funcionais

Terminamos a última aula prática com os Testes Funcionais e de Unidade passando com sucesso.
No entanto, existe um comportamento errado da aplicação com relação aos Testes Funcionais.
Isto é, cada vez que rodamos os Testes Funcionais, estamos deixando dados no banco de dados, o que pode intereferir na próxima vez rodarmos os testes.
Devemos garantir **isolamento** entre os Testes Funcionais.

Veja que esse problema não ocorre com os Testes de Unidade, pois o framework Django gerencia automaticamente um banco de dados de testes (separado do BD real) para os Testes de Unidade.
No entanto, nossos Testes Funcionais rodam diretamente no BD real, **db.sqlite3**.

Uma solução simples seria limpar o BD através dos métodos `setUp` e `tearDown`.
No entanto, existe uma classe chamada `LiveServerTestCase` que gerencia essa tarefa.
Tal classe cria automaticamente um BD de teste (como nos Testes de Unidade) e inicia um servidor de desenvolvimento para rodar os Testes Funcionais.

Vamos organizar os Testes Funcionais em uma pasta.
Para isso crie uma pasta **functional_tests** com um arquivo chamado **\_\_init\_\_.py** (para ser um pacote válido em Python):

```ShellSession
$ mkdir functional_tests
$ touch functional_tests/__init__.py
```

Mova o Teste Funcional **functional_tests.py** para a pasta **functional_tests** e renomeie o arquivo para **tests.py**. 
Use seguinte comando para notificar o Git:

```ShellSession
$ git mv functional_tests.py functional_tests/tests.py
```

Devemos ter a estrutura:

```
.
├── db.sqlite3
├── functional_tests
│   ├── __init__.py
│   └── tests.py
├── lists
│   ├── admin.py
│   ├── apps.py
│   ├── __init__.py
│   ├── migrations
│   │   ├── 0001_initial.py
│   │   ├── 0002_item_text.py
│   │   ├── __init__.py
│   │   └── __pycache__
│   ├── models.py
│   ├── __pycache__
│   ├── templates
│   │   └── home.html
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

A partir de agora, nosso Teste Funcional é **functional_tests/tests.py**.
Agora, rodamos o Teste Funcional através do comando `python manage.py test functional_tests`.

Altere o Teste Funcional **functional_tests/tests.py** para utilizar a classe `LiveServerTestCase`:

```Python
from django.test import LiveServerTestCase
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
import time

class NewVisitorTest(LiveServerTestCase):

    def setUp(self):
        self.browser = webdriver.Firefox()
    ...
```

Em seguida, remova o endereço hardcoded para visitar o localhost porta 8000.
A classe `LiveServerTestCase` fornece o atributo `live_server_url` para isso:

```Python
...
    def test_can_start_a_list_and_retrieve_it_later(self): 
    
        # Maria decidiu utilizar o novo app TODO. Ela entra em sua página principal:
        self.browser.get(self.live_server_url)
...
```

Podemos também remover o `if __name__ == '__main__'` no final do arquivo **functional_tests/tests.py**, já que o framework Django será o responsável por inicar o teste.

Rode o Teste Funcional através do comando `python manage.py test functional_tests`:

```ShellSession
$ python manage.py test functional_tests
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
.
----------------------------------------------------------------------
Ran 1 test in 8.366s
OK
```

O Teste Funcional continua passando, assim como já estava passando antes da refatoração.
Note também que ao rodar o teste uma segunda vez, os dados não são mais mantidos no BD como anteriormente.
Ou seja, o BD é limpo cada vez que rodamos o teste.
Dessa forma garantimos **isolamento** entre os Testes Funcionais.

#### COMMIT & PUSH com a mensagem: Melhoria 1: Isolando Testes Funcionais

**IMPORTANTE:** agora, o comando `python manage.py test` roda ambos os testes: Funcional + Unidade.

Para rodar os testes separamente, utilize os seguintes comandos:

- Rodar Testes de Unidade: `python manage.py test lists`
- Rodar Testes Funcionais: `python manage.py test functional_tests`

## 2. Melhoria 2: Removendo Sleeps para Evitar Não Determinismo

Note que o Teste Funcional utiliza a função `time.sleep(1)` para esperar a página carregar, também conhecidos como *explicit wait*.

Mas quem garante que 1 segundo é suficiente? Ou seria 1 segundo muito tempo?
Por exemplo, se tivermos centenas de Testes Funcionais, cada um demorando 1 segundo, teriamos uma suite de testes que levaria vários minutos para executar.
Seria 0.1 segundo o bastante? Ou 0.1 segundo não seria suficiente para carregar a página?

De fato, mesmo com 1 segundo, não teremos certeza que página irá carregar, o que pode causar uma falha no teste, ou seja, um falso positivo (leia mais sobre não determinismo em testes [aqui](https://martinfowler.com/articles/nonDeterminism.html)).

Então, vamos substituir os sleeps por uma ferramenta que espera pelo tempo necessário.
Renomeie o método `check_for_row_in_list_table` para `wait_for_row_in_list_table` em **functional_tests/tests.py** e adicione a seguinte lógica nele:

```Python
...
from selenium.common.exceptions import WebDriverException
...
MAX_WAIT = 5
...
    def wait_for_row_in_list_table(self, row_text):
        start_time = time.time()
        while True:  
            try:
                table = self.browser.find_element_by_id('id_list_table')  
                rows = table.find_elements_by_tag_name('tr')
                self.assertIn(row_text, [row.text for row in rows])
                return  
            except (AssertionError, WebDriverException) as e:  
                if time.time() - start_time > MAX_WAIT:  
                    raise e  
                time.sleep(0.5)
...
```

Utilizamos a constante `MAX_WAIT` para definir o limite máximo de tempo para esperar, 5 segundos.
Em seguida temos um loop que termina apenas com duas condições: (1) se o assert passar (`return`) ou (2) se uma exceção for lançada (`raise e`).
Durante a execução do try, se uma exceção for capturada, esperamos 0.5 segundos e iniciamos o loop novamente.
Queremos capturar dois tipos de exceção: `WebDriverException` para quando a página ainda não foi carregada e `AssertionError` para quando a tabela está carregada, mas com dados desatualizados.

Continuando...

Atualize as chamadas de métodos de `check_for_row_in_list_table` para `wait_for_row_in_list_table` em `test_can_start_a_list_and_retrieve_it_later`.
Também remova os `time.sleep(1)` em `test_can_start_a_list_and_retrieve_it_later`.

Rode o Teste Funcional e veja como ele foi executado mais rápido (e mais importante: com menor risco de não determinismo):

```ShellSession
$ python manage.py test functional_tests
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
.
----------------------------------------------------------------------
Ran 1 test in 8.366s
OK
```

#### COMMIT & PUSH com a mensagem: Melhoria 2: Removendo Sleeps para Evitar Não Determinismo

## 3. Trabalhando com Múltiplas Listas & API REST

Note que a nossa aplicação suporta apenas uma lista global.
Ou seja, todos os usuários alteram a mesma lista de tarefas, o que, claro, não é o comportamento esperado.
Vamos alterar nosso design para suportar múltiplas listas.
Isto é, vamos gerar URLs únicas para os usuários de modo que um usuário não veja a lista do outro, conforme já especificado em nosso Teste Funcional:

```Python
...
        # Maria se pergunta se o site vai lembrar da sua lista. Então, ela verifica que
        # o site gerou uma URL única para ela -- existe uma explicação sobre essa feature

        # Ela visita a URL: a sua lista TODO ainda está armazenada

        # Satisfeita, ela vai dormir
...        
```

Em suma:

- Queremos que cada usuário armazene sua própria lista
- Uma lista contém vários itens; cada item contém uma descrição textual
- Precisamos salvar as listas para os usuários poderem alterá-las depois. Por enquanto, podemos gerar uma URL única, para cada usuário, contendo a lista. Futuramente, podemos melhorar esse processo.

Iremos criar uma API REST para facilitar o uso da nossa aplicação.
REST sugere que tenhamos uma estrutura URL que corresponda a nossa estrutura de dados, isto é, lista e items.

Para criar uma nova lista, podemos ter a seguinte URL:

```
/lists/new
```

Cada lista deve ter sua própria URL:

```
/lists/<list identifier>/
```

Para adicionar um item em uma lista existente, temos outra URL:

```
/lists/<list identifier>/add_item
```

Vamos expandir nosso Teste Funcional **functional_tests/tests.py**.

Primeiro, renomei o método `test_can_start_a_list_and_retrieve_it_later` para `test_can_start_a_list_for_one_user`:

```Python
def test_can_start_a_list_for_one_user(self):

        # Maria decidiu utilizar o novo app TODO. Ela entra em sua página principal:
        self.browser.get(self.live_server_url)
        ...
```

Vamos agora testar que múltiplos usuários podem utilizar nossa aplicação.
Queremos verificar que um segundo usuário (João) não irá ver a lista do primeiro usuário (Maria).
Em **functional_tests/tests.py**, adicione o seguinte método de teste `test_multiple_users_can_start_lists_at_different_urls`:

```Python
    def test_multiple_users_can_start_lists_at_different_urls(self):
        # Maria começa uma nova lista
        self.browser.get(self.live_server_url)
        inputbox = self.browser.find_element_by_id('id_new_item')
        inputbox.send_keys('Estudar testes funcionais')
        inputbox.send_keys(Keys.ENTER)
        self.wait_for_row_in_list_table('1: Estudar testes funcionais')

        # Ela nota que sua lista possui uma URL única
        maria_list_url = self.browser.current_url
        self.assertRegex(maria_list_url, '/lists/.+')

        # Agora, um novo usuário, João, entra no site
        self.browser.quit()
        self.browser = webdriver.Firefox()

        # João visita a página inicial. Não existe nenhum sinal da lista de Maria
        self.browser.get(self.live_server_url)
        page_text = self.browser.find_element_by_tag_name('body').text
        self.assertNotIn('1: Estudar testes funcionais', page_text)
        self.assertNotIn('2: Estudar testes de unidade', page_text)

        # João inicia uma nova lista
        inputbox = self.browser.find_element_by_id('id_new_item')
        inputbox.send_keys('Comprar leite')
        inputbox.send_keys(Keys.ENTER)
        self.wait_for_row_in_list_table('1: Comprar leite')

        # João pega sua URL única
        joao_list_url = self.browser.current_url
        self.assertRegex(joao_list_url, '/lists/.+')
        self.assertNotEqual(joao_list_url, maria_list_url)

        # Novamente, não existe sinal da lista de Maria
        page_text = self.browser.find_element_by_tag_name('body').text
        self.assertNotIn('Estudar testes funcionais', page_text)
        self.assertIn('Comprar leite', page_text)

        # Satisfeitos, ambos vão dormir
```

Note o uso de `assertRegex`, que verifica se uma string corresponde a uma expressão regular.

Rode o Teste Funcional e veja a falha esperada:

```ShellSession
$ python manage.py test functional_tests
...
    self.assertRegex(maria_list_url, '/lists/.+')
AssertionError: Regex didn't match: '/lists/.+' not found in 'http://localhost:58214/'
----------------------------------------------------------------------
Ran 2 tests in 13.757s
FAILED (failures=1)
```

#### COMMIT & PUSH com a mensagem: Trabalhando com Múltiplas Listas & API REST

## 4. Implementando o Novo Projeto (Listas e Items) - PARTE 1

Vamos iniciar de fato a implementação do novo projeto da nossa aplicação, isto é, Listas possuem URLs únicas e são formadas por Itens.

Como sempre no TDD, iniciamos o desenvolvimento pelos Testes de Unidade.
Altere o método `test_redirects_after_POST` no Teste de Unidade **lists/tests.py** de:

```Python
    def test_redirects_after_POST(self):
        response = self.client.post('/', data={'item_text': 'A new list item'})

        self.assertEqual(response.status_code, 302)
        self.assertEqual(response['location'], '/')
```

Para:

```Python
    def test_redirects_after_POST(self):
        response = self.client.post('/', data={'item_text': 'A new list item'})

        self.assertEqual(response.status_code, 302)
        self.assertEqual(response['location'], '/lists/the-only-list-in-the-world/')
```

Para entender o teste acima pense o seguinte: atualmente, não existe nada implementado em nossa aplicação para lidar com URLs únicas.
O teste acima é o primeiro (e pequeno) passo para implementarmos essa funcionalidade.
Em outras palavras: quando o teste acima passar, estaremos mais próximo da solução esperada.

Rode o Teste de Unidade e veja a falha esperada:

```ShellSession
$ python manage.py test lists
...
AssertionError: '/' != '/lists/the-only-list-in-the-world/'
```

Vamos ajustar a view `home_page` em **lists/views.py**:

```Python
from django.shortcuts import redirect, render
from lists.models import Item

def home_page(request):
    if request.method == 'POST':
        new_item_text = request.POST['item_text']
        Item.objects.create(text=new_item_text)
        return redirect('/lists/the-only-list-in-the-world/')

    items = Item.objects.all()
    return render(request, 'home.html', {'items': items})

```

Tente rodar o Teste Funcional e você verá que ele foi totalmente quebrado.
Ou seja, uma regressão foi introduzida na nossa aplicação, pois a URL `'/lists/the-only-list-in-the-world/'` ainda não existe.
Tentaremos voltar para um estado estável da aplicação o quanto antes.

#### COMMIT & PUSH com a mensagem: Implementando o Novo Projeto (Listas e Items) - PARTE 1

## 5. Implementando o Novo Projeto (Listas e Items) - PARTE 2

Vamos voltar para os Testes de Unidade.
Adicione uma classe `ListViewTest` em **lists/tests.py**, conforme segue:

```Python
...
class ListViewTest(TestCase):

    def test_displays_all_items(self):
        Item.objects.create(text='itemey 1')
        Item.objects.create(text='itemey 2')

        response = self.client.get('/lists/the-only-list-in-the-world/')

        self.assertContains(response, 'itemey 1')
        self.assertContains(response, 'itemey 2')
```

Note que estamos utilizando `self.assertContains` (ao invés de `self.assertIn`), um assert "mais inteligente" que sabe lidar com respostas e bytes.

Rode o Teste de Unidade:

```ShellSession
$ python manage.py test lists
...
self.assertContains(response, 'itemey 1')
...
AssertionError: 404 != 200 : Couldn't retrieve content: Response code was 404 (expected 200)
```

A mensagem (`404 != 200`) indica que o teste está falhando pois a nova URL ainda não existe, logo o erro [HTTP 404](https://pt.wikipedia.org/wiki/HTTP_404) é retornado.

Vamos adicionar a nova URL.
Abra o arquivo **superlists/urls.py** e altere `urlpatterns`, conforme segue:

```Python
urlpatterns = [
    url(r'^$', views.home_page, name='home'),
    url(r'^lists/the-only-list-in-the-world/$', views.view_list, name='view_list'),
]
```

Rode o Teste de Unidade:

```ShellSession
$ python manage.py test lists
...
AttributeError: module 'lists.views' has no attribute 'view_list'
```

O erro é autoexplicativo.
Crie a nova view `view_list` em **lists/views.py**:

```Python
def view_list(request):
    pass
```

Rode o Teste de Unidade novamente:

```ShellSession
$ python manage.py test lists
...
ValueError: The view lists.views.view_list didn't return an HttpResponse object. It returned None instead.
```

Atualize a view `view_list` em **lists/views.py** para:

```Python
def view_list(request):
    items = Item.objects.all()
    return render(request, 'home.html', {'items': items})
```

Rode o Teste de Unidade e veja ele passar com sucesso:

```ShellSession
$ python manage.py test lists
...
Ran 7 tests in 0.029s
OK
```

Agora vamos voltar para o Teste Funcional.
Rode o Teste Funcional:

```ShellSession
$ python manage.py test functional_tests
...
AssertionError: '2: Estudar testes de unidade' not found in ['1: Estudar testes funcionais']
...
AssertionError: '1: Estudar testes funcionais' unexpectedly found in 'Your To-Do list\n1: Estudar testes funcionais'
...
```

Parece que a situação melhorou um pouco (em relação a última vez que rodamos o Teste Funcional), mas o teste ainda está falhando.
Note que o problema ocorre exatamente ao tentar inserir o segundo item na lista, logo, o primeiro item está sendo adicionado com sucesso.

**IMPORTANTE:** nosso Teste de Unidade está passando, mas o Teste Funcional não está passando.
Isso indica que algum problema importante não está sendo coberto pelo Teste de Unidade.

A resposta é que nosso template **home.html** atualmente não especifica a URL no POST. Atualize a tag `<form>` em **lists/templates/home.html** de:

```html
<form method="POST">
```

Para:

```html
<form method="POST" action="/">
```

Rode o Teste Funcional novamente:

```ShellSession
$ python manage.py test functional_tests
...
test_multiple_users_can_start_lists_at_different_urls
    self.assertNotIn('1: Estudar testes funcionais', page_text)
AssertionError: '1: Estudar testes funcionais' unexpectedly found in 'Your To-Do list\n1: Estudar testes funcionais'

----------------------------------------------------------------------
Ran 2 tests in 22.790s

FAILED (failures=1)
```

Note que o primeiro teste (isto é, `test_can_start_a_list_and_retrieve_it_later`) voltou a passar com sucesso.
Ou seja, removemos a regressão que foi adicionada na etapa anterior.
No entanto, o novo teste (isto é, `test_multiple_users_can_start_lists_at_different_urls`) está falhando.
Isso é esperado, pois a implementação da nova funcionalidade ainda não foi finalizada.

Por fim, podemos remover o método de teste de unidade `test_displays_all_list_items`, pois ele não é mais necessário.
Logo, ao rodar nosso Teste de Unidade, teremos 6 testes, ao invés de 7:

```ShellSession
$ python manage.py test lists
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
......
----------------------------------------------------------------------
Ran 6 tests in 0.064s
OK
```

#### COMMIT & PUSH com a mensagem: Implementando o Novo Projeto (Listas e Items) - PARTE 2

## 6. Implementando o Novo Projeto (Listas e Items) - PARTE 3

Atualmente temos apenas um template **home.html** para view `home_page`.
Nesta seção, vamos criar um novo template **list.html** a view `view_list`.
Como sempre, começamos pelos testes e com pequenos passos.

No Teste de Unidade **lists/tests.py**, adicione o método `test_uses_list_template` na classe `ListViewTest` para garantir que o template correto será sempre utizado:

```Python
...
class ListViewTest(TestCase):

    def test_uses_list_template(self):
        response = self.client.get('/lists/the-only-list-in-the-world/')
        self.assertTemplateUsed(response, 'list.html')

    def test_displays_all_items(self):
        ...
```

Rode o Teste de Unidade:

```ShellSession
$ python manage.py test lists
...
AssertionError: False is not true : Template 'list.html' was not a template used to render the response. Actual template(s) used: home.html
```

Agora atualize a view `view_list` em **lists/views.py** de:

```Python
def view_list(request):
    items = Item.objects.all()
    return render(request, 'home.html', {'items': items})
```

Para:

```Python
def view_list(request):
    items = Item.objects.all()
    return render(request, 'list.html', {'items': items})
```

Rode o Teste de Unidade:

```ShellSession
$ python manage.py test lists
...
django.template.exceptions.TemplateDoesNotExist: list.html
```

Agora, crie o novo template **list.html** vazio, com comando:

```ShellSession
$ touch lists/templates/list.html
```

Rode o Teste de Unidade para ver, finalmente, o teste falhando:

```ShellSession
$ python manage.py test lists
...
AssertionError: False is not true : Couldn't find 'itemey 1' in response
```

Para implementar o template list, vamos reaproveitar o template home.
Então, copie o template **home.html** para **list.html**:

```ShellSession
$ cp lists/templates/home.html lists/templates/list.html
```

Rode o Teste de Unidade e veja ele passar:

```ShellSession
$ python manage.py test lists
Ran 7 tests in 0.069s
OK
```

Como agora temos dois template (home.html e list.html), podemos simplificar home.html.
Logo, altere o template **lists/templates/home.html** para:

```html
<html>
    <head>
        <title>To-Do lists</title>
    </head>
    <body>
      <h1>Start a new To-Do list</h1>
      <form method="POST">
        <input name="item_text" id="id_new_item" placeholder="Enter a to-do item" />
        {% csrf_token %}
      </form>
    </body>
</html>
```

Rode o Teste de Unidade novamente para garantir que nada foi quebrado:

```ShellSession
$ python manage.py test lists
Ran 7 tests in 0.069s
OK
```

Em seguida, simplifique o código a view `home_page` em **lists/views.py**:

```Python
def home_page(request):
    if request.method == 'POST':
        new_item_text = request.POST['item_text']
        Item.objects.create(text=new_item_text)
        return redirect('/lists/the-only-list-in-the-world/')

    return render(request, 'home.html')
```

Rode o Teste de Unidade novamente e veja que nada foi quebrado:

```ShellSession
$ python manage.py test lists
Ran 7 tests in 0.067s
OK
```

Agora rode o Teste Funcional:

```ShellSession
$ python manage.py test functional_tests
...
FAIL: test_multiple_users_can_start_lists_at_different_urls (functional_tests.tests.NewVisitorTest)
...
AssertionError: '1: Comprar leite' not found in ['1: Estudar testes funcionais', '2: Comprar leite']
```

Veja que nenhuma regressão foi inserida na aplicação, ou seja, o teste antigo `test_can_start_a_list_and_retrieve_it_later` continua passando.
O teste que está falhando é o novo teste que verifica múltiplos usuários, isto é, `test_multiple_users_can_start_lists_at_different_urls`.
Ou seja, apesar do novo teste não estar passando, garantimos que nenhuma regressão foi inserida na aplicação.

#### COMMIT & PUSH com a mensagem: Implementando o Novo Projeto (Listas e Items) - PARTE 3

## 6. Implementando o Novo Projeto (Listas e Items) - PARTE 4

Vamos trabalhar um pouco na API REST `/lists/new`.

Mova os métodos `test_can_save_a_POST_request` e `test_redirects_after_POST` para uma nova classe `NewListTest` e altere o código deles conforme segue em **lists/tests.py**:

```Python
class NewListTest(TestCase):

    def test_can_save_a_POST_request(self):
        self.client.post('/lists/new', data={'item_text': 'A new list item'})
        self.assertEqual(Item.objects.count(), 1)
        new_item = Item.objects.first()
        self.assertEqual(new_item.text, 'A new list item')

    def test_redirects_after_POST(self):
        response = self.client.post('/lists/new', data={'item_text': 'A new list item'})
        self.assertRedirects(response, '/lists/the-only-list-in-the-world/')
```

Rode o Teste de Unidade e veja a falha esperada:

```ShellSession
$ python manage.py test lists
...
self.assertEqual(Item.objects.count(), 1)
AssertionError: 0 != 1
...
self.assertRedirects(response, '/lists/the-only-list-in-the-world/')
...
AssertionError: 404 != 302 : Response didn't redirect as expected: Response code was 404 (expected 302)
...
```

Precisamos atualizar o arquivo **superlists/urls.py** com a nova URL `lists/new`:

```
urlpatterns = [
    url(r'^$', views.home_page, name='home'),
    url(r'^lists/new$', views.new_list, name='new_list'),
    url(r'^lists/the-only-list-in-the-world/$', views.view_list, name='view_list'),
]
```

Crie também a nova view `new_list` em **lists/views.py**:

```
def new_list(request):
    Item.objects.create(text=request.POST['item_text'])
    return redirect('/lists/the-only-list-in-the-world/')
```

Rode o Teste de Unidade e veja ele passar:

```ShellSession
$ python manage.py test lists
...
Ran 7 tests in 0.030s
OK
```

#### COMMIT & PUSH com a mensagem: Implementando o Novo Projeto (Listas e Items) - PARTE 4

## 7. Implementando o Novo Projeto (Listas e Items) - PARTE 5

Note as que novas views `view_list` e `new_list` estão fazendo boa parte do trabalho que a view `home_page` estava fazendo orginalmente, isto é, listar e criar tarefas.
Desse modo, podemos simplificar bastante a nossa primeira view `home_page`.

Remova o teste `test_only_saves_items_when_necessary` em `HomePageTest`.
Além disso, abra o arquivo **lists/views.py** e altere o código de `home_page` para:

```Python
def home_page(request):
    return render(request, 'home.html')
```

Rode o Teste de Unidade:

```ShellSession
$ python manage.py test lists
...
Ran 6 tests in 0.030s
OK
```

Agora, rode o Teste Funcional:

```ShellSession
$ python manage.py test functional_tests
...
ERROR: test_can_start_a_list_and_retrieve_it_later (functional_tests.tests.NewVisitorTest)
...
ERROR: test_multiple_users_can_start_lists_at_different_urls (functional_tests.tests.NewVisitorTest)
...
```

Temos uma regressão em `test_can_start_a_list_and_retrieve_it_later`.
Isso ocorreu pois esquecemos de atualizar as URLs do POSTs nos templates.

Abra os arquivos **lists/templates/home.html** e **lists/templates/list.html**, e altere em ambos de:

```html
<form method="POST" action="/">
```

Para:

```html
<form method="POST" action="/lists/new">
```

Rode o Teste Funcional novamente e teremos apenas a nossa falha esperada:

```ShellSession
...
FAIL: test_multiple_users_can_start_lists_at_different_urls (functional_tests.tests.NewVisitorTest)
```

#### COMMIT & PUSH com a mensagem: Implementando o Novo Projeto (Listas e Items) - PARTE 5

## 8. Implementando o Novo Projeto: Ajustando o Modelo

Atualmente, nosso modelo contém apenas a classe `Item`.
Vamos adicionar mais uma classe no modelo, `List`, para representar a lista de tarefas de um usuário.

Iniciando pelos Testes de Unidade **lists/tests.py**.
Renomeie a classe `ItemModelTest` para `ListAndItemModelsTest`, e altere seu código conforme segue:

```Python
from django.test import TestCase
from lists.models import Item, List

...

class ListAndItemModelsTest(TestCase):

    def test_saving_and_retrieving_items(self):
        my_list = List()
        my_list.save()

        first_item = Item()
        first_item.text = 'O primeiro item'
        first_item.list = my_list
        first_item.save()

        second_item = Item()
        second_item.text = 'O segundo item'
        second_item.list = my_list
        second_item.save()

        saved_list = List.objects.first()
        self.assertEqual(saved_list, my_list)

        saved_items = Item.objects.all()
        self.assertEqual(saved_items.count(), 2)

        first_saved_item = saved_items[0]
        second_saved_item = saved_items[1]
        self.assertEqual(first_saved_item.text, 'O primeiro item')
        self.assertEqual(first_saved_item.list, my_list)
        self.assertEqual(second_saved_item.text, 'O segundo item')
        self.assertEqual(second_saved_item.list, my_list)
```

Note que estamos criando um novo objeto `List` com uma propriedade `.list`.
Além disso, estamos verificando (1) que `my_list` é salvo corretamente e (2) que os dois itens estão relacionados com `my_list`.

Rode o Teste de Unidade e tente implementar o código mínimo da aplicação para suprimir cada mensagem de erro:

```ShellSession
$ python manage.py test lists
...
ImportError: cannot import name 'List'
```

Para resolver o erro acima, você deve criar a classe `List` em **lists/models.py**:

```Python
class List(models.Model):
    pass
```

Rode o Teste de Unidade novamente:

```ShellSession
$ python manage.py test lists
...
django.db.utils.OperationalError: no such table: lists_list
```

Rode o `makemigrations`:

```ShellSession
$ python manage.py makemigrations
Migrations for 'lists':
  lists/migrations/0003_list.py
    - Create model List
```

Rode o Teste de Unidade novamente:

```ShellSession
$ python manage.py test lists
...
AttributeError: 'Item' object has no attribute 'list'
```

Em **lists/models.py**, adicione o atributo `list` na classe `Item` para representar uma chave estrageira:

```Python
from django.db import models

class List(models.Model):
    pass

class Item(models.Model):
    text = models.TextField(default='')
    list = models.ForeignKey(List, default=None)
```

Rode o `makemigrations` novamente:

```ShellSession
$ python manage.py makemigrations
Migrations for 'lists':
  lists/migrations/0004_item_list.py
    - Add field list to item
```

#### COMMIT & PUSH com a mensagem: Implementando o Novo Projeto: Ajustando o Modelo

## 8. Implementando o Novo Projeto: Ajustando o Resto do Sistema

Rode o Teste de Unidade:

```ShellSession
$ python manage.py test lists
...
django.db.utils.IntegrityError: NOT NULL constraint failed: lists_item.list_id
```

Note que os testes de algumas views estão falhando.
Ocorre que precisamos atualizar os testes a aplicação para lidar com o nosso novo modelo, que agora contém `List` e `Item`.

No Teste de Unidade **lists/tests.py**, atualize o método `test_displays_all_items` na classe `ListViewTest` de:

```Python
    ...
    def test_displays_all_items(self):
        Item.objects.create(text='itemey 1')
        Item.objects.create(text='itemey 2')
        ...
```

Para:

```Python
    ...
    def test_displays_all_items(self):
        my_list = List.objects.create()
        Item.objects.create(text='itemey 1', list=my_list)
        Item.objects.create(text='itemey 2', list=my_list)
        ...
```

Atualize também a view `new_list` em **lists/views.py** de:

```Python
from lists.models import Item
...
def new_list(request):
    Item.objects.create(text=request.POST['item_text'])
    return redirect('/lists/the-only-list-in-the-world/')
```

Para:

```Python
from lists.models import Item, List
...
def new_list(request):
    my_list = List.objects.create()
    Item.objects.create(text=request.POST['item_text'], list=my_list)
    return redirect('/lists/the-only-list-in-the-world/')
```

Rode o Teste de Unidade novamente, com sucesso:

```ShellSession
$ python manage.py test lists
...
Ran 6 tests in 0.039s
OK
```

#### COMMIT & PUSH com a mensagem: Implementando o Novo Projeto: Ajustando o Resto do Sistema

## 9. Implementando o Novo Projeto: Cada Lista Deve Ter Uma URL

Estamos chegando na parte final do nosso novo projeto: cada lista deve ter uma URL única.
Talvez a solução mais simples seja gerar um `id` para cada lista.

Atualize o teste `ListViewTest` conforme segue, renomeando também o método `test_displays_all_items` para `test_displays_only_items_for_that_list`:

```Python
class ListViewTest(TestCase):

    def test_uses_list_template(self):
        my_list = List.objects.create()
        response = self.client.get(f'/lists/{my_list.id}/')
        self.assertTemplateUsed(response, 'list.html')

    def test_displays_only_items_for_that_list(self):
        correct_list = List.objects.create()
        Item.objects.create(text='itemey 1', list=correct_list)
        Item.objects.create(text='itemey 2', list=correct_list)
        other_list = List.objects.create()
        Item.objects.create(text='other list item 1', list=other_list)
        Item.objects.create(text='other list item 2', list=other_list)

        response = self.client.get(f'/lists/{correct_list.id}/')

        self.assertContains(response, 'itemey 1')
        self.assertContains(response, 'itemey 2')
        self.assertNotContains(response, 'other list item 1')
        self.assertNotContains(response, 'other list item 2')
```

Rode o Teste de Unidade e veja a falha esperada:

```ShellSession
$ python manage.py test lists
...
AssertionError: 404 != 200 : Couldn't retrieve content: Response code was 404 (expected 200)
...
AssertionError: No templates used to render the response
```

Vamos implementar a funcionalidade.
Primeiro atualize o arquivo **superlists/urls.py** de:

```Python
urlpatterns = [
    url(r'^$', views.home_page, name='home'),
    url(r'^lists/new$', views.new_list, name='new_list'),
    url(r'^lists/the-only-list-in-the-world/$', views.view_list, name='view_list'),
]
```

Para:

```Python
urlpatterns = [
    url(r'^$', views.home_page, name='home'),
    url(r'^lists/new$', views.new_list, name='new_list'),
    url(r'^lists/(\d+)/$', views.view_list, name='view_list'),
]
```

`(\d+)` indica que o inteiro capturado nessa URL será passado como um argumento para a view `view_list`.
Por exemplo, se o usuário digitar a URL `/lists/123/`, a view `view_list` receberá como segundo argumento `123`.
Se o usuário digitar a URL `/lists/9/`, a view `view_list` receberá como segundo argumento `9`.

Rode o Teste de Unidade e veja a falha esperada:

```ShellSession
$ python manage.py test lists
...
TypeError: view_list() takes 1 positional argument but 2 were given
```

De fato, a view `view_list` possui atualmente apenas um parâmetro, `request`.
Em **lists/views.py**, atualize a view `view_list` para incluir o segundo parâmetro `list_id` e utilizar `List`:

```Python
def view_list(request, list_id):
    my_list = List.objects.get(id=list_id)
    items = Item.objects.filter(list=my_list)
    return render(request, 'list.html', {'items': items})
```

Rode o Teste de Unidade e veja uma nova falha, agora no teste `test_redirects_after_POST` em `NewListTest`:

```ShellSession
$ python manage.py test lists
...
ERROR: test_redirects_after_POST (lists.tests.NewListTest)
...
ValueError: invalid literal for int() with base 10: 'the-only-list-in-the-world'
```

Abra o Teste de Unidade **lists/tests.py** e atualize `test_redirects_after_POST` em `NewListTest`:

```Python
    def test_redirects_after_POST(self):
        response = self.client.post('/lists/new', data={'item_text': 'A new list item'})
        new_list = List.objects.first()
        self.assertRedirects(response, f'/lists/{new_list.id}/')
```

Atualize também a view `new_list` em **lists/views.py**:

```Python
def new_list(request):
    my_list = List.objects.create()
    Item.objects.create(text=request.POST['item_text'], list=my_list)
    return redirect(f'/lists/{my_list.id}/')
```

Rode o Teste de Unidade com sucesso:

```ShellSession
$ python manage.py test lists
...
Ran 6 tests in 0.044s
OK
```

#### COMMIT & PUSH com a mensagem: Implementando o Novo Projeto: Cada Lista Deve Ter Uma URL

## 10. Implementando o Novo Projeto: View para Adicionar Items em uma Lista Existente


Rode o Teste Funcional, e você verá que não é possível adicionar um segundo item em uma lista existente:

```ShellSession
$ python manage.py test functional_tests
...
AssertionError: '2: Estudar testes de unidade' not found in ['1: Estudar testes de unidade']
```

Precisamos implementar nossa última API REST para adicionar itens em uma lista existente: 

`/lists/<list identifier>/add_item`.

Em **lists/tests.py**, adicione a seguinte classe de teste:

```Python
class NewItemTest(TestCase):

    def test_can_save_a_POST_request_to_an_existing_list(self):
        other_list = List.objects.create()
        correct_list = List.objects.create()

        self.client.post(
            f'/lists/{correct_list.id}/add_item',
            data={'item_text': 'A new item for an existing list'}
        )

        self.assertEqual(Item.objects.count(), 1)
        new_item = Item.objects.first()
        self.assertEqual(new_item.text, 'A new item for an existing list')
        self.assertEqual(new_item.list, correct_list)


    def test_redirects_to_list_view(self):
        other_list = List.objects.create()
        correct_list = List.objects.create()

        response = self.client.post(
            f'/lists/{correct_list.id}/add_item',
            data={'item_text': 'A new item for an existing list'}
        )

        self.assertRedirects(response, f'/lists/{correct_list.id}/')
```

Rode o Teste de Unidade e veja a falha esperada (pois a URL `/lists/<list identifier>/add_item` ainda não existe):

```ShellSession
$ python manage.py test lists
...
AssertionError: 404 != 302 : Response didn't redirect as expected: Response code was 404 (expected 302)
```

Atualize o arquivo **superlists/urls.py** para incluir a nova URL:

```Python
urlpatterns = [
    url(r'^$', views.home_page, name='home'),
    url(r'^lists/new$', views.new_list, name='new_list'),
    url(r'^lists/(\d+)/$', views.view_list, name='view_list'),
    url(r'^lists/(\d+)/add_item$', views.add_item, name='add_item'),
]
```

Rode o Teste de Unidade:

```ShellSession
$ python manage.py test lists
AttributeError: module 'lists.views' has no attribute 'add_item'
```

Adicione a view `add_item` em **lists/views.py**:

```Python
def add_item(request):
    pass
```

Rode o Teste de Unidade:

```ShellSession
$ python manage.py test lists
TypeError: add_item() takes 1 positional argument but 2 were given
```

Atualize a view `add_item` em **lists/views.py**:

```
def add_item(request, list_id):
    pass
```

Rode o Teste de Unidade novamente:

```ShellSession
$ python manage.py test lists
ValueError: The view lists.views.add_item didn't return an HttpResponse object. It returned None instead.
```

Modifique a view `add_item` em **lists/views.py**:

```Python
def add_item(request, list_id):
    list_ = List.objects.get(id=list_id)
    Item.objects.create(text=request.POST['item_text'], list=list_)
    return redirect(f'/lists/{list_.id}/')
```

Rode o Teste de Unidade, com sucesso.
A nova view `add_item` está funcionando!

```ShellSession
$ python manage.py test lists
...
Ran 8 tests in 0.055s
OK
```

#### COMMIT & PUSH com a mensagem: Implementando o Novo Projeto: View para Adicionar Items em uma Lista Existente

## 11. Novo Projeto: Atualizando o Template

Vamos agora atualizar o template para utilizar a nova view.
Desse modo, modifique o template list em **lists/templates/list.html** para:

```html
...
<form method="POST" action="/lists/{{ list.id }}/add_item">
...
{% for item in list.item_set.all %}
...
```

Note que para a modificação acima funcionar, a view deverá passar uma `List` para o template.

Adicione o seguinte Teste de Unidade na classe `ListViewTest` em **lists/tests.py**:

```Python
    def test_passes_correct_list_to_template(self):
        other_list = List.objects.create()
        correct_list = List.objects.create()
        response = self.client.get(f'/lists/{correct_list.id}/')
        self.assertEqual(response.context['list'], correct_list)
```

Por fim, em **lists/views.py**, faça com que a view `view_list` passe uma `List`.
Logo, altere `view_list` de:

```Python
def view_list(request, list_id):
    my_list = List.objects.get(id=list_id)
    items = Item.objects.filter(list=my_list)
    return render(request, 'list.html', {'items': items})
```

Para:

```Python
def view_list(request, list_id):
    my_list = List.objects.get(id=list_id)
    return render(request, 'list.html', {'list': my_list})
```

Rode o Teste de Unidade com sucesso:

```ShellSession
$ python manage.py test lists
...
Ran 9 tests in 0.066s
OK
```

Rode também o Teste Funcional com sucesso:

```ShellSession
$ python manage.py test functional_tests
...
Ran 2 tests in 25.576s
OK
```

Teste a aplicação em http://localhost:8000.
Caso obtenha o erro `no such table: lists_list`:

- Delete o arquivo `db.sqlite3`
- Rode o comando `python manage.py migrate` para que o banco de dados seja atualizado de acordo com novo esquema (List, Item, etc)
- Teste novamente a aplicação em http://localhost:8000

#### COMMIT & PUSH com a mensagem: Novo Projeto: Atualizando o Template


