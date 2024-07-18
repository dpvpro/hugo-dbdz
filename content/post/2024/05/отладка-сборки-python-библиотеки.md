---
title: Отладка сборки Python библиотеки на Manjaro Linux
date: 2024-05-09T01:55:26+03:00
# description: "Отладка сборки python библиотеки на Manjaro Linux"
categories:
  - python
  - debug
tags:
  - debug python library
  - pytest
---

При очередном обновлении **Manjaro**, обновлении двух `python` библиотек `python-jarowinkler` и `python-async_generator` завершилось ошибками.

Библиотеку `python-async_generator` я удалил, потому что она выглядела старой и уже не поддерживаемой. На библиотеке `python-jarowinkler` я задержался, так как она выглядела поддерживаемой.

Было интересно понять где проблема: в недавнем переход **Arch(Manjaro)** на новый выпуск `python` 3.12 или проблема кроется где то еще.

Я использую **Manjaro**, но пакетная база **Arch** и **Manjaro** очень близка.

<!--more-->

При сборке возникала проблема:
```python
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _

    def find_ruff_bin() -> str:
        """Return the ruff binary path."""

        ruff_exe = "ruff" + sysconfig.get_config_var("EXE")

        path = os.path.join(sysconfig.get_path("scripts"), ruff_exe)
        if os.path.isfile(path):
            return path

        if sys.version_info >= (3, 10):
            user_scheme = sysconfig.get_preferred_scheme("user")
        elif os.name == "nt":
            user_scheme = "nt_user"
        elif sys.platform == "darwin" and sys._framework:
            user_scheme = "osx_framework_user"
        else:
            user_scheme = "posix_user"

        path = os.path.join(sysconfig.get_path("scripts", scheme=user_scheme), ruff_exe)
        if os.path.isfile(path):
            return path

>       raise FileNotFoundError(path)
E       FileNotFoundError: /home/dp/.local/bin/ruff

/usr/lib/python3.12/site-packages/ruff/__main__.py:28: FileNotFoundError

---------- coverage: platform linux, python 3.12.3-final-0 -----------
Name                        Stmts   Miss  Cover
-----------------------------------------------
jarowinkler/__init__.py        22      3    86%
tests/test_JaroWinkler.py      28      1    96%
tests/test_hypothesis.py       66      0   100%
-----------------------------------------------
TOTAL                         116      4    97%

==================================== short test summary info ===========================
FAILED jarowinkler/__init__.py::ruff - FileNotFoundError: /home/dp/.local/bin/ruff
FAILED jarowinkler/__init__.py::ruff::format - FileNotFoundError: /home/dp/.local/bin/ruff
FAILED tests/test_JaroWinkler.py::ruff - FileNotFoundError: /home/dp/.local/bin/ruff
FAILED tests/test_JaroWinkler.py::ruff::format - FileNotFoundError: /home/dp/.local/bin/ruff
FAILED tests/test_hypothesis.py::ruff - FileNotFoundError: /home/dp/.local/bin/ruff
FAILED tests/test_hypothesis.py::ruff::format - FileNotFoundError: /home/dp/.local/bin/ruff
================================== 6 failed, 5 passed in 7.69s ==========================
==> ERROR: A failure occurred in check().
    Aborting...
 -> error making: python-jarowinkler
```

Т.е тесты не находили утилиту `ruff` по пути `/home/dp/.local/bin/`. Совершенно не понятно было почему брался этот путь, а не общесистемный.

Я попробовал вручную собрать пакет, воспроизведя шаги из `PKGBUILD`:

`cat /home/dp/.cache/yay/python-jarowinkler/PKGBUILD`

```bash
# Maintainer: Bao Trinh <qubidt@gmail.com>
# Contributor: Antonio Rojas <arojas@archlinux.org>
# Contributor: Pekka Ristola <pekkarr [at] protonmail [dot] com>

_name=jarowinkler
pkgname=python-$_name
pkgver=2.0.1
pkgrel=2
pkgdesc='A library for fast approximate string matching using Jaro and Jaro-Winkler similarity'
arch=(x86_64)
url='https://github.com/maxbachmann/JaroWinkler'
license=(MIT)
depends=(python python-rapidfuzz)
makedepends=(python-pip python-build python-installer python-setuptools python-scikit-build ninja)
checkdepends=(python-hypothesis python-pytest)
source=("https://files.pythonhosted.org/packages/source/${_name::1}/$_name/$_name-$pkgver.tar.gz")
sha256sums=('7640c79f8d2d5e9eed6691cb49e3018a23b2319daad9a2178df253368b5432b7')

build() {
  cd $_name-$pkgver
  JAROWINKLER_BUILD_EXTENSION=1 \
  python -m build --wheel --no-isolation
}

check() {
  cd $_name-$pkgver
  python -m venv --system-site-packages test-env
  test-env/bin/python -m installer dist/*.whl
  test-env/bin/python -m pytest
}

package() {
  cd $_name-$pkgver
  python -m installer --destdir="$pkgdir" dist/*.whl
  install -Dm644 -t "$pkgdir"/usr/share/licenses/$pkgname LICENSE
```

Для этого переходим в сборочный каталог и начинаем воспроизводить шаги из секций `build()` и `check()`:

`cd /home/dp/.cache/yay/python-jarowinkler/src/jarowinkler-2.0.1`

На шаге `test-env/bin/python -m pytest` проблема воспроизводилась.

Начал смотреть проблемный файл `/usr/lib/python3.12/site-packages/ruff/__main__.py` забивая его отладочными print'ами.

В выводе все равно фигурировали `/home/dp/.local/bin/`.

Запустил тесты от системного `python` и тесты прошли (!), а в выводе не было информации из отладочных print'ов.

```bash
python -m pytest tests/test_hypothesis.py
```

Стало понятно что где то есть какая то разница, но как её определить?

Использовал отладку с добавлением в файл `/usr/lib/python3.12/site-packages/ruff/__main__.py` точек останова:

```python
import pdb
pdb.set_trace()
```

Оттуда было видно что из модуля `sysconfig` приходит разная информация об окружении при запуске общесистемного `python` и из `venv` окружения.

Подробно можно увидеть если запустить следующие команды:

`python -m sysconfig | grep script`
`test-env/bin/python -m sysconfig | grep script`

Я вспомнил что у меня есть тестовый стенд, и попробовал воспроизвести проблему на нём. При установке пакета `python-jarowinkler` проблем не было. Значит есть разница в системном окружении, понял я.

Я установил что есть разница в пакетах `ruff`. К слову, `ruff` это очень быстрый линтер и инструмент для проверки форматирования кода `python`.

На проблемной системе были внешние пакеты связанные с `ruff`:

```bash
> yay -Qs ruff
local/python-pytest-ruff 0.3.2-1
    Pytest plugin to check ruff requirements
local/python-ruff 0.4.3-1
    An extremely fast Python linter, written in Rust
local/ruff 0.4.3-1
    An extremely fast Python linter, written in Rust
```

Там где проблем с установкой не наблюдалось, этих пакетов не было.
Проблема решилась после удаления вышеперечисленных пакетов.

```bash
yay -Rnsu python-pytest-ruff python-ruff ruff
```

Сборка начала проходить `yay -S python-jarowinkler`.

Я вспомнил про первый проблемный пакет - `python-async_generator`. Скорее всего с ним такая же история.

При сборке данного пакета возникала следующая ошибка:

```python
---------- coverage: platform linux, python 3.12.3-final-0 -----------
Name                                             Stmts   Miss Branch BrPart  Cover
----------------------------------------------------------------------------------
async_generator/__init__.py                          4      0      0      0 100.0%
async_generator/_impl.py                           205     22     64      5  87.0%
async_generator/_tests/__init__.py                   0      0      0      0 100.0%
async_generator/_tests/conftest.py                  20      0      8      0 100.0%
async_generator/_tests/test_async_generator.py     589      4    172      7  98.6%
async_generator/_tests/test_util.py                152      2     80      1  98.7%
async_generator/_util.py                            55      1     20      1  97.3%
async_generator/_version.py                          1      0      0      0 100.0%
----------------------------------------------------------------------------------
TOTAL                                             1026     29    344     14  96.3%

================================================================= mypy =================================================================
Found 9 errors in 3 files (checked 10 source files)
======================================================= short test summary info ========================================================
FAILED async_generator/__init__.py::mypy-status
FAILED async_generator/_impl.py::mypy
FAILED docs/source/conf.py::mypy
FAILED setup.py::mypy
============================================== 4 failed, 50 passed, 48 warnings in 36.01s ==============================================
==> ERROR: A failure occurred in check().
    Aborting...
 -> error making: python-async_generator
```

В ошибочных строках фигурируют модули `mypy-status` и `mypy`. Ищем установленные пакеты с такими именами:

```bash
> yay -Qs mypy
local/mypy 1.10.0-1
    Optional static typing for Python 2 and 3 (PEP484)
local/python-mypy_extensions 1.0.0-4
    Experimental type system extensions for programs checked with the mypy typechecker
local/python-pytest-mypy 0.10.3-6
    Mypy static type checker plugin for Pytest
```

Удаляем лишние пакеты - `yay -R python-pytest-mypy mypy`

Пакет `python-mypy_extensions` являлся зависимостью другого пакета, поэтому его оставил.

После удаления сборка начала проходить - `yay -S python-async_generator`
