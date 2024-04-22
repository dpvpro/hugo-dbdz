---
title: "AttributeError: 'module' object has no attribute 'command' в Python"
date: "2016-02-26"
categories: 
  - "python"
tags: 
  - "attribute error"
  - "python"
---
<!--more-->

При возникновении подобной ошибки стоит проверить, не назван ли ваш скрипт на Python, так же как и библиотека которую вы используете в этом скрипте.

```python
13:31:55 [user@voztd-aleroz Загрузки]$ python click.py --count=3
Traceback (most recent call last):
  File "click.py", line 4, in <module>
    import click
  File "/home/user/Загрузки/click.py", line 6, in <module>
    @click.command()
AttributeError: 'module' object has no attribute 'command'

```

В данном случае нужно переименовать скрипт `lick.py`, во что то еще.
