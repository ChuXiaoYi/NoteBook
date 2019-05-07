!!! note "注意"
    该笔记主要翻译自[官方文档](http://www.django-rest-framework.org/tutorial/1-serialization/)

# 入门

首先，创建新项目
```
django-admin.py startproject tutorial
cd tutorial
```
完成后，我们可以创建一个我们将用于创建简单Web API的应用程序。
```
./manage.py startapp snippets
```
我们需要添加我们的新 `snippets` 应用和 `rest_framework` 应用 `INSTALLED_APPS` 。让我们编辑 `tutorial/settings.py` 文件
```
INSTALLED_APPS = (
    ...
    'rest_framework',
    'snippets.apps.SnippetsConfig',
) 
```
好的，我们已经准备好了。

# 创建一个模型
首先，创建一个 `Snippet` 用于存储代码片段的简单model。继续编辑 `snippets/models.py` 文件
```python
from django.db import models
from pygments.lexers import get_all_lexers
from pygments.styles import get_all_styles

LEXERS = [item for item in get_all_lexers() if item[1]]
LANGUAGE_CHOICES = sorted([(item[1][0], item[0]) for item in LEXERS])
STYLE_CHOICES = sorted((item, item) for item in get_all_styles())


class Snippet(models.Model):
    created = models.DateTimeField(auto_now_add=True)
    title = models.CharField(max_length=100, blank=True, default='')
    code = models.TextField()
    linenos = models.BooleanField(default=False)
    language = models.CharField(choices=LANGUAGE_CHOICES, default='python', max_length=100)
    style = models.CharField(choices=STYLE_CHOICES, default='friendly', max_length=100)

    class Meta:
        ordering = ('created',)
```
我们还需要为我们的代码段模型创建初始迁移，并首次同步数据库。(这里使用的是默认的sqlite)
```
./manage.py makemigrations snippets
./manage.py migrate
```

# 创建一个Serializer类
我们需要做的第一件事就是提供一个序列化和反序列化 `snippet` 实例的方法，并把它放到例如 `json` 中。我们可以通过声明与Django表单非常相似的序列化器来完成此操作。在 `snippets` 名为的目录中创建一个文件 `serializers.py` 并添加以下内容。
```python
from rest_framework import serializers
from snippets.models import Snippet, LANGUAGE_CHOICES, STYLE_CHOICES


class SnippetSerializer(serializers.Serializer):
    id = serializers.IntegerField(read_only=True)
    title = serializers.CharField(required=False, allow_blank=True, max_length=100)
    code = serializers.CharField(style={'base_template': 'textarea.html'})
    linenos = serializers.BooleanField(required=False)
    language = serializers.ChoiceField(choices=LANGUAGE_CHOICES, default='python')
    style = serializers.ChoiceField(choices=STYLE_CHOICES, default='friendly')

    def create(self, validated_data):
        """
        Create and return a new `Snippet` instance, given the validated data.
        """
        return Snippet.objects.create(**validated_data)

    def update(self, instance, validated_data):
        """
        Update and return an existing `Snippet` instance, given the validated data.
        """
        instance.title = validated_data.get('title', instance.title)
        instance.code = validated_data.get('code', instance.code)
        instance.linenos = validated_data.get('linenos', instance.linenos)
        instance.language = validated_data.get('language', instance.language)
        instance.style = validated_data.get('style', instance.style)
        instance.save()
        return instance
```
serializer类定义了序列化/反序列化的字段。`create()` 和 `update()` 方法定义了在调用 `serializer.save()` 时实例如何被创建或修改

serializer类和django `Form` 类很相似，并且在各个字段上包含类似的验证标志，例如 `required` , `max_length` 和 `default`。

字段标志还可以控制在某些情况下应该如何显示序列化程序，例如在渲染HTML时。`{'base_template': 'textarea.html'}` 相当于在django `Form`类中使用 `widget=widgets.Textarea` 。这对于控制可浏览API的显示方式特别有用，我们将在本教程后面看到。

我们实际上也可以通过使用 `ModelSerializer` 类来节省一些时间，我们稍后会看到，但是现在我们先保持我们定义的serializer

# 使用Serializers
在进一步深入之前，我们将熟悉如何使用新的Serializer类。让我们进入Django shell。
```
python manage.py shell
```
好的，一旦我们完成了一些导入，让我们创建几个代码片段来处理。
```python
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer
from rest_framework.renderers import JSONRenderer
from rest_framework.parsers import JSONParser

snippet = Snippet(code='foo = "bar"\n')
snippet.save()

snippet = Snippet(code='print "hello, world"\n')
snippet.save()
```
我们现在有几个snippet实例可供使用。我们来看看序列化其中一个实例。
```
serializer = SnippetSerializer(snippet)
serializer.data
# {'id': 2, 'title': u'', 'code': u'print "hello, world"\n', 'linenos': False, 'language': u'python', 'style': u'friendly'}
```
此时我们已将model实例转换为Python自然数据类型。为了完成序列化过程，我们将数据渲染到 `json`。
```
content = JSONRenderer().render(serializer.data)
content
# '{"id": 2, "title": "", "code": "print \\"hello, world\\"\\n", "linenos": false, "language": "python", "style": "friendly"}'
```
反序列化是类似的。首先，我们将流解析为Python数据类型...
```python
from django.utils.six import BytesIO

stream = BytesIO(content)
data = JSONParser().parse(stream)
```

...然后我们将解析后的数据类型还原为完全填充的对象实例。
```
serializer = SnippetSerializer(data=data)
serializer.is_valid()
# True
serializer.validated_data
# OrderedDict([('title', ''), ('code', 'print "hello, world"\n'), ('linenos', False), ('language', 'python'), ('style', 'friendly')])
serializer.save()
# <Snippet: Snippet object>
```
请注意API与表单的相似程度。当我们开始编写使用序列化器的视图时，相似性应该变得更加明显。

我们还可以序列化querysets而不是model实例。为此，我们只需在序列化类中添加参数 `many=True`
```
serializer = SnippetSerializer(Snippet.objects.all(), many=True)
serializer.data
# [OrderedDict([('id', 1), ('title', u''), ('code', u'foo = "bar"\n'), ('linenos', False), ('language', 'python'), ('style', 'friendly')]), OrderedDict([('id', 2), ('title', u''), ('code', u'print "hello, world"\n'), ('linenos', False), ('language', 'python'), ('style', 'friendly')]), OrderedDict([('id', 3), ('title', u''), ('code', u'print "hello, world"'), ('linenos', False), ('language', 'python'), ('style', 'friendly')])]
```

# 使用ModelSerializers
`SnippetSerializer` 类复制了很多信息，这些都包含在 `Snippet` 模型中。如果我们能够使代码更简洁，那将是很好的。

与Django提供 `Form` 类和 `ModelForm` 类的方式相同，REST框架包括 `Serializer` 类和 `ModelSerializer` 类。

让我们看看使用 `ModelSerializer` 类重构我们的序列化程序。再次打开 `snippets/serializers.py` 文件，并使用以下内容替换 `SnippetSerializer` 类。
```python 
class SnippetSerializer(serializers.ModelSerializer):
    class Meta:
        model = Snippet
        fields = ('id', 'title', 'code', 'linenos', 'language', 'style')
```
序列化程序具有的一个不错的属性是，您可以通过打印其对象来检查序列化程序实例中的所有字段。打开Django shell `python manage.py shell` ，然后尝试以下操作：
```
from snippets.serializers import SnippetSerializer
serializer = SnippetSerializer()
print(repr(serializer))
# SnippetSerializer():
#    id = IntegerField(label='ID', read_only=True)
#    title = CharField(allow_blank=True, max_length=100, required=False)
#    code = CharField(style={'base_template': 'textarea.html'})
#    linenos = BooleanField(required=False)
#    language = ChoiceField(choices=[('Clipper', 'FoxPro'), ('Cucumber', 'Gherkin'), ('RobotFramework', 'RobotFramework'), ('abap', 'ABAP'), ('ada', 'Ada')...
#    style = ChoiceField(choices=[('autumn', 'autumn'), ('borland', 'borland'), ('bw', 'bw'), ('colorful', 'colorful')...
```
重要的是要记住 `ModelSerializer` 类没有做任何特别神奇的事情，它们只是创建序列化程序类的快捷方式：

- 自动确定字段集。
- `create()` 和 `update()`方法的简单默认实现。

# 使用我们的Serializer编写常规Django视图

让我们看看如何使用我们的新Serializer类编写一些API视图。目前我们不会使用任何REST框架的其他功能，我们只会将视图写为常规Django视图。
编辑 `snippets/views.py`文件，然后添加以下内容。
```python
from django.http import HttpResponse, JsonResponse
from django.views.decorators.csrf import csrf_exempt
from rest_framework.renderers import JSONRenderer
from rest_framework.parsers import JSONParser
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer
```

我们API的核心是一个支持列出所有存在的snippets，或创建一个新snippets的视图
```python
@csrf_exempt
def snippet_list(request):
    """
    List all code snippets, or create a new snippet.
    """
    if request.method == 'GET':
        snippets = Snippet.objects.all()
        serializer = SnippetSerializer(snippets, many=True)
        return JsonResponse(serializer.data, safe=False)

    elif request.method == 'POST':
        data = JSONParser().parse(request)
        serializer = SnippetSerializer(data=data)
        if serializer.is_valid():
            serializer.save()
            return JsonResponse(serializer.data, status=201)
        return JsonResponse(serializer.errors, status=400)
```

请注意，因为我们希望能够从没有CSRF令牌的客户端POST到此视图，所以我们需要将视图标记为 `csrf_exempt` 。这不是您通常想要做的事情，REST框架视图实际上使用的行为比这更明智，但它现在可以用于我们的目的

我们还需要一个与单个snippet相对应的视图，并可用于检索，更新或删除代码段。
```python
@csrf_exempt
def snippet_detail(request, pk):
    """
    Retrieve, update or delete a code snippet.
    """
    try:
        snippet = Snippet.objects.get(pk=pk)
    except Snippet.DoesNotExist:
        return HttpResponse(status=404)

    if request.method == 'GET':
        serializer = SnippetSerializer(snippet)
        return JsonResponse(serializer.data)

    elif request.method == 'PUT':
        data = JSONParser().parse(request)
        serializer = SnippetSerializer(snippet, data=data)
        if serializer.is_valid():
            serializer.save()
            return JsonResponse(serializer.data)
        return JsonResponse(serializer.errors, status=400)

    elif request.method == 'DELETE':
        snippet.delete()
        return HttpResponse(status=204)
```
最后，我们需要链接这些视图。创建 `snippets/urls.py` 文件：
```python
from django.conf.urls import url
from snippets import views

urlpatterns = [
    url(r'^snippets/$', views.snippet_list),
    url(r'^snippets/(?P<pk>[0-9]+)/$', views.snippet_detail),
]
```
我们还需要链接tutorial/urls.py文件中的根urlconf ，以包含我们的snippet应用的URL。
```python
from django.conf.urls import url, include

urlpatterns = [
    url(r'^', include('snippets.urls')),
]
```
<font color="red">值得注意的是，目前还有一些我们没有正确处理的边缘情况。如果我们发送错误的json，或者如果使用视图无法处理的方法发出请求，那么我们最终将得到500“服务器错误”响应。不过，现在这样做。</font>

# 测试我们对Web API的第一次尝试
现在我们启动服务器
退出shell。。。
```
quit()
```
并启动django服务器
```
./manage.py runserver

Validating models...

0 errors found
Django version 1.11, using settings 'tutorial.settings'
Development server is running at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```
在另一个终端窗口中，我们可以测试服务器。

我们可以使用[curl](https://curl.haxx.se/)或[httpie](https://github.com/jakubroztocil/httpie#installation)测试我们的API 。Httpie是一个用Python编写的用户友好的http客户端。我们安装一下吧。

您可以使用pip安装httpie：
```
pip install httpie
```

最后，我们可以获取所有snippets的列表信息
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

或者我们可以通过引用其id来获取特定snippet：
```
http http://127.0.0.1:8000/snippets/2/

HTTP/1.1 200 OK
...
{
  "id": 2,
  "title": "",
  "code": "print \"hello, world\"\n",
  "linenos": false,
  "language": "python",
  "style": "friendly"
}
```
同样，您可以通过在Web浏览器中访问这些URL来显示相同​​的json。

# 我们现在在哪
到目前为止我们做得还不错，我们有一个序列化API，感觉非常类似于Django的Forms API，以及一些常规的Django视图。

我们的API视图目前没有做任何特别特别的事情，除了提供json响应之外，还有一些我们仍想清理的错误处理边缘情况，但它是一个正常运行的Web API。

我们将在本教程的第2部分中看到如何开始改进。












