

从这里开始，我们将真正开始涵盖REST框架的核心。让我们介绍几个基本构建块。

# Request对象
REST框架引入了一个 `Request`对象 用来扩展常规 `HttpRequest` 对象，并提供更灵活的请求解析。`Request` 对象的核心功能是 `request.data` 属性，它类似于 `request.POST` ，但对于使用Web API更有用。

```
request.POST  # Only handles form data.  Only works for 'POST' method.
request.data  # Handles arbitrary data.  Works for 'POST', 'PUT' and 'PATCH' methods.
```

# Response对象
REST框架还引入了一个 `Response` 对象，这是TemplateResponse的一种类型，它接受未渲染的内容并根据内容来确定返回给客户端的正确内容类型。
```
return Response(data)  # Renders to content type as requested by the client.
```

# Status codes
在视图中使用数字HTTP状态代码并不总能明显地读取数据，而且很容易忽略错误代码。REST框架为每个状态代码提供了更显式的标识符，例如状态模块中的 `HTTP_400_BAD_REQUEST` 。在整个过程中使用这些标识符是一个好主意，而不是使用数字标识符。

# 包装API视图
REST框架提供了两个可用于编写API视图的装饰器。

- `@api_view` 用于处理基于函数的视图的装饰器。
- `APIView` 类用于处理基于类的视图。

这些包装器提供了一些功能，例如确保在视图中接收 `Request` 实例，并向 `Response` 对象添加上下文，以便执行内容协议。

包装器还提供一些行为，例如在适当的时候返回 `405 Method Not Allowed` 的响应，并且处理当错误的输入进入 `request.data`时造成的 `ParseError`异常

# 组合起来

好的，让我们继续并开始使用这些新组件来编写一些视图。

我们不再需要 `views.py` 中的 `JSONResponse` 类了，所以删除它。开始重构视图。
```python
from rest_framework import status
from rest_framework.decorators import api_view
from rest_framework.response import Response
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer


@api_view(['GET', 'POST'])
def snippet_list(request):
    """
    List all code snippets, or create a new snippet.
    """
    if request.method == 'GET':
        snippets = Snippet.objects.all()
        serializer = SnippetSerializer(snippets, many=True)
        return Response(serializer.data)

    elif request.method == 'POST':
        serializer = SnippetSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```
我们的实例视图是对前一个示例的改进。它更简洁一些，现在代码与我们使用Forms API非常相似。我们还使用命名的状态代码status code ，这使得响应更明确。

以下是 `views.py` 模块中单个snippet的视图。
```python
@api_view(['GET', 'PUT', 'DELETE'])
def snippet_detail(request, pk):
    """
    Retrieve, update or delete a code snippet.
    """
    try:
        snippet = Snippet.objects.get(pk=pk)
    except Snippet.DoesNotExist:
        return Response(status=status.HTTP_404_NOT_FOUND)

    if request.method == 'GET':
        serializer = SnippetSerializer(snippet)
        return Response(serializer.data)

    elif request.method == 'PUT':
        serializer = SnippetSerializer(snippet, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    elif request.method == 'DELETE':
        snippet.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```
这应该都非常熟悉 - 与使用常规Django视图没有太大区别。

注意，我们不再显式地将请求或响应绑定到给定的content type。`request.data`可以处理传入的json请求，但也可以处理其他格式。类似的，我们将数据返回响应对象，但允许REST框架将响应渲染为正确的content type。

# 向url添加可选格式后缀

为了利用我们的响应不再硬连接到单一内容类型的事实，让我们在API端点中添加对格式后缀的支持。使用格式后缀为我们提供了明确引用给定格式的url，这意味着我们的API能够处理诸如`http://example.com/api/items/4.json` 之类的url。

首先向两个视图都添加format关键字参数，如下所示。
```
def snippet_list(request, format=None):
```
```
def snippet_detail(request, pk, format=None):
```
现在更新 `snippets/urls.py` 文件，在现有url之外附加一组format_suffix_patterns。
```python
from django.conf.urls import url
from rest_framework.urlpatterns import format_suffix_patterns
from snippets import views

urlpatterns = [
    url(r'^snippets/$', views.snippet_list),
    url(r'^snippets/(?P<pk>[0-9]+)$', views.snippet_detail),
]

urlpatterns = format_suffix_patterns(urlpatterns)
```
我们不需要添加这些额外的url模式，但是它提供了一种简单、干净的方式来引用特定的格式。

# 它看起来怎么样?
继续从命令行测试API，就像我们在[教程第1部分](http://www.django-rest-framework.org/tutorial/1-serialization/)中所做的那样。一切工作都非常相似，但是我们在发送无效请求时得到了更好的错误处理。
我们可以像以前一样得到所有snippets的列表。
```
http http://127.0.0.1:8000/snippets/

HTTP/1.1 200 OK
...
[
  {
    "id": 1,
    "title": "",
    "code": "foo = \"bar\"\n",
    "linenos": false,
    "language": "python",
    "style": "friendly"
  },
  {
    "id": 2,
    "title": "",
    "code": "print \"hello, world\"\n",
    "linenos": false,
    "language": "python",
    "style": "friendly"
  }
]
```
我们可以通过使用Accept头来控制返回的响应的格式:
```
http http://127.0.0.1:8000/snippets/ Accept:application/json  # Request JSON
http http://127.0.0.1:8000/snippets/ Accept:text/html         # Request HTML
```
或加上格式后缀:
```
http http://127.0.0.1:8000/snippets.json  # JSON suffix
http http://127.0.0.1:8000/snippets.api   # Browsable API suffix
```
类似地，我们可以使用Content-Type头来控制发送请求的格式。
```
# POST using form data
http --form POST http://127.0.0.1:8000/snippets/ code="print 123"

{
  "id": 3,
  "title": "",
  "code": "print 123",
  "linenos": false,
  "language": "python",
  "style": "friendly"
}

# POST using JSON
http --json POST http://127.0.0.1:8000/snippets/ code="print 456"

{
    "id": 4,
    "title": "",
    "code": "print 456",
    "linenos": false,
    "language": "python",
    "style": "friendly"
}
```
如果您向上面的 `http` 请求添加 `—debug` 切换，您将能够在请求标头中看到请求类型。

现在，通过访问http://127.0.0.1:8000/snippets/，在web浏览器中打开API。

# 可浏览性
因为API根据客户端请求选择响应的内容类型，所以默认情况下，当web浏览器请求该资源时，它将返回一个html格式的资源表示。这允许API返回完全web浏览的HTML表示。


有一个web浏览的API是一个巨大的可用性胜利，并且使开发和使用您的API更加容易。它还极大地降低了其他开发人员想要检查和使用您的API的门槛。


有关可浏览api特性以及如何自定义的更多信息，请参阅[可浏览api](http://www.django-rest-framework.org/topics/browsable-api/)主题。














