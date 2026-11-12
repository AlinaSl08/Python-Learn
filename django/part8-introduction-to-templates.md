# Шаблоны (Templates) в Django

**Шаблоны (Templates)** — это вторая компонента паттерна проектирования **MTV** (Model-Template-View). Они позволяют отделить HTML-разметку от программного кода на Python, чтобы не писать громоздкий HTML-код прямо внутри функций представлений (`views`).

* Синтаксис шаблонов Django очень похож на шаблонизатор **Jinja2** ([плейлист с уроками](https://rutube.ru/plst/534699)).
* Официальная документация: [Django Templates](https://docs.djangoproject.com/en/4.2/topics/templates/).

---

## 1. Где хранить шаблоны (структура папок)

По умолчанию Django ищет шаблоны в папке `templates` внутри каждого зарегистрированного приложения. 

**Важный нюанс:** Если в разных приложениях будут файлы с одинаковыми именами (например, `index.html`), Django возьмет первый попавшийся. Чтобы избежать конфликта (коллизии) имен, внутри папки `templates` принято создавать еще одну вложенную папку **с именем самого приложения**.

**Правильная структура:**
```text
women/
├── templates/
│   └── women/
│       ├── index.html
│       └── about.html
├── urls.py
└── views.py
```

> **Совет:** Все HTML-шаблоны нужно сохранять в кодировке **UTF-8** для корректного отображения кириллицы.

---

## 2. Как подключать и рендерить шаблоны

В Django есть два основных способа загрузить и отобразить шаблон в функции представления (`views.py`):

### Способ 1: `render_to_string` (базовый)
Функция загружает шаблон, обрабатывает его и возвращает в виде обычной строки, которую затем нужно передать в `HttpResponse`.

```python
from django.http import HttpResponse
from django.template.loader import render_to_string

def index(request):
    t = render_to_string('women/index.html')
    return HttpResponse(t)
```

### Способ 2: `render` (рекомендуемый)
Функция-ярлык, которая объединяет в себе рендеринг шаблона и формирование ответа сервера (`HttpResponse`).

```python
from django.shortcuts import render

def index(request):
    return render(request, 'women/index.html')

def about(request):
    return render(request, 'women/about.html')
```

---

## 3. Пример создания страницы «О сайте»

1. **Прописываем маршрут** в `women/urls.py`:
   ```python
   urlpatterns = [
       path('', views.index, name='home'),
       path('about/', views.about, name='about'),
   ]
   ```

2. **Создаем представление** в `women/views.py`:
   ```python
   def about(request):
       return render(request, 'women/about.html')
   ```

3. **Создаем HTML-файл** `women/templates/women/about.html`:
   ```html
   <!DOCTYPE html>
   <html>
   <head>
       <meta charset="UTF-8">
       <title>О сайте</title>
   </head>
   <body>
       <h1>О сайте</h1>
   </body>
   </html>
   ```

---

## 4. Настройки шаблонизатора (`settings.py`)

За работу шаблонов отвечает список `TEMPLATES` в файле конфигурации `settings.py`:

```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [],
        'APP_DIRS': True,
        # ...
    },
]
```

### Основные параметры:
* **`BACKEND`** — указывает используемый шаблонизатор (по умолчанию встроенный `DjangoTemplates`).
* **`DIRS`** — список нестандартных путей, где Django должен искать шаблоны. Поиск всегда начинается **сначала здесь**, а уже потом в папках приложений.
* **`APP_DIRS`** — булево значение (`True` / `False`). Если `True`, Django автоматически ищет папки `templates` внутри всех приложений, перечисленных в `INSTALLED_APPS`.

### Частая ошибка: `TemplateDoesNotExist`
Возникает, когда Django не может найти указанный файл шаблона. Основные причины:
1. Приложение не добавлено в список `INSTALLED_APPS` в `settings.py`.
2. Параметр `APP_DIRS` установлен в значение `False`.
3. Опечатка в пути к шаблону или в названии папки `templates`.

> **Примечание:** Нестандартные пути в `DIRS` можно прописывать через переменную `BASE_DIR` (например, `BASE_DIR / 'women' / 'templates'`), но для стандартных приложений делать этого не нужно — достаточно регистрации в `INSTALLED_APPS` и правильной структуры папок.

---

Ссылка на источник: https://stepik.org/lesson/1089287/step/2?unit=1099865
