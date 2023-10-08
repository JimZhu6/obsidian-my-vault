# Python爬虫笔记

> 学http协议，知道哪个协议可以帮你省带宽和时间
>
> 学数据库，不然咋存数据，咋优化？数据库分布式也要了解一点吧？
>
> 学算法，基本的调度算法，爬虫调度也要了解吧？
>
> 学分布式、学redis，分布式总要懂一点，不然爬虫怎么协作呢？
>
> 学JavaScript，不然你怎么看懂人家的数据是怎么处理的，不然你怎么反向解析？
>
> 基本的解密破解知识要懂吧？
>
> 验证码破解要懂吧？机器学习要懂吧？现在破解验证码都上机器学习了！
>
> ios开发要学吧？安卓开发也要学吧？不然怎么反编译人家的app去拿人家隐藏的接口加密算法？
>
>
>
> 1.人家检测出你是爬虫，拉黑你IP （人家究竟是通过你的ua、行为特则 还是别的检测出你是爬虫的？你怎么规避？）
>
> 2.人家给你返回脏数据，你怎么辨认？
>
> 3.对方被你爬死，你怎么设计调度规则？
>
> 4.要求你一天爬完10000w数据，你一台机器带宽有限，你如何用分布式的方式来提高效率？
>
> 5.数据爬回来，要不要清洗？对方的脏数据会不会把原有的数据弄脏？
>
> 6.对方的部分数据没有更新，这些未更新的你也要重新下载吗？怎么识别？怎么优化你的规则？
>
> 7.数据太多，一个数据库放不下，要不要分库？
>
> 8.对方数据是JavaScript渲染，那你怎么抓？要不要上PhantomJS？
>
> 9.对方返回的数据是加密的，你怎么解密？
>
> 10.对方有验证码，你怎么破解？
>
> 11.对方有个APP，你怎么去得到人家的数据接口？
>
> 12.数据爬回来，你怎么展示？怎么可视化？怎么利用？怎么发挥价值？
>
> [参考链接](https://www.zhihu.com/question/265808959/answer/307295445)

## 入门

### 了解Python爬虫所需要的库 -- [requests](http://docs.python-requests.org/zh\_CN/latest/user/quickstart.html)

#### 导入库

```python
import requests
```

#### 发送get请求

```python
>>> r = requests.get('https://api.github.com/events')
```

#### 发送post请求

```python
>>> r = requests.post('http://httpbin.org/post', data = {'key':'value'})
```

#### 发送其他请求

```python
>>> r = requests.put('http://httpbin.org/put', data = {'key':'value'})
>>> r = requests.delete('http://httpbin.org/delete')
>>> r = requests.head('http://httpbin.org/get')
>>> r = requests.options('http://httpbin.org/get')
```

#### 传递带参数的url

`requests`的请求方法里使用`params`接收一个字符串字典或者列表

```python
>>> payload = {'key1': 'value1', 'key2': 'value2'}
# http://httpbin.org/get?key2=value2&key1=value1

>>> payload = {'key1': 'value1', 'key2': ['value2', 'value3']}
# http://httpbin.org/get?key1=value1&key2=value2&key2=value3

>>> r = requests.get("http://httpbin.org/get", params=payload)
```

#### 读取内容

```python
>>> print(r.text)
```

Requests 会自动解码来自服务器的内容。大多数 unicode 字符集都能被无缝地解码。

请求发出后，Requests 会基于 HTTP 头部对响应的编码作出有根据的推测。当你访问 `r.text` 之时，Requests 会使用其推测的文本编码。你可以找出 Requests 使用了什么编码，并且能够使用`r.encoding` 属性来改变它：

```python
>>> r.encoding
'utf-8'
>>> r.encoding = 'ISO-8859-1'
```

如果你改变了编码，每当你访问 `r.text` ，Request 都将会使用 `r.encoding` 的新值。你可能希望在使用特殊逻辑计算出文本的编码的情况下来修改编码。比如 HTTP 和 XML 自身可以指定编码。这样的话，你应该使用 `r.content` 来找到编码，然后设置 `r.encoding` 为相应的编码。这样就能使用正确的编码解析 `r.text` 了。

#### 二进制读取内容

```python
>>> r.content
```

Requests 会自动为你解码 `gzip` 和 `deflate` 传输编码的响应数据。

例如，以请求返回的二进制数据创建一张图片，你可以使用如下代码：

```python
>>> from PIL import Image
>>> from io import BytesIO

>>> i = Image.open(BytesIO(r.content))
```

#### JSON响应内容

Requests 中也有一个内置的 JSON 解码器，助你处理 JSON 数据：

```
>>> import requests

>>> r = requests.get('https://api.github.com/events')
>>> r.json()
[{u'repository': {u'open_issues': 0, u'url': 'https://github.com/...
```

如果 JSON 解码失败， `r.json()` 就会抛出一个异常。例如，响应内容是 401 (Unauthorized)，尝试访问 `r.json()` 将会抛出 `ValueError: No JSON object could be decoded` 异常。

需要注意的是，成功调用 `r.json()` 并**不**意味着响应的成功。有的服务器会在失败的响应中包含一个 JSON 对象（比如 HTTP 500 的错误细节）。这种 JSON 会被解码返回。要检查请求是否成功，请使用 `r.raise_for_status()` 或者检查 `r.status_code` 是否和你的期望相同。

> status\_code可返回状态码

#### 原始响应内容

在罕见的情况下，你可能想获取来自服务器的原始套接字响应，那么你可以访问 `r.raw`。 如果你确实想这么干，那请你确保在初始请求中设置了 `stream=True`。具体你可以这么做：

```python
>>> r = requests.get('https://api.github.com/events', stream=True)
>>> r.raw
<requests.packages.urllib3.response.HTTPResponse object at 0x101194810>
>>> r.raw.read(10)
'\x1f\x8b\x08\x00\x00\x00\x00\x00\x00\x03'
```

但一般情况下，你应该以下面的模式将文本流保存到文件：

```python
with open(filename, 'wb') as fd:
    for chunk in r.iter_content(chunk_size):
        fd.write(chunk)
```

使用 `Response.iter_content` 将会处理大量你直接使用 `Response.raw` 不得不处理的。 当流下载时，上面是优先推荐的获取内容方式。请注意，`chunk_size`可以自由调整为可以更好地适合您的用例的数字。

#### 定制请求头

如果你想为请求添加 HTTP 头部，只要简单地传递一个 `dict` 给 `headers` 参数就可以了。

例如，在前一个示例中我们没有指定 content-type:

```python
>>> url = 'https://api.github.com/some/endpoint'
>>> headers = {'user-agent': 'my-app/0.0.1'}

>>> r = requests.get(url, headers=headers)
```

注意: 定制 header 的优先级低于某些特定的信息源，例如：

* 如果在 `.netrc` 中设置了用户认证信息，使用 headers= 设置的授权就不会生效。而如果设置了 `auth=` 参数，`.netrc` 的设置就无效了。
* 如果被重定向到别的主机，授权 header 就会被删除。
* 代理授权 header 会被 URL 中提供的代理身份覆盖掉。
* 在我们能判断内容长度的情况下，header 的 Content-Length 会被改写。

更进一步讲，Requests 不会基于定制 header 的具体情况改变自己的行为。只不过在最后的请求中，所有的 header 信息都会被传递进去。

注意: 所有的 header 值必须是 `string`、`bytestring` 或者 `unicode`。尽管传递 `unicode` header 也是允许的，但不建议这样做。

#### 更加复杂的 POST 请求

通常，你想要发送一些编码为表单形式的数据——非常像一个 HTML 表单。要实现这个，只需简单地传递一个字典给 data 参数。你的数据字典在发出请求时会自动编码为表单形式：

```python
>>> payload = {'key1': 'value1', 'key2': 'value2'}

>>> r = requests.post("http://httpbin.org/post", data=payload)
>>> print(r.text)
{
  ...
  "form": {
    "key2": "value2",
    "key1": "value1"
  },
  ...
}
```

你还可以为 `data` 参数传入一个元组列表。在表单中多个元素使用同一 key 的时候，这种方式尤其有效：

```python
>>> payload = (('key1', 'value1'), ('key1', 'value2'))
>>> r = requests.post('http://httpbin.org/post', data=payload)
>>> print(r.text)
{
  ...
  "form": {
    "key1": [
      "value1",
      "value2"
    ]
  },
  ...
}
```

很多时候你想要发送的数据并非编码为表单形式的。如果你传递一个 `string` 而不是一个 `dict`，那么数据会被直接发布出去。

例如，Github API v3 接受编码为 JSON 的 POST/PATCH 数据：

```python
>>> import json

>>> url = 'https://api.github.com/some/endpoint'
>>> payload = {'some': 'data'}

>>> r = requests.post(url, data=json.dumps(payload))
```

此处除了可以自行对 `dict` 进行编码，你还可以使用 `json` 参数直接传递，然后它就会被自动编码。这是 2.4.2 版的新加功能：

```python
>>> url = 'https://api.github.com/some/endpoint'
>>> payload = {'some': 'data'}

>>> r = requests.post(url, json=payload)
```

#### 响应状态码

我们可以检测响应状态码：

```python
>>> r = requests.get('http://httpbin.org/get')
>>> r.status_code
200
```

为方便引用，Requests还附带了一个内置的状态码查询对象：

```python
>>> r.status_code == requests.codes.ok
True
```

如果发送了一个错误请求(一个 4XX 客户端错误，或者 5XX 服务器错误响应)，我们可以通过[`Response.raise_for_status()`](http://docs.python-requests.org/zh\_CN/latest/api.html#requests.Response.raise\_for\_status) 来抛出异常：

```python
>>> bad_r = requests.get('http://httpbin.org/status/404')
>>> bad_r.status_code
404

>>> bad_r.raise_for_status()
Traceback (most recent call last):
  File "requests/models.py", line 832, in raise_for_status
    raise http_error
requests.exceptions.HTTPError: 404 Client Error
```

但是，由于我们的例子中 `r` 的 `status_code` 是 `200` ，当我们调用 `raise_for_status()` 时，得到的是：

```python
>>> r.raise_for_status()
None
```

#### Cookie

如果某个响应中包含一些 cookie，你可以快速访问它们：

```python
>>> url = 'http://example.com/some/cookie/setting/url'
>>> r = requests.get(url)

>>> r.cookies['example_cookie_name']
'example_cookie_value'
```

要想发送你的cookies到服务器，可以使用 `cookies` 参数：

```python
>>> url = 'http://httpbin.org/cookies'
>>> cookies = dict(cookies_are='working')

>>> r = requests.get(url, cookies=cookies)
>>> r.text
'{"cookies": {"cookies_are": "working"}}'
```

Cookie 的返回对象为 [`RequestsCookieJar`](http://docs.python-requests.org/zh\_CN/latest/api.html#requests.cookies.RequestsCookieJar)，它的行为和字典类似，但接口更为完整，适合跨域名跨路径使用。你还可以把 Cookie Jar 传到 Requests 中：

```python
>>> jar = requests.cookies.RequestsCookieJar()
>>> jar.set('tasty_cookie', 'yum', domain='httpbin.org', path='/cookies')
>>> jar.set('gross_cookie', 'blech', domain='httpbin.org', path='/elsewhere')
>>> url = 'http://httpbin.org/cookies'
>>> r = requests.get(url, cookies=jar)
>>> r.text
'{"cookies": {"tasty_cookie": "yum"}}'
```

#### 重定向与请求历史

默认情况下，除了 HEAD, Requests 会自动处理所有重定向。

可以使用响应对象的 `history` 方法来追踪重定向。

[`Response.history`](http://docs.python-requests.org/zh\_CN/latest/api.html#requests.Response.history) 是一个 [`Response`](http://docs.python-requests.org/zh\_CN/latest/api.html#requests.Response) 对象的列表，为了完成请求而创建了这些对象。这个对象列表按照从最老到最近的请求进行排序。

例如，Github 将所有的 HTTP 请求重定向到 HTTPS：

```python
>>> r = requests.get('http://github.com')

>>> r.url
'https://github.com/'

>>> r.status_code
200

>>> r.history
[<Response [301]>]
```

如果你使用的是GET、OPTIONS、POST、PUT、PATCH 或者 DELETE，那么你可以通过 `allow_redirects` 参数禁用重定向处理：

```python
>>> r = requests.get('http://github.com', allow_redirects=False)
>>> r.status_code
301
>>> r.history
[]
```

如果你使用了 HEAD，你也可以启用重定向：

```python
>>> r = requests.head('http://github.com', allow_redirects=True)
>>> r.url
'https://github.com/'
>>> r.history
[<Response [301]>]
```

#### 超时

你可以告诉 requests 在经过以 `timeout` 参数设定的秒数时间之后停止等待响应。基本上所有的生产代码都应该使用这一参数。如果不使用，你的程序可能会永远失去响应

```python
>>> requests.get('http://github.com', timeout=0.001)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
requests.exceptions.Timeout: HTTPConnectionPool(host='github.com', port=80): Request timed out. (timeout=0.001)
```

> **注意:**`timeout` 仅对连接过程有效，与响应体的下载无关。 `timeout` 并不是整个下载响应的时间限制，而是如果服务器在 `timeout` 秒内没有应答，将会引发一个异常（更精确地说，是在`timeout` 秒内没有从基础套接字上接收到任何字节的数据时）If no timeout is specified explicitly, requests do not time out.

**连接**超时指的是在你的客户端实现到远端机器端口的连接时（对应的是`connect()`\_），Request 会等待的秒数。一个很好的实践方法是把连接超时设为比 3 的倍数略大的一个数值，因为 [TCP 数据包重传窗口 (TCP packet retransmission window)](http://www.hjp.at/doc/rfc/rfc2988.txt) 的默认大小是 3。

#### 错误与异常

遇到网络问题（如：DNS 查询失败、拒绝连接等）时，Requests 会抛出一个 `ConnectionError` 异常。

如果 HTTP 请求返回了不成功的状态码， [`Response.raise_for_status()`](http://docs.python-requests.org/zh\_CN/latest/api.html#requests.Response.raise\_for\_status) 会抛出一个 `HTTPError`异常。

若请求超时，则抛出一个 `Timeout` 异常。

若请求超过了设定的最大重定向次数，则会抛出一个 `TooManyRedirects` 异常。

所有Requests显式抛出的异常都继承自 `requests.exceptions.RequestException`
