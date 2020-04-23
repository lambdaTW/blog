+++
title = "Closure"
author = "lambda@lambda.tw"
categories = ["python"]
date = "2019-04-18"
description = ""
featured = ""
featuredalt = ""
featuredpath = ""
linktitle = ""
type = "post"
tags = ["python", "closure", "django", "django q", "lisp"]
+++

# Closure

## What is closure

Closure 簡單來說，就是某函數在另一個函數內被創造並且參照了創建函數的某些變數，此時該變數會存留於記憶內，儘管創建函數已經結束。

## First time meet to Closure

N年前在學習 Common Lisp 時教學內出現了一個陌生又奇特的技巧，[Closure](https://acl.readthedocs.io/en/latest/zhTW/ch6.html#closures)，以下是他的實做

{{< highlight lisp >}}
(let ((counter 0))
  (defun reset ()
    (setf counter 0))
  (defun stamp ()
    (setf counter (+ counter 1))))
(list (stamp) (stamp) (reset) (stamp))
; (1 2 0 1)
{{< /highlight >}}

為了怕正常人看不懂，以下用 Python 翻譯

{{< highlight python3 >}}
def gen_counter():
    counter = 0

    def reset():
        nonlocal counter
        counter = 0
        return counter

    def stamp():
        nonlocal counter
        counter += 1
        return counter

    return reset, stamp


reset, stamp = gen_counter()
print(stamp())                  # 1
print(stamp())                  # 2
print(reset())                  # 0
print(stamp())                  # 1
{{< /highlight >}}

可以看出在 `gen_counter` 內的 兩個函數 (reset, stamp) 一同共用內部變數 `counter` 儘管 gen_counter 已經回傳並且結束，但是在之後的程式卻還是擁有當初初始化的 count，亦即 `counter` 在記憶體中不會因為 gen_counter 已經回傳就被回收。

# Django Q

## What is Django Q

Django 的 ORM 十分的簡易讓新手們可以簡單的寫出一般的增刪改查，但是如果要用比較進階的搜尋 (SQL WHERE CLAUSE)，例如：正常人都寫的出來的 OR

{{< highlight postgresql >}}
SELECT * FROM post
WHERE content LIKE '%HELLO%' OR title LIKE '%HELLO%';
{{< /highlight >}}

在 Django ORM 就必須要使用 Q 來達成

{{< highlight python3 >}}
from django.db.models import Q


Post.objects.filter(
    Q(content__contains='HELLO') | Q(title__contains='HELLO')
)
{{< /highlight >}}

其中使用 `|` 作為 `OR` 所有的 Q 就如同原本寫 ORM 的條件

# Dancing with

在專案中有一項很常見的功能，就是關鍵字搜尋，很容易想像的是，如果使用一般的 SQL 就用 `LIKE` 慢慢組起來，但是在每個需要搜尋的功能中使用 Django Q 來組建實在很不好維護，以下利用簡化版的真實專案的案例演示 Closure 如何使它看起來更優雅

### models.py

{{< highlight python3 >}}
from django.db import models
from django.contrib.postgres.fields import JSONField


class Content(models.Model):
    title = models.CharField(max_length=200)
    title_pinyin = models.CharField(max_length=400)
    tags = JSONField(default=list)
    content = models.TextField()
    content_pinyin = models.TextField()
{{< /highlight >}}

### sql.py

{{< highlight python3 >}}
from django.db.models import Q


def gen_keywords_search():
    q = Q()

    def icontains(contain=None):
        nonlocal q
        if contain:
            q = (q |
                 Q(title__icontains=contain) |
                 Q(title_pinyin__icontains=contain) |
                 Q(tags__icontains=contain) |
                 Q(content__icontains=contain) |
                 Q(content_pinyin__icontains=contain))
        return q
    return icontains
{{< /highlight >}}

### views.py

{{< highlight python3 >}}
from rest_framework.decorators import api_view

from api.page import pager

from .models import Content
from .sql import gen_keywords_search
from .serializer import PostListSerializer, SearchContentSerializer


@api_view(['post'])
def search(request):
    serializer = SearchContentSerializer(data=request.data)
    serializer.is_valid(raise_exception=True)
    data = serializer.data
    qs = request.user.post_set.all()

    if 'keywords' in data:
        q = gen_keywords_search()
        [q(key) for key in data['keywords'].split()]
        qs = qs.filter(q())

    return pager(
            request,
            qs,
            PostListSerializer,
            page_size=5
        )
{{< /highlight >}}

