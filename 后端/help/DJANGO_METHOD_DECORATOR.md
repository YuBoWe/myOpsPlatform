# Django `method_decorator` 使用指南

在 Django 的类视图（CBV）中，直接使用装饰器（如 `@login_required`）到类方法上通常会报错。这是因为普通的函数装饰器期望接收 `request` 作为第一个参数，而类方法（如 `get`, `post`）的第一个参数是 `self`。

`method_decorator` 的作用就是将一个**函数装饰器**转换成一个**方法装饰器**。

---

## 1. 基本用法：装饰特定方法

直接在 `get` 或 `post` 方法上使用。

```python
from django.utils.decorators import method_decorator
from django.views.decorators.http import require_GET
from django.views import View

class MyView(View):
    # 将 require_GET 转换并应用到 get 方法
    @method_decorator(require_GET)
    def get(self, request, *args, **kwargs):
        return HttpResponse("Hello, this is a GET request!")
```

---

## 2. 装饰整个类 (推荐做法)

如果你不想进入方法内部修改代码，可以在类定义上方使用 `method_decorator`。此时必须指定 `name` 参数，告诉 Django 你想装饰哪个方法。

```python
from django.contrib.auth.decorators import login_required

@method_decorator(login_required, name='dispatch')
class ProtectedView(View):
    # 装饰 dispatch 意味着装饰了所有 HTTP 方法 (get, post 等)
    def get(self, request):
        return HttpResponse("Secret data")
```

---

## 3. 配合 `require_http_methods` 系列

这是最常见的场景，用于限制视图支持的 HTTP 方法。

```python
from django.views.decorators.http import require_GET, require_POST, require_http_methods

# 案例 A: 只允许 GET
@method_decorator(require_GET, name='dispatch')
class OnlyGetView(View):
    def get(self, request):
        pass

# 案例 B: 同时装饰多个方法
@method_decorator(require_POST, name='post')
class FormView(View):
    def get(self, request):
        return render(request, 'form.html')

    def post(self, request):
        # 只有 POST 请求能进入这里
        return HttpResponse("Success")

---

## 4. 带参数的装饰器用法

如果装饰器本身需要传入参数（如 `require_http_methods` 或 `permission_required`），其用法如下：

```python
from django.views.decorators.http import require_http_methods
from django.contrib.auth.decorators import permission_required

# 案例 A: 限制多个 HTTP 方法
@method_decorator(require_http_methods(["GET", "POST"]), name='dispatch')
class MultiMethodView(View):
    def get(self, request):
        pass
    def post(self, request):
        pass

# 案例 B: 需要特定权限
@method_decorator(permission_required('auth.view_user', raise_exception=True), name='dispatch')
class UserDetailView(View):
    def get(self, request):
        pass
```

> [!NOTE]
> `require_GET` 实际上是 `require_http_methods(["GET"])` 的快捷方式。虽然 `require_GET` 本身不需要括号 and 参数，但它的“底层函数” `require_http_methods` 是需要参数的。

---

## 5. 在 urls.py 中直接使用装饰器

由于 `as_view()` 的执行结果是一个**函数**（闭包），因此你可以直接在 `urls.py` 中使用标准的函数装饰器包裹它，而**无需**使用 `method_decorator`。

```python
from django.urls import path
from django.views.decorators.http import require_GET
from .views import MyView

urlpatterns = [
    # 将 as_view() 作为参数传给 require_GET
    path('api/data/', require_GET(MyView.as_view()), name='data_api'),
]
```

### 适用场景
*   你想在不修改 `views.py` 源码的情况下增加限制。
*   同一个视图类在不同的 URL 中需要不同的装饰器。

---

## 6. 组合多个装饰器

你可以传递一个列表或元组给 `method_decorator`，或者堆叠使用。

```python
decorators = [login_required, require_POST]

@method_decorator(decorators, name='dispatch')
class SecurePostView(View):
    def post(self, request):
        return HttpResponse("Secure POST")

# 或者堆叠
@method_decorator(login_required, name='dispatch')
@method_decorator(require_POST, name='dispatch')
class AnotherView(View):
    pass
```

---

## 7. 为什么装饰 `dispatch`？

在 Django 视图流程中（见 [DJANGO_AS_VIEW_FLOW.md](DJANGO_AS_VIEW_FLOW.md)），`dispatch` 是所有请求的必经之路。

*   **装饰 `dispatch`**：会影响到类中定义的所有方法（GET, POST, PUT 等）。
*   **装饰具体的 `get`**：只影响 GET 请求。

> [!TIP]
> 大多数权限校验（如 `login_required`）建议装饰 `dispatch`。

---

## 8. 常见内置装饰器参考

| 装饰器 | 作用 |
| :--- | :--- |
| `require_GET` | 仅允许 GET 请求 |
| `require_POST` | 仅允许 POST 请求 |
| `require_http_methods(["GET", "POST"])` | 仅允许指定的请求方法 |
| `login_required` | 要求用户必须登录 |
| `permission_required('app.perm')` | 要求用户拥有特定权限 |
| `csrf_exempt` | 免除 CSRF 检查 (通常用于 API) |
| `never_cache` | 禁用响应缓存 |
