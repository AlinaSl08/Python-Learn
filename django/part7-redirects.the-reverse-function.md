# Перенаправления (редиректы) и именование маршрутов в Django

## 1. Коды перенаправлений (редиректов)

При переносе страниц на новые адреса поисковым системам необходимо сообщать о статусе перемещения, чтобы сохранить позиции сайта в поиске:
* **`301` (Permanent Redirect):** Страница перемещена на новый адрес **постоянно**.
* **`302` (Temporary Redirect):** Страница перемещена на новый адрес **временно**.

---

## 2. Функция `redirect()`

Для выполнения перенаправлений используется функция `redirect` из модуля `django.shortcuts`.

* **Временный редирект (код 302 по умолчанию):**
  ```python
  from django.shortcuts import redirect

  def archive(request, year):
      if year > 2023:
          return redirect('/')  # Перенаправление на главную (код 302)
      return HttpResponse(f"<h1>Архив по годам</h1><p>{year}</p>")
  ```
* **Постоянный редирект (код 301):** Задается с помощью параметра `permanent=True`:
  ```python
  return redirect('/', permanent=True)
  ```
* **Передача представления:** Первым аргументом в `redirect()` можно передать не строку с URL, а ссылку на саму функцию представления (например, `return redirect(index, permanent=True)`).

---

## 3. Классы ответов для редиректов

Вместо универсальной функции `redirect()` можно напрямую использовать специальные классы HTTP-ответов (на которых и основана работа самой `redirect()`):

| Класс ответа | Код статуса | Назначение |
| :--- | :--- | :--- |
| **`HttpResponseRedirect`** | `302` | Временное перенаправление на указанный URL. |
| **`HttpResponsePermanentRedirect`** | `301` | Постоянное перенаправление на указанный URL. |

* *Пример использования:*
  ```python
  from django.http import HttpResponseRedirect

  if year > 2023:
      return HttpResponseRedirect('/')
  ```

---

## 4. Именование маршрутов (параметр `name`)

Указывать прямые URL-адреса внутри кода представлений («хардкодинг») — плохая практика, так как при изменении адреса придется править его во всех файлах. Вместо этого маршрутам присваивают уникальные имена в `urls.py`:

```python
urlpatterns = [
    path('', views.index, name='home'),
    path('cats/<slug:cat_slug>/', views.categories_by_slug, name='cats'),
    path('cats/<int:cat_id>/', views.categories, name='cats_id'),
    re_path(r'^archive/(?P<year>[0-9]{4})/', views.archive, name='archive'),
]
```

* **Использование имени в редиректе:** Теперь вместо пути `'/'` достаточно указать имя маршрута `'home'`. Если URL-путь изменится в `urls.py`, перенаправление продолжит работать автоматически:
  ```python
  return redirect('home', permanent=True)
  ```

---

## 5. Передача параметров и функция `reverse()`

### Редирект с параметрами
Если целевой маршрут принимает динамические параметры (например, `<slug:cat_slug>`), их можно передать в `redirect()` следующими аргументами:
```python
return redirect('cats', 'music')  # Перенаправит на /cats/music/
```

### Функция `reverse()`
Функция `reverse` из модуля `django.urls` позволяет разделить **вычисление URL-адреса** и **само перенаправление**. Она принимает имя маршрута и кортеж/список аргументов (`args`), возвращая готовую строку URL:

```python
from django.urls import reverse
from django.http import HttpResponsePermanentRedirect

url_redirect = reverse('cats', args=('music', ))  # Вычислит '/cats/music/'
return HttpResponsePermanentRedirect(url_redirect) # Или redirect(url_redirect)
```

---
Ссылка на источник: https://stepik.org/lesson/1089286/step/2?unit=1099864
