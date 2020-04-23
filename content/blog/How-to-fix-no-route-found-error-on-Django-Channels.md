+++
title = "How to fix no route found error on Django Channels"
author = "lambda@lambda.tw"
categories = ["Django", "Middleware"]
tags = ["Django", "Django Channels", "Middleware"]
date = "2020-01-25"
description = ""
featured = ""
featuredalt = ""
featuredpath = ""
linktitle = ""
type = "post"
+++
# Issue description

Django Channels 對於不存在的路徑存取，全部會拋出錯誤，而不是一般性的警告處理，所以如果和我一樣在 Djangoo Channels 有裝上 Sentry ，而且伺服器在被惡意嘗試路徑時就會看到一堆 `ValueError: No route found for path '...'.` 的錯誤資訊，好處是知道被打了，壞處就是會噴錢（如果不是自己 Hosting）。



# Fix it

## Make `HandleRouteNotFoundMiddleware` for this issue

```python3
from datetime import datetime
from logging import getLogger
from django.urls.exceptions import Resolver404


logger = getLogger(__file__)


class HandleRouteNotFoundMiddleware:

    def __init__(self, inner):
        self.inner = inner

    def __call__(self, scope):
        try:
            inner_instance = self.inner(scope)
            return inner_instance
        except (Resolver404, ValueError) as e:
            if 'No route found for path' not in str(e) and \
               scope["type"] not in ['http', 'websocket']:
                raise e

            logger.warning(
                f'{datetime.now()} - {e} - {scope}'
            )

            if scope["type"] == "http":
                return self.handle_http_route_error
            elif scope["type"] == "websocket":
                return self.handle_ws_route_error

    async def handle_ws_route_error(self, receive, send):
        await send({"type": "websocket.close"})

    async def handle_http_route_error(self, receive, send):
        await send({
            "type": "http.response.start",
                    "status": 404,
                    "headers": {},
        })
        await send({
            "type": "http.response.body",
            "body": "",
            "more_body": "",
        })
```

## Usage

```python3
from core.middleware import HandleRouteNotFoundMiddleware


application = ProtocolTypeRouter({
    'websocket': AuthMiddlewareStack(
        HandleRouteNotFoundMiddleware(
            URLRouter(
                routing.websocket_urlpatterns
            )
        )
    ),
    'channel': router,
    'http': HandleRouteNotFoundMiddleware(
        URLRouter(
            urlpatterns
        )
    )
})
```

# How it works

## ProtocolTypeRouter

首先我們看到在 Django Channels 我們使用的 Router，可以看到在 `__init__` 時把我們對應表放進去，在被 `Call` 時直接把 `scope` 塞到對應的 `Instance` 一樣是執行該 `Instance` 的 `__call__` （或是該物件已經是 Function 可以直接執行）

```python3
class ProtocolTypeRouter:
    """
    Takes a mapping of protocol type names to other Application instances,
    and dispatches to the right one based on protocol name (or raises an error)
    """

    def __init__(self, application_mapping):
        self.application_mapping = application_mapping
        if "http" not in self.application_mapping:
            self.application_mapping["http"] = AsgiHandler

    def __call__(self, scope):
        if scope["type"] in self.application_mapping:
            return self.application_mapping[scope["type"]](scope)
        else:
            raise ValueError("No application configured for scope type %r" % scope["type"])
```

## URLRouter

依照上面所述說的，我們常在 `Protocol` 對應裡面放入 `URLRouter` 所以我們這裡就只要看 `___call__` 就好了，可以看到在最後 `else` 的部份，會拋出兩個錯誤，也是我們這次主要要修正的問題。

```python3
class URLRouter:
    """
    Routes to different applications/consumers based on the URL path.

    Works with anything that has a ``path`` key, but intended for WebSocket
    and HTTP. Uses Django's django.conf.urls objects for resolution -
    url() or path().
    """
	# ...

    def __call__(self, scope):
        # Get the path
        path = scope.get("path_remaining", scope.get("path", None))
        if path is None:
            raise ValueError("No 'path' key in connection scope, cannot route URLs")
        # Remove leading / to match Django's handling
        path = path.lstrip("/")
        # Run through the routes we have until one matches
        for route in self.routes:
            try:
                match = route_pattern_match(route, path)
                if match:
                    new_path, args, kwargs = match
                    # Add args or kwargs into the scope
                    outer = scope.get("url_route", {})
                    return route.callback(dict(
                        scope,
                        path_remaining=new_path,
                        url_route={
                            "args": outer.get("args", ()) + args,
                            "kwargs": {**outer.get("kwargs", {}), **kwargs},
                        },
                    ))
            except Resolver404 as e:
                pass
        else:
            if "path_remaining" in scope:
                raise Resolver404("No route found for path %r." % path)
            # We are the outermost URLRouter
            raise ValueError("No route found for path %r." % path)
```

## Middleware

我們要想辦法在 `ProtocolTypeRouter` 呼叫 `URLRouter` 前，想辦法抓住這個錯誤，回傳正確找不到路徑的回傳，並且寫下 Log，為此，我們參考 `Django Channels` 的 `Middleware` ，它通常被包在 `URLRouter` 外層，在 `consumer` 前後處理 `scope`，並參考其實做方法，最後自己刻一個專門處理此問題的 `Middleware`。

### How Django Channels middlewares work

首先我們可以看到 `Django Channels` 的 `BaseMiddleware` 在 `__init__` 時，只是把它傳來的值放進 `inner` 這個變數，在被呼叫時 (`__call__`) 回傳一個可以接受 `receive` 和 `send` 的異步函數，這個函數會在連線近來時被建立，且將 `receive` 和 `send` 被丟入 `epoll` 監聽的事件內，供異步伺服器和 client 溝通。

```python3
class BaseMiddleware:

    def __init__(self, inner):
        """
        Middleware constructor - just takes inner application.
        """
        self.inner = inner

    def __call__(self, scope):
        """
        ASGI constructor; can insert things into the scope, but not
        run asynchronous code.
        """
        # Copy scope to stop changes going upstream
        scope = dict(scope)
        # Allow subclasses to change the scope
        self.populate_scope(scope)
        # Call the inner application's init
        inner_instance = self.inner(scope)
        # Partially bind it to our coroutine entrypoint along with the scope
        return partial(self.coroutine_call, inner_instance, scope)

    async def coroutine_call(self, inner_instance, scope, receive, send):
        """
        ASGI coroutine; where we can resolve items in the scope
        (but you can't modify it at the top level here!)
        """
        await self.resolve_scope(scope)
        await inner_instance(receive, send)
```

# PS

以上程式我也回在 [GitHub issue](https://github.com/django/daphne/issues/165#issuecomment-577024577) 上，有任何更好的建議也希望您能發出來，幫助大家。
