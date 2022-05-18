# Teste de Software

## Aula Prática 4: Melhorando Layout e Implementando Validação

Prof. André Hora

Objetivo: Reproduzir uma sessão de desenvolvimento de software usando TDD (Test-Driven Development), com base no livro *Test-Driven-Development with Python*.
Especificamente, iremos reproduzir os exemplos dos capítulos 8, 12 e 13.
Faça o exercício com o mindset de TDD.
Se necessário, estude antes o material visto em sala de aula.

Instruções:

- Siga o roteiro, passo a passo.
- Após cada passo, rode os testes.
- Nos passos marcados com COMMIT & PUSH, faça um commit e um push no seu repositório. 
Isso será usado no momento da correção, para garantir que seguiu a sequência do roteiro, passo a passo.

## 1. Layout e o estilo da aplicação

Ao final da aula 3, concluímos as funcionalidade básicas da nossa aplicação TODO. Criamos três APIs REST: `/lists/new`, `/lists/<list identifier>/` e `/lists/<list identifier>/add_item` e os seus testes.

Teste a aplicação em: http://localhost:8000. Lembre-se de subir o servidor através do comando `python manage.py runserver`. OBS: caso tenha algum problema no BD, delete o arquivo `db.sqlite3` e rode o comando `python manage.py migrate` para que o BD seja atualizado.

Explore o website: entre na página inicial e crie uma lista; entre novamente na página principal e crie outra lista; volte para primeira lista e verifique que os itens estão sendo salvos. Tudo deve estar funcionando corretamente!

Note que nossa página inicial está muito simples (sem layout e sem estilo). Devemos trabalhar um pouco na estética na aplicação, com o uso de CSS.

**Devemos testar detalhes de layout e estilo?**

Podemos testar a estética da nossa aplicação: apenas o básico para garantir que o layout da aplicação está ok, como um **teste de fumaça** ([Smoke Test](https://en.wikipedia.org/wiki/Smoke_testing_(software))).
Por exemplo, o CSS foi carregado? a caixa de entrada está alinhada? Isso nos dará confiança que pelo menos o básico da estética estará funcionando!

Crie o seguinte método de teste na classe `NewVisitorTest` em **functional_tests/tests.py**:

```python
    def test_layout_and_styling(self):
        # Edith entra na home page
        self.browser.get(self.live_server_url)
        self.browser.set_window_size(1024, 768)

        # Ela nota que o input box está centralizado
        inputbox = self.browser.find_element_by_id('id_new_item')
        self.assertAlmostEqual(
            inputbox.location['x'] + inputbox.size['width'] / 2,
            512,
            delta=10
        )

        # Ela inicia uma nova lista e nota que o input
        # também está centralizado
        inputbox.send_keys('testing')
        inputbox.send_keys(Keys.ENTER)
        self.wait_for_row_in_list_table('1: testing')
        inputbox = self.browser.find_element_by_id('id_new_item')
        self.assertAlmostEqual(
            inputbox.location['x'] + inputbox.size['width'] / 2,
            512,
            delta=10
        )
```

Basicamente, esse teste verifica alguns detalhes básicos de estética.
Rode o teste funcional e veja falha esperada; algo semelhante a:

```sh
$ python manage.py test functional_tests
...
AssertionError: 80.33333587646484 != 512 within 10 delta (431.66666412353516 difference)
...
```

#### COMMIT & PUSH com a mensagem: Layout e o estilo da aplicação

## 2. Bootstrap + base template

Iremos utilizar o [Bootstrap](https://getbootstrap.com) para melhorar a estética da aplicação:

- Baixe o seguinte arquivo: https://github.com/twbs/bootstrap/releases/download/v3.3.4/bootstrap-3.3.4-dist.zip
- Descompacte o arquivo baixado
- Renomeie a pasta `bootstrap-3.3.4-dist` para `bootstrap`
- Crie uma pasta `lists/static` dentro da aplicação
- Mova a pasta `bootstrap` para `lists/static` (gerando `lists/static/bootstrap`)

Antes de utilizar o Bootstrap, iremos realizar algumas alterações no nosso código HTML: **home.html** e **list.html**.

Vamos criar um template base para reutilizar código HTML e facilitar o uso do Bootstrap.
Para isso, crie um arquivo **base.html** em **lists/templates**, com o seguinte código:

```html
<html>
  <head>
    <title>To-Do lists</title>
  </head>

  <body>
    <h1>{% block header_text %}{% endblock %}</h1>
    <form method="POST" action="{% block form_action %}{% endblock %}">
      <input name="item_text" id="id_new_item" placeholder="Enter a to-do item" />
      {% csrf_token %}
    </form>
    {% block table %}
    {% endblock %}
  </body>
</html>
```

O template base pode ser visto como uma superclasse, onde outros templates herdam dele.
Observe o código acima e note os comandos `block`: eles representam pontos de extensão do template base, ou seja, outros templates podem estender esse template e adicionar seu próprio conteúdo.

Atualize **lists/templates/home.html** para: 

```html
{% extends 'base.html' %}

{% block header_text %}Start a new To-Do list{% endblock %}

{% block form_action %}/lists/new{% endblock %}
```

Atualize também **lists/templates/list.html** para:

```html
{% extends 'base.html' %}

{% block header_text %}Your To-Do list{% endblock %}

{% block form_action %}/lists/{{ list.id }}/add_item{% endblock %}

{% block table %}
  <table id="id_list_table">
    {% for item in list.item_set.all %}
      <tr><td>{{ forloop.counter }}: {{ item.text }}</td></tr>
    {% endfor %}
  </table>
{% endblock %}
```

Note que essa alteração nos HTMLs não deve alterar parte visual no nosso website.
Essa alteração apenas deixou o uso do HTML mais eficiente!

Rode o teste funcional novamente para verificar que ainda estamos com o mesmo erro anterior:

```sh
$ python manage.py test functional_tests
...
AssertionError: 80.33333587646484 != 512 within 10 delta (431.66666412353516 difference)
...
```

#### COMMIT & PUSH com a mensagem: Bootstrap + base template

Lembre-se de versionar a pasta bootstrap também: `git add lists/static/bootstrap`.

## 3. Integrando o Bootstrap

Agora que temos um template base, podemos integrar o bootstrap com facilidade.
Altere o template base para:

```html
<!DOCTYPE html>
<html lang="en">

  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>To-Do lists</title>
    <link href="/static/bootstrap/css/bootstrap.min.css" rel="stylesheet">
  </head>

  <body>
    <div class="container">

      <div class="row">
        <div class="col-md-6 col-md-offset-3">
          <div class="text-center">
            <h1>{% block header_text %}{% endblock %}</h1>
            <form method="POST" action="{% block form_action %}{% endblock %}">
              <input name="item_text" id="id_new_item"
                     placeholder="Enter a to-do item" />
              {% csrf_token %}
            </form>
          </div>
        </div>
      </div>

      <div class="row">
        <div class="col-md-6 col-md-offset-3">
          {% block table %}
          {% endblock %}
        </div>
      </div>

    </div>
  </body>

</html>
```

Veja que o visual da aplicação ficou um pouco melhor.
Caso você nunca tenha visto o Bootstrap em ação, explore um pouco sua [documentação](https://getbootstrap.com)

Rode o teste funcional e veja que o erro ainda continua:

```sh
$ python manage.py test functional_tests
...
AssertionError: 80.33333587646484 != 512 within 10 delta (431.66666412353516 difference)
...
```

Para resolver esse problema, precisamos utilizar a classe `StaticLiveServerTestCase` ao invés de `LiveServerTestCase` no teste funcional.
Logo, altere o teste funcional **functional_tests/tests.py** de:

```python
from django.test import LiveServerTestCase
...
class NewVisitorTest(LiveServerTestCase):
...
```

Para:

```python
from django.contrib.staticfiles.testing import StaticLiveServerTestCase
...
class NewVisitorTest(StaticLiveServerTestCase):
...
```

Rode o teste funcional e veja ele passar com sucesso:

```sh
$ python manage.py test functional_tests
...
Ran 3 tests in 30.959s
OK
```

#### COMMIT & PUSH com a mensagem: Integrando o Bootstrap

## 4. Melhorando a estética da aplicação

Iremos agora melhorar o visual da aplicação.

Altere o arquivo **lists/templates/base.html** para utilizar a classe `jumbotron`:

```html
...
<div class="col-md-6 col-md-offset-3 jumbotron">
  <div class="text-center">
    <h1>{% block header_text %}{% endblock %}</h1>
      ...
```

Altere também o `<input>` do arquivo **lists/templates/base.html** para incluir fontes maiores, isto é, `class="form-control input-lg"`:

```html
...
<input name="item_text" id="id_new_item"
     class="form-control input-lg"
     placeholder="Enter a to-do item" />
...
```

Altere o arquivo **lists/templates/list.html** para incluir a classe `table`:

```html
...
<table id="id_list_table" class="table">
...
```

Verifique o novo visual da aplicação em http://localhost:8000.

#### COMMIT & PUSH com a mensagem: Melhorando a estética da aplicação

## 5. Refatorando o Teste Funcional

Antes de implementarmos nossa próxima funcionalidade (validação dos itens), iremos refatorar nosso Teste Funcional.
Atualmente, esse teste está longo, com diversos testes bem distintos. Precisamos organizar o Teste Funcional!

Primeiramente, em **functional_tests/tests.py**, coloque os testes nas suas próprias classes:

```python
class FunctionalTest(StaticLiveServerTestCase):

    def setUp(self):
        [...]
    def tearDown(self):
        ...
    def wait_for_row_in_list_table(self, row_text):
        ...


class NewVisitorTest(FunctionalTest):

    def test_can_start_a_list_for_one_user(self):
        ...
    def test_multiple_users_can_start_lists_at_different_urls(self):
        ...


class LayoutAndStylingTest(FunctionalTest):

    def test_layout_and_styling(self):
        ...
```

Rode o teste funcional para garantir que nada foi quebrado:

```sh
$ python manage.py test functional_tests
...
Ran 3 tests in 32.163s
OK
```

Agora mova cada classe de teste para um arquivo próprio.
Para facilitar, rode os seguintes comandos:

```sh
$ git mv functional_tests/tests.py functional_tests/base.py
$ cp functional_tests/base.py functional_tests/test_simple_list_creation.py
$ cp functional_tests/base.py functional_tests/test_layout_and_styling.py
```

Temos agora três novos arquivos: base.py, test_simple_list_creation.py e test_layout_and_styling.py.
Altere cada arquivo de teste para conter apenas sua classe de teste.

**functional_tests/base.py**
```python
from django.contrib.staticfiles.testing import StaticLiveServerTestCase
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
from selenium.common.exceptions import WebDriverException
import time

MAX_WAIT = 5

class FunctionalTest(StaticLiveServerTestCase):

    def setUp(self):
        ...

    def tearDown(self):
        ...

    def wait_for_row_in_list_table(self, row_text):
        ....

```

**functional_tests/test_simple_list_creation.py**
```python
from .base import FunctionalTest
from selenium import webdriver
from selenium.webdriver.common.keys import Keys

class NewVisitorTest(FunctionalTest):

    def test_can_start_a_list_for_one_user(self):
        ...

    def test_multiple_users_can_start_lists_at_different_urls(self):
        ...
```

**functional_tests/test_layout_and_styling.py**
```python
from .base import FunctionalTest
from selenium.webdriver.common.keys import Keys

class LayoutAndStylingTest(FunctionalTest):

    def test_layout_and_styling(self):
        ...
```

Ok, agora nosso teste funcional está corretamente organizado em três arquivos:

- base.py
- test_simple_list_creation.py
- test_layout_and_styling.py

Rode o teste funcional para garantir que nada foi quebrado.

```sh
$ python manage.py test functional_tests
...
Ran 3 tests in 32.163s
OK
```

Agora podemos também rodar apenas um teste funcional por vez.
Tente:

```sh
$ python manage.py test functional_tests.test_layout_and_styling
...
Ran 1 test in 11.333s
OK
```

#### COMMIT & PUSH com a mensagem: Refatorando o Teste Funcional

Lembre-se adicionar os novos arquivos no git:

```sh
$ git add functional_tests/test_layout_and_styling.py
$ git add functional_tests/test_simple_list_creation.py
...
```

## 6. Refatorando o Teste de Unidade

Em seguida, vamos realizar uma refatoração semelhante no Teste de Unidade, movendo os testes para `lists/tests`.

```sh
$ mkdir lists/tests
$ touch lists/tests/__init__.py
$ git mv lists/tests.py lists/tests/tests.py
```

Rode o teste de unidade para garantir que nada foi quebrado:

```sh
$ python manage.py test lists
...
Ran 9 tests in 0.071s
OK
```

Em seguida, vamos criar dois novos arquivos: test_views.py e test_models.py.
Para facilitar, rode os seguintes comandos:

```sh
$ git mv lists/tests/tests.py lists/tests/test_views.py
$ cp lists/tests/test_views.py lists/tests/test_models.py
```

Organize os novos arquivos conforme segue.

O arquivo **lists/tests/test_models.py** vai conter apenas a classe `ListAndItemModelsTest`:

```python
from django.test import TestCase
from lists.models import Item, List

class ListAndItemModelsTest(TestCase):
    ...
```
O arquivo **lists/tests/test_views.py** vai conter as demais classes: `HomePageTest`, `ListViewTest`, `NewListTest` e `NewItemTest`.

```python
from django.test import TestCase
from lists.models import Item, List

class HomePageTest(TestCase):
    ...

class ListViewTest(TestCase):
    ...

class NewListTest(TestCase):
    ...

class NewItemTest(TestCase):
    ...

```

Rode o teste de unidade para garantir que está tudo ok:

```sh
$ python manage.py test lists
...
Ran 9 tests in 0.043s
OK
```

#### COMMIT & PUSH com a mensagem: Refatorando o Teste de Unidade

Lembre-se de adicionar os novos arquivos no git: test_views.py e test_models.py.

## 7. Validação: Prevenindo itens vazios - PARTE 1

Vamos criar alguns testes para validação das entradas da aplicação.

Primeiramente, queremos prevenir que o usuário adicione itens vazios (atualmente isso é possível, verifique!).
Em **functional_tests**, crie um arquivo chamado **test_list_item_validation.py** com o seguinte teste funcional:

```python
from selenium.webdriver.common.keys import Keys
from unittest import skip
from .base import FunctionalTest

class ItemValidationTest(FunctionalTest):

    def test_cannot_add_empty_list_items(self):

        # Edith vai na home page e acidentalmente submete um item vazio
        self.browser.get(self.live_server_url)
        self.browser.find_element_by_id('id_new_item').send_keys(Keys.ENTER)

        # A página atualiza e apresenta uma mensagem de erro
        self.wait_for(lambda: self.assertEqual(
            self.browser.find_element_by_css_selector('.has-error').text,
            "You can't have an empty list item"
        ))

        # Ela adiciona alguns itens reais
        self.browser.find_element_by_id('id_new_item').send_keys('Buy milk')
        self.browser.find_element_by_id('id_new_item').send_keys(Keys.ENTER)
        self.wait_for_row_in_list_table('1: Buy milk')

        # Ela decide submeter uma lista vazia novamente
        self.browser.find_element_by_id('id_new_item').send_keys(Keys.ENTER)

        # A página atualiza e apresenta uma mensagem de erro
        self.wait_for(lambda: self.assertEqual(
            self.browser.find_element_by_css_selector('.has-error').text,
            "You can't have an empty list item"
        ))

        # E ela pode corrigir o erro, adicionando algum item correto
        self.browser.find_element_by_id('id_new_item').send_keys('Make tea')
        self.browser.find_element_by_id('id_new_item').send_keys(Keys.ENTER)
        self.wait_for_row_in_list_table('1: Buy milk')
        self.wait_for_row_in_list_table('2: Make tea')
```

Leia com atenção o teste acima!
Estamos apenas verificando se não é permitido adicionar um item vazio.
Essa funcionalidade ainda não está implementada.

Adicione também método de suporte `wait_for` em **functional_tests/base.py**:

```python
    def wait_for(self, fn):
        start_time = time.time()
        while True:
            try:
                return fn()  
            except (AssertionError, WebDriverException) as e:
                if time.time() - start_time > MAX_WAIT:
                    raise e
                time.sleep(0.5)
```

Rode apenas o novo teste funcional e veja a falha esperada:

```sh
$ python manage.py test functional_tests.test_list_item_validation
...
ERROR: test_cannot_add_empty_list_items (functional_tests.test_list_item_validation.ItemValidationTest)
...
selenium.common.exceptions.NoSuchElementException: Message: Unable to locate element: .has-error
```

#### COMMIT & PUSH com a mensagem: Validação: Prevenindo itens vazios - PARTE 1

Lembre-se de versionar o novo arquivo test_list_item_validation.py.

## 8. Validação: Prevenindo itens vazios - PARTE 2

Vamos iniciar a implementação da validação.
Em uma aplicação web, podemos validar os dados de entrada no lado do cliente ou servidor.
Validar no lado do servidor é mais seguro, pois alguém pode "desviar" do lado do cliente com código malicioso.
Desse modo, vamos fazer nossa validação do lado do servidor.


Como sempre no TDD, iniciamos pelos testes de unidade.
Adicione o seguinte teste na classe `ListAndItemModelsTest` em **lists/tests/test_models.py**

```python
from django.core.exceptions import ValidationError
...
    def test_cannot_save_empty_list_items(self):
        list_ = List.objects.create()
        item = Item(list=list_, text='')
        with self.assertRaises(ValidationError):
            item.save()
            item.full_clean()
```

O teste acima verifica se uma exceção é lançada quando adicionamos um item vazio no nosso BD, através do `assertRaises`.
O método `full_clean` do Django faz uma validação completa!

Rode o teste de unidade veja que ele vai passar:

```sh
$ python manage.py test lists
...
Ran 10 tests in 0.047s
OK
```

Mas como o teste passou?
Esse teste deveria falhar, uma vez que ainda não implementamos a validação?
Ou seja, estamos testando se uma exceção é lançada ao adicionar um item vazio e de fato uma exceção está sendo lançada.
Na verdade, por default, o `TextField` (utilizado na classe `Item` em **lists/models.py**) possui `blank=False`, isto é, não permite valores vazios.
Ou seja, tudo certo!

O nosso teste funcional continua falhando pois não estamos forçando esses erros serem mostrados na aplicação, fora do teste de unidade.

#### COMMIT & PUSH com a mensagem: Validação: Prevenindo itens vazios - PARTE 2

## 9. Validando os erros na view

Os erros não estão sendo mostrados na aplicação, pois não estamos fazendo nenhuma validação nas views nem transmitindo os erros para os templates, de modo que os usuários possam vê-los.
Em seguida, vamos trabalhar nessa validação.

Atualize o `<form>` do arquivo **lists/templates/base.html** para verificar se um erro foi passado utilizando `{% if error %}...{% endif %}`:

```html
...
            <form method="POST" action="{% block form_action %}{% endblock %}">
              <input name="item_text" id="id_new_item"
                     class="form-control input-lg"
                     placeholder="Enter a to-do item" />
              {% csrf_token %}
              {% if error %}
                <div class="form-group has-error">
                  <span class="help-block">{{ error }}</span>
                </div>
              {% endif %}
            </form>
...
```

Crie o seguinte teste na classe `NewListTest` em **lists/tests/test_views.py** (OBS: lembre-se de importar `escape`):

```python
from django.utils.html import escape
...

class NewListTest(TestCase):
    ...
    
    def test_validation_errors_are_sent_back_to_home_page_template(self):
        response = self.client.post('/lists/new', data={'item_text': ''})
        self.assertEqual(response.status_code, 200)
        self.assertTemplateUsed(response, 'home.html')
        expected_error = escape("You can't have an empty list item")
        self.assertContains(response, expected_error)
```

Rode o teste de unidade e veja a falha esperada:

```sh
$ python manage.py test lists
...
AssertionError: 302 != 200
```

A view está retornando um **302 redirect**, ao invés de um **200 normal**.

Altere o código do view `new_list` em **lists/views.py**, conforme apresentado abaixo.
Note que o código abaixo está capturando a exceção `ValidationError` e apresentando a página inicial.

```python
from django.core.exceptions import ValidationError
...
def new_list(request):
    list_ = List.objects.create()
    item = Item.objects.create(text=request.POST['item_text'], list=list_)
    try:
        item.full_clean()
    except ValidationError:
        return render(request, 'home.html')
    return redirect(f'/lists/{list_.id}/')
```

Rode o teste de unidade novamente.
Veja que agora não temos mais o `AssertionError: 302 != 200`, mas temos outro erro.

```sh
$ python manage.py test lists
...
AssertionError: False is not true : Couldn't find 'You can&#39;t have an empty list item' in response
```

Ocorre que ainda não estamos transmitindo a mensagem de erro para o template.
Altere o código do view `new_list` em **lists/views.py** para fazer exatamente isso: transmitir a mensagem de erro para o template:

```python
def new_list(request):
    list_ = List.objects.create()
    item = Item.objects.create(text=request.POST['item_text'], list=list_)
    try:
        item.full_clean()
    except ValidationError:
        error = "You can't have an empty list item"
        return render(request, 'home.html', {"error": error})
    return redirect(f'/lists/{list_.id}/')
```

Rode o teste de unidade novamente e veja ele passar com sucesso!

```sh
$ python manage.py test lists
...
Ran 11 tests in 0.063s
OK
```

Entre na aplicação (http://localhost:8000) e verifique a mensagem de erro ao tentar inserir um item vazio!

#### COMMIT & PUSH com a mensagem: Validando os erros na view

## 10. Entrada inválida não deve ser salva no BD

Note que a nossa última implementação contém um pequeno erro de lógica.
Observe o método `new_list` em **lists/views.py** e veja que estamos criando um objeto (linha 3) mesmo quando a validação falha:

```python
def new_list(request):
    list_ = List.objects.create()
    item = Item.objects.create(text=request.POST['item_text'], list=list_)
    try:
        item.full_clean()
    except ValidationError:
        error = "You can't have an empty list item"
        return render(request, 'home.html', {"error": error})
    return redirect(f'/lists/{list_.id}/')
```

Vamos criar um teste de unidade para pegar esse erro, ou seja, garantir que um item vazio **não** seja criado nem salvo no BD.

Na classe `NewListTest` em **lists/tests/test_views.py**, adicione o seguinte método de teste:

```python
class NewListTest(TestCase):
    ...
    
    def test_invalid_list_items_arent_saved(self):
        self.client.post('/lists/new', data={'item_text': ''})
        self.assertEqual(List.objects.count(), 0)
        self.assertEqual(Item.objects.count(), 0)
```

Rode o teste de unidade e veja a falha esperada.
De fato, o BD não está vazio após inserção de um item vazio.

```sh
$ python manage.py test lists
...
AssertionError: 1 != 0
```
Vamos consertar esse erro, de modo que só salvamos o dado **após** a validação.

Atualize o método `new_list` em **lists/views.py** para:

```python
def new_list(request):
    list_ = List.objects.create()
    item = Item(text=request.POST['item_text'], list=list_)
    try:
        item.full_clean()
        item.save()
    except ValidationError:
        list_.delete()
        error = "You can't have an empty list item"
        return render(request, 'home.html', {"error": error})
    return redirect(f'/lists/{list_.id}/')
```

Rode o teste de unidade novamente e veja ele passar com sucesso.

```sh
$ python manage.py test lists
...
Ran 12 tests in 0.054s
OK
```

#### COMMIT & PUSH com a mensagem: Entrada inválida não deve ser salva no BD

## 11. De volta ao teste funcional - PARTE 1

Lembra que o teste funcional `test_cannot_add_empty_list_items` em **test_list_item_validation.py** estava falhando completamente?
Vamos verificar se melhorou um pouco após as últimas implementações.

Rode teste funcional **test_list_item_validation** através do comando `python manage.py test functional_tests.test_list_item_validation`:

```sh
$ python manage.py test functional_tests.test_list_item_validation
...
File ... line 29, in test_cannot_add_empty_list_items
...
```

Veja que agora o erro ocorre na linha 29, no segundo `assert` e não mais no início (abra e confira o teste `test_cannot_add_empty_list_items` em **test_list_item_validation.py**).
Ou seja, passamos na parte inicial do teste funcional quando adicionamos um item vazio em uma **lista nova**.
Agora, precisamos passar na segunda parte do teste quando adicionamos um item vazio em uma **lista existente**.

Atualmente temos (1) uma view e URL para processar apresentar a lista (`view_list`) e (2) uma view e URL para adicionar na lista (`add_item`).
Vamos combinar ambos em uma única view para que essa lógica fique mais simples.

Primeiramente, atualize o alvo do template **lists/templates/list.html** de

```html
...
{% block form_action %}/lists/{{ list.id }}/add_item{% endblock %}
...
```

Para: 

```html
...
{% block form_action %}/lists/{{ list.id }}/{% endblock %}
...
```

Rode o teste funcional e veja que ele vai falhar complementamente: `python manage.py test functional_tests`.

Em **lists/tests/test_views.py**, mova os dois métodos da classe `NewItemTest` para a classe `ListViewTest` (com pequenas alterações na URL).
Após realizar a movimentação, lembre-se de remover a classe vazia `NewItemTest`.

```python
class ListViewTest(TestCase):

    def test_uses_list_template(self):
        ...

    def test_displays_only_items_for_that_list(self):
        ...

    def test_passes_correct_list_to_template(self):
        ...

    def test_can_save_a_POST_request_to_an_existing_list(self):
        other_list = List.objects.create()
        correct_list = List.objects.create()

        self.client.post(
            f'/lists/{correct_list.id}/',
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
            f'/lists/{correct_list.id}/',
            data={'item_text': 'A new item for an existing list'}
        )
        self.assertRedirects(response, f'/lists/{correct_list.id}/')
```

Rode o teste de unidade (`python manage.py test lists`) e veja a falha esperada.

Em seguida, atualize o método `view_list` em **lists/views.py** para lidar com dois tipos de requisições:

```python
def view_list(request, list_id):
    list_ = List.objects.get(id=list_id)
    if request.method == 'POST':
        Item.objects.create(text=request.POST['item_text'], list=list_)
        return redirect(f'/lists/{list_.id}/')
    return render(request, 'list.html', {'list': list_})
```

Por fim, remova a view `add_item` em **lists/views.py**, uma vez que ela não é mais utilizada.
Delete também a linha que faz referência a view `add_item` em  **superlists/urls.py**.

Rode o teste de unidade novamente:

```sh
$ python manage.py test lists
...
Ran 12 tests in 0.054s
OK
```

Ok, parte 1 concluída.

#### COMMIT & PUSH com a mensagem: De volta ao teste funcional - PARTE 1

## 12. De volta ao teste funcional - PARTE 2

Estamos na parte final da aula.

Após a alteração na última etapa, fica mais fácil implementar a funcionalidade de validação para que o teste funcional possa passar complementamente.

Adicione o seguinte teste de unidade na view na classe `ListViewTest` em **lists/tests/test_views.py**:

```python
class ListViewTest(TestCase):
    ...
    
    def test_validation_errors_end_up_on_lists_page(self):
        list_ = List.objects.create()
        response = self.client.post(
            f'/lists/{list_.id}/',
            data={'item_text': ''}
        )
        self.assertEqual(response.status_code, 200)
        self.assertTemplateUsed(response, 'list.html')
        expected_error = escape("You can't have an empty list item")
        self.assertContains(response, expected_error)
```

Rode o teste de unidade e veja a falha esperada:

```sh
$ python manage.py test lists
...
AssertionError: 302 != 200
...
Ran 13 tests in 0.061s
```

Atualize o método `view_list` em **lists/views.py** para:

```python
def view_list(request, list_id):
    list_ = List.objects.get(id=list_id)
    error = None

    if request.method == 'POST':
        try:
            item = Item(text=request.POST['item_text'], list=list_)
            item.full_clean()
            item.save()
            return redirect(f'/lists/{list_.id}/')
        except ValidationError:
            error = "You can't have an empty list item"

    return render(request, 'list.html', {'list': list_, 'error': error})
```

Rode o teste de unidade com sucesso:

```sh
$ python manage.py test lists
...
Ran 13 tests in 0.060s
OK
```

Por fim, rode o teste funcional, também com sucesso:

```sh
$ python manage.py test functional_tests
...
Ran 4 tests in 40.520s
OK
```

Estamos de volta ao estado VERDE, com a validação implementada e todos os testes passando.
Explore a aplicação (http://localhost:8000) e veja a validação em funcionamento.

#### COMMIT & PUSH com a mensagem: De volta ao teste funcional - PARTE 2
