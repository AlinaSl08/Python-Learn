# Пользовательские шаблонные теги в Django

В Django можно создавать собственные шаблонные теги двух видов:
1. **Simple tags** (простые теги) — возвращают данные (строку, список, словарь и т.д.).
2. **Inclusion tags** (включающие теги) — формируют собственный HTML-шаблон на основе данных и возвращают готовый фрагмент разметки.

---

## 1. Подготовка структуры проекта

Пользовательские теги должны располагаться внутри пакета `templatetags` в каталоге приложения (на одном уровне с `views.py` и `models.py`). Приложение обязательно должно быть зарегистрировано в `INSTALLED_APPS`.

```text
women/
├── templatetags/
│   ├── __init__.py      # Обязательный файл, делающий папку пакетом
│   └── women_tags.py    # Файл с логикой пользовательских тегов
├── templates/
└── views.py
```

> **Важно:** После создания пакета `templatetags` и нового файла с тегами необходимо **перезапустить тестовый веб-сервер**, иначе Django выдаст ошибку о том, что библиотека тегов не зарегистрирована.

---

## 2. Простые теги (Simple Tags)

### Создание тега (`women/templatetags/women_tags.py`)
Для регистрации тегов создается экземпляр класса `template.Library()`, а сама функция оборачивается в декоратор `@register.simple_tag()`.

```python
from django import template
import women.views as views

register = template.Library()

@register.simple_tag()
def get_categories():
    return views.cats_db
```

*Чтобы задать тегу другое имя для вызова в шаблоне, используется параметр `name`:*
```python
@register.simple_tag(name='getcats')
def get_categories():
    return views.cats_db
```

### Использование в шаблоне (`base.html`)
1. Подключить файл с тегами в начале шаблона:
   ```django
   {% load women_tags %}
   ```
2. Вызвать тег и сохранить результат его работы в переменную с помощью ключевого слова **`as`** (чтобы данные можно было перебрать в цикле, а не просто вывести на экран):
   ```django
   {% get_categories as categories %}
   {# Или, если задано имя getcats: {% getcats as categories %} #}

   <ul id="leftchapters">
       <li class="selected">Все категории</li>
       {% for cat in categories %}
           <li><a href="{% url 'category' cat.id %}">{{ cat.name }}</a></li>
       {% endfor %}
   </ul>
   ```

---

## 3. Включающие теги (Inclusion Tags)

Позволяют отделить кусок HTML-разметки (например, меню или список рубрик) в отдельный мини-шаблон.

### Создание тега (`women/templatetags/women_tags.py`)
Функция должна возвращать **словарь контекста**, переменные из которого станут доступны в указанном в декораторе шаблоне.

```python
@register.inclusion_tag('women/list_categories.html')
def show_categories():
    cats = views.cats_db
    return {"cats": cats}
```

### Создание фрагмента шаблона (`women/templates/women/list_categories.html`)
```django
{% for cat in cats %}
    <li><a href="{% url 'category' cat.id %}">{{ cat.name }}</a></li>
{% endfor %}
```

### Вызов в основном шаблоне (`base.html`)
В нужном месте просто вызывается тег — он сам подставит сформированный HTML:
```django
{% show_categories %}
```

---

## 4. Передача параметров в пользовательские теги

Теги могут принимать аргументы (как позиционные, так и именованные) для гибкой настройки вывода — например, для выделения активной рубрики.

### Обновление функции тега (`women_tags.py`)
```python
@register.inclusion_tag('women/list_categories.html')
def show_categories(cat_selected=0):
    cats = views.cats_db
    return {"cats": cats, "cat_selected": cat_selected}
```

### Обновление шаблона тега (`list_categories.html`)
```django
{% for cat in cats %}
    {% if cat.id == cat_selected %}
        <li class="selected">{{ cat.name }}</li>
    {% else %}
        <li><a href="{% url 'category' cat.id %}">{{ cat.name }}</a></li>
    {% endif %}
{% endfor %}
```

### Вызов тега с параметром в `base.html`
Значение `cat_selected` должно быть предварительно передано в контекст из функции представления (`views.py`).

* **Позиционная передача:**
  ```django
  {% show_categories cat_selected %}
  ```
* **Именованная передача:**
  ```django
  {% show_categories cat_selected=cat_selected %}
  ```
