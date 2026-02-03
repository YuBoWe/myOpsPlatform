# Django 中间件 (Middleware) 流程详解

Django 中间件是一个轻量级、底层的 "插件" 系统，用于在全局范围内介入 Django 的请求 (Request) 和响应 (Response) 处理过程。

## 1. 核心机制

在 `settings.py` 的 `MIDDLEWARE` 列表中配置中间件。虽然它们只是普通的 Python 类，但其在列表中的**顺序至关重要**。

中间件就像洋葱皮一样包裹着视图：
*   **请求阶段 (Request)**: 每一个请求都会**自上而下**穿过 `MIDDLEWARE` 列表。
*   **视图层 (View)**: 请求到达并被视图处理。
*   **响应阶段 (Response)**: 响应会**自下而上**穿过 `MIDDLEWARE` 列表返回给用户。

### 执行流程图

```mermaid
graph TD
    User([用户请求]) --> M1_Pre[Middleware A (Pre)]
    M1_Pre --> M2_Pre[Middleware B (Pre)]
    M2_Pre --> View[执行 View 视图]
    View --> M2_Post[Middleware B (Post)]
    M2_Post --> M1_Post[Middleware A (Post)]
    M1_Post --> User([返回响应])

    subgraph "Request Phase (Top-Down)"
    M1_Pre
    M2_Pre
    end

    subgraph "Response Phase (Bottom-Up)"
    M2_Post
    M1_Post
    end
```

## 2. 标准中间件结构

自 Django 1.10 起，中间件的标准写法是一个接受 `get_response` 回调的类。

```python
class SimpleMiddleware:
    def __init__(self, get_response):
        # 这里的 get_response可能是后续的中间件，或者是最终的视图函数
        self.get_response = get_response
        # One-time configuration and initialization.

    def __call__(self, request):
        # type: (HttpRequest) -> HttpResponse
        
        # ----------------------------------------------------
        # 1. 请求预处理 (Process Request)
        # 在视图(或下一个中间件)被调用之前执行的代码
        # ----------------------------------------------------
        print("About to process request (Top-Down)")

        # 调用下一个中间件或最终视图
        response = self.get_response(request)

        # ----------------------------------------------------
        # 2. 响应后处理 (Process Response)
        # 在视图调用之后执行的代码
        # ----------------------------------------------------
        print("Finished processing request (Bottom-Up)")

        return response
```

## 3. 高级钩子方法 (Hooks)

除了 `__call__` 之外，中间件还可以定义以下方法来介入特定流程：

### `process_view(request, view_func, view_args, view_kwargs)`
*   **触发时机**: 在 Django URL 路由匹配之后，但在调用视图函数之前。
*   **执行顺序**: 自上而下。
*   **返回值**:
    *   `None`: 继续执行后续中间件的 `process_view`，最终执行视图。
    *   `HttpResponse` 对象: **拦截**。不再执行后续的 `process_view` 和视图，直接进入响应阶段（从负责该响应的中间件开始自下而上返回）。

### `process_exception(request, exception)`
*   **触发时机**: 视图函数抛出异常时。
*   **执行顺序**: **自下而上** (与 Request 相反，类似冒泡)。
*   **用途**: 用于全局异常捕获、错误日志记录或返回自定义错误页。
*   **返回值**:
    *   `None`: 异常继续向上传递。
    *   `HttpResponse`: 异常被捕获处理，不再抛出，流程转为正常的响应返回。

### `process_template_response(request, response)`
*   **触发时机**: 视图返回的对象是 `TemplateResponse` (或其子类) 时。
*   **执行顺序**: 自下而上。
*   **用途**: 在渲染模板前修改上下文数据 (Context) 或装饰器。

## 4. 常用内置中间件举例

*   **`django.middleware.security.SecurityMiddleware`**: 通常排第一，负责处理请求/响应的安全增强（如 SSL 重定向）。
*   **`django.contrib.sessions.middleware.SessionMiddleware`**: 处理 session，在 request 上添加 `request.session`。
*   **`django.middleware.common.CommonMiddleware`**: 处理 URL 重写（如 `APPEND_SLASH`）、User-Agent 检查等。
*   **`django.contrib.auth.middleware.AuthenticationMiddleware`**: 依赖 SessionMiddleware，在 request 上添加 `request.user`。

## 5. 编写注意事项

1.  **Lightweight**: 中间件在每次请求都会运行，务必保持轻量，避免进行耗时的数据库查询或IO操作，否则会严重拖慢全站性能。
2.  **Order dependencies**: 注意依赖关系。例如 `AuthenticationMiddleware` 必须放在 `SessionMiddleware` 之后，因为它需要 session 来获取用户信息。
