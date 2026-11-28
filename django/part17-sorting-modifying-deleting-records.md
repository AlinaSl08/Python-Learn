# Сортировка, изменение и удаление записей в ORM Django

## 1. Сортировка записей: метод `order_by()`
Используется для упорядочивания записей в выборке (аналог SQL `ORDER BY`).

* **По возрастанию (`ASC`):**
  ```python
  Women.objects.all().order_by('title')
  # Или короче (all() можно опустить):
  Women.objects.order_by('title')
  ```
* **По убыванию (`DESC`):** перед именем поля ставится знак минус (`-`).
  ```python
  Women.objects.order_by('-time_update')
  ```
* **Цепочки методов:** `order_by()` можно комбинировать с фильтрами:
  ```python
  Women.objects.filter(pk__lte=4).order_by('title')
  ```

---

## 2. Глобальные настройки модели: вложенный класс `Meta`
Внутри модели можно создать класс `Meta`, чтобы задать сортировку по умолчанию (применится ко всем выборкам) и добавить индексы для ускорения работы БД:

```python
class Women(models.Model):
    # ... поля модели ...

    class Meta:
        ordering = ['-time_create']  # Сортировка по умолчанию (по убыванию)
        indexes = [
            models.Index(fields=['-time_create']),  # Индекс для ускорения сортировки
        ]

    def __str__(self):
        return self.title
```

---

## 3. Изменение записей

### Способ 1: Изменение одного объекта (`get()` + `save()`)
Подходит, когда нужно поменять конкретную запись:
```python
wu = Women.objects.get(pk=2)
wu.title = 'Марго Робби'
wu.content = 'Биография Марго Робби'
wu.save()  # Выполняет SQL UPDATE для конкретной записи
```

### Способ 2: Массовое обновление (`update()`)
Используется для быстрого изменения сразу группы записей одним SQL-запросом:
```python
# Обновить все записи в таблице
Women.objects.update(is_published=0)

# Обновить записи по фильтру
Women.objects.filter(pk__lte=4).update(is_published=1)
```

> **Важные ограничения `update()`:**
> * Нельзя применять к срезам: `Women.objects.all()[:4].update(...)` — **ошибка**.
> * Нельзя применять к одиночному объекту (результату `get()`): `Women.objects.get(pk=5).update(...)` — **ошибка** (`update()` работает только с `QuerySet`).

---

## 4. Удаление записей: метод `delete()`
Выполняется в два шага: сначала отбираются нужные записи через `filter()`, затем вызывается `delete()`.

```python
wd = Women.objects.filter(pk__gte=5)
wd.delete()  # Удалит все записи с id >= 5
```


---

Ссылка на источник: https://stepik.org/lesson/1089299/step/2?unit=1099877
