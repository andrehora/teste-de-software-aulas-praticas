# Teste de Software

## Aula Prática 5: Autenticação e Test Doubles

Prof. André Hora

Objetivo: Reproduzir uma sessão de desenvolvimento de software usando TDD (Test-Driven Development), com base no livro *Test-Driven-Development with Python*.
Especificamente, iremos reproduzir os exemplos dos capítulos 18 e 19 (parcialmente).
Faça o exercício com o mindset de TDD.
Se necessário, estude antes o material visto em sala de aula.

Instruções:

- Siga o roteiro, passo a passo.
- Após cada passo, rode os testes.
- Nos passos marcados com COMMIT & PUSH, faça um commit e um push no seu repositório. 
Isso será usado no momento da correção, para garantir que seguiu a sequência do roteiro, passo-a-passo.

## 1. Explorando autenticação por email (passwordless)

Nesta aula, iremos implementar e testar uma funcionalidade de autenticação por email, chamada de **passwordless**.
Basicamente, o usuário entra com seu email pessoal na aplicação TODO e a aplicação envia uma URL para que o usuário possa fazer login.
A vantagem dessa solução de login é que o usuário não precisa memorizar senhas: basta entrar com o seu email pessoal e clicar na URL enviada de login.

Para isso, devemos implementar uma solução para a aplicação enviar emails.
O envio de emails pelo Django será realizado através de uma **nova conta de email** que você deve criar no Gmail; explicamos isso em breve.

O Passo 1 da aula é meramente exploratório, também conhecido no TDD como [Spike](https://en.wikipedia.org/wiki/Spike_(software_development)).
Ou seja, nesse passo, não iremos aplicar TDD: vamos apenas implementar um protótipo de autenticação passwordless.
Nos próximos passos (2, 3, 4...), voltamos ao TDD.

Antes de tudo, crie uma nova branch de trabalho no git, chamada `passwordless-feature`:

```sh
$ git checkout -b passwordless-feature
```

Rode o comando a seguir para criar uma nova aplicação Django chamada `accounts`:

```sh
$ python manage.py startapp accounts
```

Adicione a nova aplicação `accounts` em **superlists/settings.py**

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'lists',
    'accounts',
]
```

### URLs

Atualize o arquivo **superlists/urls.py** para:

```python
from django.conf.urls import url
from lists import views as list_views
from accounts import views as accounts_views

urlpatterns = [
    url(r'^$', list_views.home_page, name='home'),
    url(r'^lists/new$', list_views.new_list, name='new_list'),
    url(r'^lists/(\d+)/$', list_views.view_list, name='view_list'),

    url(r'^accounts/send_email$', accounts_views.send_login_email, name='send_login_email'),
    url(r'^accounts/login$', accounts_views.login, name='login'),
    url(r'^accounts/logout$', accounts_views.logout, name='logout'),
]
```

### Views

Crie as views de `account` em **accounts/views.py** com os métodos `send_login_email`, `login` e `logout`.
A view `send_login_email` gera um id único `uid`, salva o id no BD e envia o id por email em uma URL de login (através do método `send_mail`).
A view `login` (que é chamada quando o usuário clica na URL enviada por email) verifica se o id único é válido (através do método `authenticate`) e autentica o usuário.

```python
import uuid, sys
from django.contrib.auth import authenticate
from django.contrib.auth import login as auth_login, logout as auth_logout
from django.core.mail import send_mail
from django.shortcuts import redirect, render
from accounts.models import Token

def send_login_email(request):
    email = request.POST['email']
    uid = str(uuid.uuid4())
    Token.objects.create(email=email, uid=uid)
    print('saving uid', uid, 'for email', email, file=sys.stderr)
    url = request.build_absolute_uri(f'/accounts/login?uid={uid}')
    send_mail(
        'Your login link for Superlists',
        f'Use this link to log in:\n\n{url}',
        'noreply@superlists',
        [email],
    )
    return render(request, 'login_email_sent.html')

def login(request):
    print('login view', file=sys.stderr)
    uid = request.GET.get('uid')
    user = authenticate(uid=uid)
    if user is not None:
        auth_login(request, user)
    return redirect('/')

def logout(request):
    auth_logout(request)
    return redirect('/')
```

### HTMLs

Adicione o seguinte arquivo HTML em **accounts/templates/login_email_sent.html**:

```html
<html>
<h1>Email sent</h1>

<p>Check your email, you'll find a message with a link that will log you into
the site.</p>

</html>
```

Atualize o arquivo **lists/templates/base.html** para:

```html
...
  <body>
    <div class="container">

      <div class="navbar">
        {% if user.is_authenticated %}
          <p>Logged in as {{ user.email }}</p>
          <p><a id="id_logout" href="{% url 'logout' %}">Log out</a></p>
        {% else %}
          <form method="POST" action ="{% url 'send_login_email' %}">
            Enter email to log in: <input name="email" type="text" />
            {% csrf_token %}
          </form>
        {% endif %}
      </div>

      <div class="row">
      ....
```

### Models

Implemente as classes `Token`, `ListUserManager` e `ListUser` em **accounts/models.py**:

```python
from django.db import models
from django.contrib.auth.models import (
    AbstractBaseUser, BaseUserManager, PermissionsMixin
)

class Token(models.Model):
    email = models.EmailField()
    uid = models.CharField(max_length=255)

class ListUserManager(BaseUserManager):

    def create_user(self, email):
        ListUser.objects.create(email=email)

    def create_superuser(self, email, password):
        self.create_user(email)

class ListUser(AbstractBaseUser, PermissionsMixin):
    email = models.EmailField(primary_key=True)
    USERNAME_FIELD = 'email'
    #REQUIRED_FIELDS = ['email', 'height']

    objects = ListUserManager()

    @property
    def is_staff(self):
        return self.email == 'harry.percival@example.com'

    @property
    def is_active(self):
        return True
```

Crie um arquivo chamado **accounts/authentication.py**, conforme mostrado abaixo.
Note que o método `authenticate` é chamado na view `login` para realizar a autenticação.
Esse método verifica que se o id único é válido, ou seja, existe no BD.
Caso seja válido, o email do usuário é recuperado e o usuário `ListUser` é retornado (ou criado caso seja um novo email).
Por fim, a view `login` irá autenticar tal usuário.

```python
import sys
from accounts.models import ListUser, Token

class PasswordlessAuthenticationBackend(object):

    def authenticate(self, uid):
        print('uid', uid, file=sys.stderr)
        if not Token.objects.filter(uid=uid).exists():
            print('no token found', file=sys.stderr)
            return None
        token = Token.objects.get(uid=uid)
        print('got token', file=sys.stderr)
        try:
            user = ListUser.objects.get(email=token.email)
            print('got user', file=sys.stderr)
            return user
        except ListUser.DoesNotExist:
            print('new user', file=sys.stderr)
            return ListUser.objects.create(email=token.email)


    def get_user(self, email):
        return ListUser.objects.get(email=email)
```

Configure alguns detalhes da autenticação em **superlists/settings.py**, referenciando as classes `ListUser` e `PasswordlessAuthenticationBackend`:

```python
...
AUTH_USER_MODEL = 'accounts.ListUser'
AUTHENTICATION_BACKENDS = [
    'accounts.authentication.PasswordlessAuthenticationBackend',
]
...
```

### Conta de Email no Gmail

Crie uma conta de email no Gmail ([criar conta](https://accounts.google.com/signup/v2/webcreateaccount?service=mail&continue=https%3A%2F%2Fmail.google.com%2Fmail%2F&ltmpl=default&gmb=exp&biz=false&flowName=GlifWebSignIn&flowEntry=SignUp)).
Após criar sua nova conta, entre em `Manage your Google Account` e `Security`.
Em `Less secure app access`, selecione `ON` e `Allow less secure apps: ON`.

OBS: se sua conta estiver em português: `Gerenciar sua Conta do Google` => `Segurança` => `Acesso a app menos seguro` => `Ativar acesso` => `Permitir aplicativos menos seguros: ATIVADO`.

Para que o `send_mail` do Django funcione, precisamos informar o servidor de email em **superlists/settings.py**.
Coloque seu novo email no lugar de `SEU_NOVO_EMAIL@gmail.com`.
OBS: não coloque sua senha nesse arquivo, ou seja, não altere o valor `EMAIL_PASSWORD`.

```python
...
EMAIL_HOST = 'smtp.gmail.com'
EMAIL_HOST_USER = 'SEU_NOVO_EMAIL@gmail.com'
EMAIL_HOST_PASSWORD = os.environ.get('EMAIL_PASSWORD')
EMAIL_PORT = 587
EMAIL_USE_TLS = True
...
```

### BD

Atualize o BD de acordo com novo esquema:

``` sh
$ python manage.py makemigrations
```

``` sh
$ python manage.py migrate
```

### Server

Informe a senha da nova conta do Gmail no terminal através da variável de ambiente `EMAIL_PASSWORD`:

```sh
$ export EMAIL_PASSWORD="SUA_SENHA"
```

Por fim, **no mesmo terminal** que você informou a senha, suba o servidor:

```sh
$ python manage.py runserver
...
Performing system checks...
System check identified no issues (0 silenced).
October 18, 2020 - 18:29:19
Django version 1.11.29, using settings 'superlists.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```

### Teste a autenticação

- Entre em http://localhost:8000 e note no canto superior esquerdo o campo para login.
- Digite o seu email pessoal (não o email que você acabou de criar) e aperte ENTER.
- Verifique o email enviado para o seu email pessoal com o seguinte título `Your login link for Superlists`.
- Clique na URL enviada por email e verifique que você foi logado com sucesso.

### Comentários finais

- Spiking: exploração de código para entender uma API ou para investigar a viabilidade de uma nova solução. Spiking pode ser realizado sem testes. É uma boa ideia fazer isso em uma nova branch (como fizemos) e depois voltar voltar para a branch principal.

- De-spiking: após explorar a nova solução, começar novamente do zero, mas agora utilizando TDD.

#### COMMIT & PUSH com a mensagem: Explorando autenticação por email (passwordless)

Lembre-se de adicionar a nova pasta `accounts` e de fazer o commit na branch `passwordless-feature`:

```sh
$ git add accounts
$ git add -u
$ git commit -m "Explorando autenticação por email (passwordless)"
$ git push -u origin passwordless-feature
```

## 2. De volta ao TDD

Agora devemos reescrever o protótipo do passo anterior com TDD.
Ou seja, agora que temos informação suficiente, devemos fazer da forma correta, com testes.

Qual primeiro passo? Um teste funcional, claro!

Adicione o seguinte teste funcional **test_login.py** em **functional_tests**:

```python
from django.core import mail
from selenium.webdriver.common.keys import Keys
import re

from .base import FunctionalTest

TEST_EMAIL = 'edith@example.com'
SUBJECT = 'Your login link for Superlists'


class LoginTest(FunctionalTest):

    def test_can_get_email_link_to_log_in(self):
        # Edith entra no site e nota o campo "Log in"
        # O campo informa para ela entrar com o email, e ela entra
        self.browser.get(self.live_server_url)
        self.browser.find_element_by_name('email').send_keys(TEST_EMAIL)
        self.browser.find_element_by_name('email').send_keys(Keys.ENTER)

        # Uma mensagem informa que o email email foi enviado para ela
        self.wait_for(lambda: self.assertIn(
            'Check your email',
            self.browser.find_element_by_tag_name('body').text
        ))

        # Ela verifica o email e encontra a mensagem
        email = mail.outbox[0]
        self.assertIn(TEST_EMAIL, email.to)
        self.assertEqual(email.subject, SUBJECT)

        # O email contém uma URL
        self.assertIn('Use this link to log in', email.body)
        url_search = re.search(r'http://.+/.+$', email.body)
        if not url_search:
            self.fail(f'Could not find url in email body:\n{email.body}')
        url = url_search.group(0)
        self.assertIn(self.live_server_url, url)

        # Ela clica na URL
        self.browser.get(url)

        # Ela está logada
        self.wait_for(
            lambda: self.browser.find_element_by_link_text('Log out')
        )
        navbar = self.browser.find_element_by_css_selector('.navbar')
        self.assertIn(TEST_EMAIL, navbar.text)
```

Leia com atenção o teste acima!
Note o atributo `mail.outbox`, que nos fornece acesso a todos os emails que o servidor tentar enviar.

Rode apenas o teste funcional **test_login.py** e veja ele passar com sucesso:

```sh
$ python manage.py test functional_tests.test_login
...
saving uid f76f7410-a04d-43f5-9179-9053be425c3e for email edith@example.com
login view
uid f76f7410-a04d-43f5-9179-9053be425c3e
got token
new user
...
Ran 1 test in 11.351s
```

Ok, agora vamos voltar para a branch principal `main`.

```sh
$ git checkout main
```

Remova a pasta **accounts** que foi criada no passo anterior (iremos re-implementá-la com TDD):

```sh
$ rm -rf accounts
```

#### COMMIT & PUSH com a mensagem: De volta ao TDD

IMPORTANTE: Adicione o teste funcional **test_login.py** na `main`:

```sh
$ git add functional_tests/test_login.py
$ git commit -m "De volta ao TDD"
$ git push -u origin main
```

## 3. Recriando a aplicação accounts

Rode o teste funcional **test_login.py** novamente e veja a falha esperada:

```sh
$ python manage.py test functional_tests.test_login
...
selenium.common.exceptions.NoSuchElementException: Message: Unable to locate element: [name="email"]
```

Crie novamente a aplicação `accounts`:

```sh
$ python manage.py startapp accounts
```

Em seguida, adicione o seguinte código em **lists/templates/base.html**:

```html
...
  <body>
    <div class="container">

      <nav class="navbar navbar-default" role="navigation">
        <div class="container-fluid">
          <a class="navbar-brand" href="/">Superlists</a>
          <form class="navbar-form navbar-right" method="POST" action="#">
            <span>Enter email to log in:</span>
            <input class="form-control" name="email" type="text" />
            {% csrf_token %}
          </form>
        </div>
      </nav>

      <div class="row">
      ...
```

Rode novamente o teste funcional **test_login.py** novamente e veja outra falha:

```sh
$ python manage.py test functional_tests.test_login
...
AssertionError: 'Check your email' not found in 'Superlists\nEnter email to log in:\nStart a new To-Do list'
```

#### COMMIT & PUSH com a mensagem: Recriando a aplicação accounts

Lembre-se de adicionar a pasta **accounts** na branch main.

## 4. Modelo de usuário mínimo

O objeto usuário da nossa aplicação será clean: iremos salvar uma única informação, o email.
Ou seja, não iremos salvar nome, sobrenome, username, etc.
De fato, como nossa aplicação vai utilizar a autenticação passwordless, precisamos apenas do email.

Em, **accounts** crie a pasta **tests** e adicione o seguinte arquivo **accounts/tests/test_models.py**:

```python
from django.test import TestCase
from django.contrib.auth import get_user_model

User = get_user_model()

class UserModelTest(TestCase):

    def test_user_is_valid_with_email_only(self):
        user = User(email='a@b.com')
        user.full_clean()  # should not raise

    def test_email_is_primary_key(self):
        user = User(email='a@b.com')
        self.assertEqual(user.pk, 'a@b.com')

```

Adicione também o arquivo **\_\_init\_\_.py** em **accounts/tests**.
Além disso, remova o arquivo gerado automaticamente **accounts/tests.py**.

Rode o novo teste de unidade e veja a falha esperada:

```sh
$ python manage.py test accounts
django.core.exceptions.ValidationError: {'password': ['This field cannot be blank.'], 'username': ['This field cannot be blank.']}
...
```

Crie a classe `User` em **accounts/models.py**

```python
from django.db import models

class User(models.Model):
    email = models.EmailField(primary_key=True)
```

Atualize o arquivo **superlists/settings.py** para incluir `accounts` e `User`:

```python
...
INSTALLED_APPS = [
    ...
    'accounts',
]
...
AUTH_USER_MODEL = 'accounts.User'
```

O próximo erro é de BD:

```sh
$ python manage.py test accounts
...
django.db.utils.OperationalError: no such table: accounts_user
```

Rode o comando `makemigrations` e veja o problema:

```sh
$ python manage.py makemigrations
...
AttributeError: type object 'User' has no attribute 'REQUIRED_FIELDS':
```

Atualize **accounts/models.py** para incluir `REQUIRED_FIELDS` (e outros atributos que você pode descobrir se realizar essa iteração algumas vezes `USERNAME_FIELD`, `is_anonymous` e `is_authenticated`):

```python
from django.db import models

class User(models.Model):
    email = models.EmailField(primary_key=True)

    REQUIRED_FIELDS = []
    USERNAME_FIELD = 'email'
    is_anonymous = False
    is_authenticated = True
```

Rode novamente o comando `makemigrations` com sucesso:

```sh
$ python manage.py makemigrations
```

Por fim, rode o teste de unidade com sucesso:

```sh
$ python manage.py test accounts
...
Ran 2 tests in 0.005s
OK
```

#### COMMIT & PUSH com a mensagem: Modelo de usuário mínimo

## 5. Modelo Token

Agora vamos trabalhar no modelo de Token, que deve fazer o link entre email e o ID único (que é gerado e enviado por email para o login).

Adicione o seguinte teste de unidade em **accounts/tests/test_models.py**:

```python
from accounts.models import Token
...
class TokenModelTest(TestCase):

    def test_links_user_with_auto_generated_uid(self):
        token1 = Token.objects.create(email='a@b.com')
        token2 = Token.objects.create(email='a@b.com')
        self.assertNotEqual(token1.uid, token2.uid)
```

Rode o teste de unidade e veja a falha esperada.

Crie a classe `Token` em **accounts/models.py**:

```python
import uuid
...

class Token(models.Model):
    email = models.EmailField()
    uid = models.CharField(default=uuid.uuid4, max_length=40)
```

Rode também o comando `makemigrations`:

```sh
$ python manage.py makemigrations
```

Por fim, rode o teste de unidade novamente com sucesso:

```sh
$ python manage.py test accounts
...
Ran 3 tests in 0.008s
OK
```

#### COMMIT & PUSH com a mensagem: Modelo Token

## 6. Testando o envio de email: redirecionando para o home

Adicione o seguinte teste de unidade em **accounts/tests/test_views.py**:

```python
from django.test import TestCase

class SendLoginEmailViewTest(TestCase):

    def test_redirects_to_home_page(self):
        response = self.client.post('/accounts/send_login_email', data={
            'email': 'edith@example.com'
        })
        self.assertRedirects(response, '/')
```

Atualize as URLs em **superlists/urls.py** para incluir a view `send_login_email`:


```python
...
from accounts import views as accounts_views

urlpatterns = [
    ...
    url(r'^accounts/send_login_email$', accounts_views.send_login_email, name='send_login_email'),
]
```

Rode o teste de unidade e veja a falha esperada:

```sh
$ python manage.py test accounts
...
AttributeError: module 'accounts.views' has no attribute 'send_login_email'
```

Implemente a view `send_login_email` em **accounts/views.py**:

```python
from django.core.mail import send_mail
from django.shortcuts import redirect

def send_login_email(request):
    email = request.POST['email']
    # send_mail(
    #     'Your login link for Superlists',
    #     'body text tbc',
    #     'noreply@superlists',
    #     [email],
    # )
    return redirect('/')
```

Observe que no código acima não estamos enviando emails ainda.
Isso será tratado no próximo passo.

Rode novamente o teste de unidade com sucesso:

```sh
$ python manage.py test accounts
... 
Ran 4 tests in 0.019s
OK
```

#### COMMIT & PUSH com a mensagem: Testando o envio de email: redirecionando para o home

## 7. Testando o envio de email: Test Double manual

Quando chamamos o método `send_mail`, o Django faz uma conexão com o servidor de email e de fato envia um email através da Internet.
Isso não é algo que queremos no teste de unidade, pois isso viola os princípios [FIRST](https://engsoftmoderna.info/cap8.html).

O mesmo "problema" pode ocorrer em diversos outros cenários onde existe uma dependência externa: enviar um tweet, enviar um SMS, chamar uma API externa, etc.
Não queremos que a cada execução do teste de unidade sejam enviados emails, tweets, etc.
Ainda sim, queremos testar se o nosso código está correto.
Nesses casos, podemos utilizar [Test Doubles](http://xunitpatterns.com/Test%20Double.html).

Considere o método `send_login_email` já criado em **accounts/views.py**:

```python
def send_login_email(request):
    email = request.POST['email']
    # send_mail(
    #     'Your login link for Superlists',
    #     'body text tbc',
    #     'noreply@superlists',
    #     [email],
    # )
    return redirect('/')
```

Como podemos testar esse método sem de fato chamar o `send_mail` real?
O teste deve substituir `send_mail` por uma versão "fake", de modo que o `send_mail` real não seja chamado.

Vamos primeiramente criar um Test Double de forma manual.
Adicione o seguinte teste de unidade na classe `SendLoginEmailViewTest` em **accounts/tests/test_views.py**:

```python
from django.test import TestCase
import accounts.views

class SendLoginEmailViewTest(TestCase):
    ...
    
    def test_sends_mail_to_address_from_post(self):
        self.send_mail_called = False

        def fake_send_mail(subject, body, from_email, to_list):
            self.send_mail_called = True
            self.subject = subject
            self.body = body
            self.from_email = from_email
            self.to_list = to_list

        accounts.views.send_mail = fake_send_mail

        self.client.post('/accounts/send_login_email', data={
            'email': 'edith@example.com'
        })

        self.assertTrue(self.send_mail_called)
        self.assertEqual(self.subject, 'Your login link for Superlists')
        self.assertEqual(self.from_email, 'noreply@superlists')
        self.assertEqual(self.to_list, ['edith@example.com'])
```

Observe com atenção o código acima.
Note que estamos definindo o método `fake_send_mail`: sua única tarefa é salvar algumas informações sobre como ele foi chamado.
Em seguida, antes do SUT ser executado, trocamos o `send_mail` real pela sua versão "fake" (linha `accounts.views.send_mail = fake_send_mail`).
Tecnicamente, qual o nome deste Test Double? Dummy, Stub, Spy, Mock ou Fake?

Em seguida, _descomente_ o código de `send_mail` em **accounts/views.py**:

```python
from django.core.mail import send_mail
from django.shortcuts import redirect

def send_login_email(request):
    email = request.POST['email']
    send_mail(
        'Your login link for Superlists',
        'body text tbc',
        'noreply@superlists',
        [email],
    )
    return redirect('/')
```

Rode o teste de unidade com sucesso:

```sh
$ python manage.py test accounts
...
Ran 5 tests in 5.026s
OK
```

#### COMMIT & PUSH com a mensagem: Testando o envio de email: Test Double manual

## 8. Testando o envio de email: Test Double lib

Estamos no último passo.

No passo anterior, testamos o envio de email através de um Test Double manual.
Em seguida, iremos utilizar uma biblioteca de Test Double para testar o envio de emails.

Python contém um módulo bem popular chamado **mock**.
Esse módulo fornece um objeto importante chamado `Mock`.

Faça um teste simples em um shell Python 3.3+ (basta digitar python na linha de comando):

```python
>>> from unittest.mock import Mock
>>> m = Mock()
>>> m.any_attribute
<Mock name='mock.any_attribute' id='4341496128'>
>>> type(m.any_attribute)
<class 'unittest.mock.Mock'>
>>> m.any_method()
<Mock name='mock.any_method()' id='4341496576'>
>>> m.foo()
<Mock name='mock.foo()' id='4344310696'>
>>> m.called
False
>>> m.foo.called
True
>>> m.bar.return_value = 1
>>> m.bar(42)
1
>>> m.bar.call_args
call(42)
````

Observe que um objeto `Mock` responde a qualquer solicitação de atributo ou chamada de método.

### Utilizando o unittest.patch

O módulo `mock` fornece uma função utilitária chamada `patch`, que podemos utilizar para simular os objetos reais.

Atualize o teste de unidade `test_sends_mail_to_address_from_post` em **accounts/tests/test_views.py** para:

```python
...
from unittest.mock import patch

class SendLoginEmailViewTest(TestCase):
    ...

    @patch('accounts.views.send_mail')
    def test_sends_mail_to_address_from_post(self, mock_send_mail):
        self.client.post('/accounts/send_login_email', data={
            'email': 'edith@example.com'
        })

        self.assertEqual(mock_send_mail.called, True)
        (subject, body, from_email, to_list), kwargs = mock_send_mail.call_args
        self.assertEqual(subject, 'Your login link for Superlists')
        self.assertEqual(from_email, 'noreply@superlists')
        self.assertEqual(to_list, ['edith@example.com'])
```

No código acima:

- O decorator `@patch` recebe o nome do objeto a ser "mocado". Isso equivale ao que fizemos no passo anterior, quando criamos um Test Double manual para `send_mail`. A vantagem do decorator é que objeto alvo (no caso `send_mail`) é substituído/gerenciado automaticamente por `Mock`. Ou seja, nesse caso não foi necessário criar um Test Double de forma manual.

- `patch` então injeta o objeto "mocado" no teste como um parâmetro no método de teste (note o parâmetro `mock_send_mail`). Podemos dar qualquer nome para esse parâmetro, mas uma boa prática é utilizar `mock_` + o nome original do objeto.

- Chamamos o método de teste normalmente, mas a view não irá chamar o `send_mail` original, mas sim o `mock_send_mail`.

- Podemos chamar os `asserts` normalmente, testando o que ocorreu com o objeto "mocado" durante o teste. Por exemplo, se foi chamado (`mock_send_mail.called`) e com quais argumentos (`subject`, `from_email`, `to_list`, etc).

Por fim, rode o teste de unidade com sucesso:

```sh
$ python manage.py test accounts
...
Ran 5 tests in 5.032s
OK
```

#### COMMIT & PUSH com a mensagem: Testando o envio de email: Test Double lib
