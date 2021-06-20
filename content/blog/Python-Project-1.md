+++
title = "Python Project (1) setup.py setup.cfg"
author = "lambda@lambda.tw"
categories = ["python", "project"]
tags = ["python", "project"]
date = "2021-04-22"
description = "Create Your Own Python Project"
featured = ""
featuredalt = ""
featuredpath = ""
linktitle = ""
type = "post"
+++
# Create Project

開專案一直以來都不是一個簡單的事情，專案開的好可以讓後面的開發更有效率，團隊也可以有比較一致開發準則，接下來幾篇文章會大該說說之前研究開 Python 專案的一些經驗

## Create `setup.py` For Install
如果專案要可以被安裝，就需要寫這檔案，裡面可寫詳述該套件需要裝哪些東西，以及一些資訊，建議放在專案的根目錄，這樣別人就可以直接用你的 Git Repo 去安裝你的套件

```python
from pathlib import Path

from setuptools import find_packages, setup

REPO_URL = "https://github.com/lambdaTW/python-project"


def get_version(rel_path):
    for line in Path(rel_path).open().read().splitlines():
        if line.startswith("__version__"):
            delim = '"' if '"' in line else "'"
            return line.split(delim)[1]

    raise RuntimeError("Unable to find version string.")


setup(
    name="python-project",
    version=get_version("src/python-project/__init__.py"),
    description="My python project",
    long_description=Path("README.md").open().read(),
    long_description_content_type="text/markdown",
    python_requires=">=3.8",
    classifiers=[
        "Programming Language :: Python :: 3.8",
    ],
    keywords=["python", "project"],
    url=REPO_URL,
    author="lambdaTW",
    author_email="lambda@lambda.tw",
    license="MIT",
    packages=find_packages(where="src", exclude=("test*",)),
    package_dir={"": "src"},
    # namespace_packages=["lambdatw"],
    install_requires=Path("requirements/base.in").open().read().splitlines(),
    include_package_data=True,
    zip_safe=False,
)
```

## Create `setup.cfg`
`setup.cfg` 原本是拿來做 `setup.py` 的設定檔，但是很多 Python 相關的 Library 也把它當作設定檔，所以除了可以替代 setup.py 的設定也可以用來設定很多東西

### For Install
如果你不想寫很多東西在 `setup.py` 像是 [pytest](https://github.com/pytest-dev/pytest) 的 `setup.py` 就十分乾淨只有以下幾行，其他都寫在其 [`setup.cfg`](https://github.com/pytest-dev/pytest/blob/main/setup.cfg)，由於是設定檔一樣大部分都放根目錄

```python
# setup.py
from setuptools import setup

if __name__ == "__main__":
    setup()
```
下面可以看到 `setup.cfg` 在 `metadata`、`options` 的部分就是 `setup.py` 的參數
```
# setup.cfg
[metadata]
name = pytest
description = pytest: simple powerful testing with Python
long_description = file: README.rst
long_description_content_type = text/x-rst
# ... more than 100 lines
```
### For Setting
由於很多套件都會使用 `setup.cfg` 當作他的設定檔，只要設定相對應的名稱在 `[ ]` 中，就可以使用，至於要設定什麼，就要依照該套件的說明去看，例如：`mypy` 這套件是用來做型別靜態檢查的，我們就可以在 `setup.cfg` 設定他的參數如下
```
# setup.cfg
[mypy]
follow_imports = silent
show_column_numbers = true
ignore_missing_imports = true
```
