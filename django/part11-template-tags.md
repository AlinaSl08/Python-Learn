# Теги шаблонов.

## Шаблонные теги `{% for %}` и `{% if %}` в Django

**Шаблонные теги** — это специальные конструкции шаблонизатора Django, которые отрабатывают **на стороне сервера** и управляют логикой формирования итогового HTML-документа (не путать с обычными HTML-тегами, которые обрабатываются в браузере).

* Синтаксис тегов: `{% <название_тега> [параметры] %}`
* Официальная документация: [Built-in template tags and filters](https://docs.djangoproject.com/en/4.2/ref/templates/builtins/).

---

## 1. Подготовка данных в представлении (`views.py`)

Для примера передадим в шаблон список словарей со статьями, где у каждой записи есть флаг публикации `is_published`:

```python
data_db = [
    {'id': 1, 'title': 'Анджелина Джоли', 'content': 'Биография Анджелины Джоли', 'is_published': True},
    {'id': 2, 'title': 'Марго Робби', 'content': 'Биография Марго Робби', 'is_published': False},
    {'id': 3, 'title': 'Джулия Робертс', 'content': 'Биография Джулия Робертс', 'is_published': True},
]

def index(request):
    data = {
        'title': 'Главная страница',
        'menu': menu,
        'posts': data_db,
    }
    return render(request, 'women/index.html', context=data)
```

---

## 2. Тег цикла `{% for %}`

Работает по аналогии с циклом `for` в Python и позволяет перебирать коллекции. Цикл **обязательно** должен закрываться тегом **`{% endfor %}`**.

```html
<ul>
    {% for p in posts %}
    <li>
        <h2>{{ p.title }}</h2>
        <p>{{ p.content }}</p>
        <hr>
    </li>
    {% endfor %}
</ul>
```

### Вспомогательная переменная `forloop`
Внутри цикла `{% for %}` автоматически доступна специальная переменная `forloop`, содержащая служебную информацию о текущей итерации:

* **`forloop.last`** — принимает значение `True` на **последней** итерации цикла (и `False` на всех остальных).
* **`forloop.first`** — принимает значение `True` на **первой** итерации.
* **`forloop.counter`** — номер текущей итерации (начиная с `1`).
* **`forloop.counter0`** — индекс текущей итерации (начиная с `0`).

---

## 3. Условный тег `{% if %}`

Позволяет проверять условия (аналог `if` в Python) и включать фрагмент HTML в итоговую страницу только при истинности условия. Блок **обязательно** закрывается тегом **`{% endif %}`**.

* Поддерживаются логические операторы: **`not`**, **`and`**, **`or`**.
* Поддерживаются дополнительные ветки: **`{% elif %}`** и **`{% else %}`**.

### Практический пример (фильтрация постов и скрытие лишней черты `<hr>`):

```html
<ul>
    {% for p in posts %}
        {% if p.is_published %}
        <li>
            <h2>{{ p.title }}</h2>
            <p>{{ p.content }}</p>
            {% if not forloop.last %}
                <hr>
            {% endif %}
        </li>
        {% endif %}
    {% endfor %}
</ul>
```

**Как это работает:**
1. `{% if p.is_published %}` — выводит элемент списка `<li>` только если статья опубликована (`True`). Статья про Марго Робби (`False`) в итоговый HTML не попадет.
2. `{% if not forloop.last %}` — добавляет разделительную линию `<hr>` после каждого поста, кроме самого последнего в списке.

## Шаблонный тег `{% url %}` в Django

**Шаблонный тег `{% url %}`** используется для динамического формирования URL-адресов по их именам, заданным в `urls.py`. Это избавляет от необходимости жестко прописывать пути в HTML-коде (хардкодинга).

* Официальная документация: [Built-in template tags and filters (`url`)](https://djangoproject.com).

---

## 1. Зачем нужен тег `{% url %}`

* ❌ **Плохо (хардкодинг):** `<a href="post/{{ p.id }}/">Читать пост</a>`
  * *Проблема:* Если путь в `urls.py` изменится (например, с `post/` на `article/`), придется вручную исправлять ссылки во всех шаблонах проекта.
* ✅ **Хорошо (динамически):** `<a href="{% url 'post' p.id %}">Читать пост</a>`
  * *Преимущество:* Тег работает как функция `reverse()` в Python — сам строит актуальный путь по имени маршрута (`name='post'`). При изменении URL-шаблона в `urls.py` все ссылки на сайте обновятся автоматически.

---

## 2. Синтаксис тега `{% url %}`

```html
<!-- Без параметров (имя маршрута берется в кавычки) -->
{% url 'home' %}

<!-- С передачей аргумента (параметры пишутся через пробел) -->
{% url 'post' p.id %}

<!-- С использованием переменной вместо строкового литерала -->
{% url m.url_name %}
```

> **Важно:** Если имя маршрута передается строкой, его обязательно нужно заключать в кавычки (`'home'`). Если кавычек нет (`m.url_name`), Django будет искать переменную с таким именем в контексте шаблона.

---

## 3. Практический пример: ссылки на посты и главное меню

### Шаг 1: Настройка маршрутов (`women/urls.py`)
Каждому маршруту задаем уникальное неизменное имя (`name`):

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.index, name='home'),
    path('about/', views.about, name='about'),
    path('addpage/', views.addpage, name='add_page'),
    path('contact/', views.contact, name='contact'),
    path('login/', views.login, name='login'),
    path('post/<int:post_id>/', views.show_post, name='post'),
]
```

### Шаг 2: Подготовка данных и представлений (`women/views.py`)
Создаем структуру меню с привязкой к именам маршрутов и функции-заглушки:

```python
from django.http import HttpResponse
from django.shortcuts import render

menu = [
    {'title': "О сайте", 'url_name': 'about'},
    {'title': "Добавить статью", 'url_name': 'add_page'},
    {'title': "Обратная связь", 'url_name': 'contact'},
    {'title': "Войти", 'url_name': 'login'}
]

def show_post(request, post_id):
    return HttpResponse(f"Отображение статьи с id = {post_id}")

def addpage(request):
    return HttpResponse("Добавление статьи")

def contact(request):
    return HttpResponse("Обратная связь")

def login(request):
    return HttpResponse("Авторизация")
```

### Шаг 3: Вывод меню и ссылок в шаблоне (`women/templates/women/index.html`)

```html
<!-- Главное меню -->
<ul>
    <li><a href="{% url 'home' %}">Главная</a></li>
    {% for m in menu %}
        {% if not forloop.last %}<li>{% else %}<li class="last">{% endif %}
            <a href="{% url m.url_name %}">{{ m.title }}</a>
        </li>
    {% endfor %}
</ul>

<!-- Список постов со ссылками -->
<ul>
    {% for p in posts %}
        {% if p.is_published %}
        <li>
            <h2>{{ p.title }}</h2>
            <p>{{ p.content }}</p>
            <p><a href="{% url 'post' p.id %}">Читать пост</a></p>
            {% if not forloop.last %}<hr>{% endif %}
        </li>
        {% endif %}
    {% endfor %}
</ul>
```

## Наследование шаблонов и тег `{% include %}` в Django

Дублирование одинаковых кусков HTML-кода (шапки, меню, подвала) в разных шаблонах нарушает принцип **DRY** (*Don't Repeat Yourself* — не повторяйся). Чтобы избежать повторов, в Django используются **наследование шаблонов** и **включение фрагментов**.

* Ссылка на плейлист по Jinja2: [Уроки по Jinja2](https://rutube.ru/plst/534699).

---

## 1. Наследование шаблонов (`{% extends %}` и `{% block %}`)

Суть механизма: создается один **базовый шаблон** с общим каркасом сайта и специальными «окнами» (блоками), а **дочерние шаблоны** наследуют его и заполняют эти блоки уникальным содержимым.

### Шаг 1: Настройка папки для базового шаблона
Базовый шаблон обычно создают на уровне всего проекта (например, `sitewomen/templates/base.html`). Чтобы Django мог его найти, нужно прописать путь в `settings.py` в списке `DIRS`:

```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [
            BASE_DIR / 'templates',  # Указываем общую папку templates в корне проекта
        ],
        'APP_DIRS': True,
        # ...
    },
]
```

### Шаг 2: Создание базового шаблона (`templates/base.html`)
Выносим сюда общую разметку (структуру документа, меню) и обозначаем изменяемые области с помощью тега **`{% block имя_блока %} ... {% endblock %}`**:

```html
<!DOCTYPE html>
<html>
<head>
    <title>{{ title }}</title>
</head>
<body>

<ul>
    <li><a href="{% url 'home' %}">Главная</a></li>
    {% for m in menu %}
        {% if not forloop.last %}<li>{% else %}<li class="last">{% endif %}
            <a href="{% url m.url_name %}">{{ m.title }}</a>
        </li>
    {% endfor %}
</ul>

<!-- Изменяемый блок для контента конкретной страницы -->
{% block content %} {% endblock %}

</body>
</html>
```

### Шаг 3: Расширение базового шаблона в дочерних файлах
В дочернем шаблоне **самой первой строчкой** указывается тег **`{% extends 'base.html' %}`**, а ниже переопределяются нужные блоки.

**Пример для `women/templates/women/about.html`:**
```html
{% extends 'base.html' %}

{% block content %}
    <h1>{{ title }}</h1>
{% endblock %}
```

**Пример для `women/templates/women/index.html`:**
```html
{% extends 'base.html' %}

{% block content %}
    <h1>{{ title }}</h1>

    <ul>
        {% for p in posts %}
            {% if p.is_published %}
            <li>
                <h2>{{ p.title }}</h2>
                <p>{{ p.content }}</p>
                <p><a href="{% url 'post' p.id %}">Читать пост</a></p>
                {% if not forloop.last %}<hr>{% endif %}
            </li>
            {% endif %}
        {% endfor %}
    </ul>
{% endblock %}
```

---

## 2. Включение шаблонов (`{% include %}`)

Тег **`{% include %}`** позволяет вставить один небольшой HTML-фрагмент в другой шаблон (например, повторяющуюся панель навигации, карточку товара или форму).

### Пример организации и подключения:
1. Создаем отдельный файл фрагмента `women/templates/women/includes/nav.html`:
   ```html
   <nav>
       <a href="#">Актрисы</a> |
       <a href="#">Певицы</a> |
       <a href="#">Спортсменки</a>
   </nav>
   ```
2. Подключаем его в нужном месте (хоть несколько раз) в `index.html`:
   ```html
   {% extends 'base.html' %}

   {% block content %}
       {% include 'women/includes/nav.html' %}

       <h1>{{ title }}</h1>
       <!-- Список постов -->

       {% include 'women/includes/nav.html' %}
   {% endblock %}
   ```

### Управление контекстом при включении:
По умолчанию включаемый шаблон (`nav.html`) имеет доступ ко **всем переменным** родительского шаблона (`index.html`). Это поведение можно настроить:

| Конструкция | Как работает |
| :--- | :--- |
| `{% include 'path/to/file.html' %}` | Передает весь контекст родительского шаблона. |
| `{% include 'path/to/file.html' only %}` | **Запрещает** передачу переменных из родительского шаблона. |
| `{% include 'path/to/file.html' only with title='Заголовок' %}` | Запрещает общий контекст, но передает **только указанные** переменные через `with`. |

---

Ссылка на источник:

https://stepik.org/lesson/1089290/step/2?unit=1099868

https://stepik.org/lesson/1089291/step/2?unit=1099869

https://stepik.org/lesson/1089292/step/2?unit=1099870
