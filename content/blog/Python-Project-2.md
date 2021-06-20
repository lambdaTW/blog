+++
title = "Python Project (2) Lint Your Project"
author = "lambda@lambda.tw"
categories = ["python", "project"]
tags = ["python", "project"]
date = "2021-04-23"
description = "Lint your project"
featured = ""
featuredalt = ""
featuredpath = ""
linktitle = ""
type = "post"
+++
# Lint
程式寫出來不難，但是要寫的好看很難很難，在多人協作時就需要一個標準，讓大家可以寫起來差不多，Python 給出了 [PEP (Python Enhancement Proposals) 8 Style Guide for Python Code](https://www.python.org/dev/peps/pep-0008/) 當然正常人不會去看完，所以有工具幫忙就很重要了，這邊介紹 `falke8`, `black`, `isort` 等，工具，方便統一 Coding Style

## Flake8
`Flake8` 可以檢查專案是否符合 `PEP 8` ，然而，該檢查有些過時或是有些規範是相互違反的，這部分就有待團隊自行去規定，此時就需要設定他，`Flake8` 支援 `setup.cfg`, `tox.ini`, 或 `.flake8` 檔，作為設定，此處為了減少檔案，我們使用上次使用到的 `setup.cfg` 作為我們的設定檔

### Usage
裝好該指令後就可以對你的專案進行檢查

```shell-script
pip3 install flake8
flake8
```

### Max Line Length
`PEP 8` 最常被調整的設定就是其每一行程式碼不可超過的字數，該設定原本為每一行不可以超過 79 字元，其由來是以前的 Terminal 長度為 80 字元，為了不讓他換行，所以建議使用該設定，但是現在螢幕都很寬，所以我們可以與團隊溝通調整最大長度

```
# setup.cfg
[flake8]
max-line-length = 88
```

### Exclude
某些檔案可能是由機器自動產生的，你不希望被納入控管，可以使用 `exclude` 的設定讓他跳過該檔案或目錄，這邊以 Django 的 migration file (機器產生)，以及常見虛擬環境目錄 (他人專案) 作為範例

```
#setup.cfg
[flake8]
exclude = */migrations/,env/,venv/
```

### Ignore
在 `PEP 8` 中不少警告是對立的，如：[W503](https://www.flake8rules.com/rules/W503.html)、[W504](https://www.flake8rules.com/rules/W504.html)，他們的 `Best practice` 剛好是對方的 `Anti-pattern` 團隊們可以選擇對於團隊而言比較好的選項，此時我們就可以忽略該筆檢查

```
#setup.cfg
[flake8]
ignore = W503
```

## Black
除了 `PEP 8` 如何讓團隊寫起來的程式碼都更加相同呢？[`Black`](https://github.com/psf/black) 提供了自動格式程式碼的功能，我們可以利用它來檢查以及自動格式化所有程式碼，讓它符合規範，該套件支援的設定檔是：[`pyproject.toml`](https://www.python.org/dev/peps/pep-0518/)

### Note
另一個差不多功能的套件是 [`yapf`](https://github.com/google/yapf)

### Usage
```shell-script
pip install black
# Check
black --check .
# Format
black .
```

### Line Length
和 `Flake8` 一樣的問題這邊我們也把自動排版的每行長度加大
```toml
# pyproject.toml
[tool.black]
line-length = 88
```

### Exclude
一樣我們可以略過不需要排版的資料夾或檔案
```toml
# pyproject.toml
[tool.black]
exclude = '''
/(
  | migrations
)/
'''
```

## isort
在 Python 我們常常 import 多個套件但是多人協作可能就不會把相同的 Library 放在附近，會導致程式碼很混亂，這時候 [isort](https://pycqa.github.io/isort/) 就可以處理我們的問題，他可以檢查 import 順序，也可以自動幫你排序，該套件也使用 [`pyproject.toml`](https://www.python.org/dev/peps/pep-0518/)

### Usage
```shell-script
pip install isort
# Check
isort -c .
# Format
isort .
```

### Configuration
```toml
# pyproject.toml
[tool.isort]
multi_line_output = 3
include_trailing_comma = true
force_grid_wrap = 0
use_parentheses = true
ensure_newline_before_comments = true
line_length = 88
extend_skip = [
    "migrations",
]
```

## Conclusion
介紹了新的設定檔，[`pyproject.toml`](https://www.python.org/dev/peps/pep-0518/) 他是 `PEP 518` 所定義的，未來套件大部分都會支援這個設定檔案，所以只要有套件支援，就放進去吧，至少專案看起來會比較統一，當然不可避免的，比較古老的套件可能還沒有完全支援，那就選擇無可避免的檔案去放他吧，這邊我們優先權，先寫起來， `pyproject.toml > setup.cfg` 
