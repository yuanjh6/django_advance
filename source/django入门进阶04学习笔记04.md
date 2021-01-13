# django入门进阶04学习笔记04

# 第一章，模型层
## 查询集API

一、QuerySet何时被提交

迭代

QuerySet是可迭代的，在首次迭代查询集时执行实际的数据库查询

切片：如果使用切片的”step“参数，Django 将执行数据库查询并返回一个列表。

Pickling/缓存

repr()

len()：当你对QuerySet调用len()时， 将提交数据库操作。

list()：对QuerySet调用list()将强制提交操作entry_list = list(Entry.objects.all())

bool()

二、QuerySet

QuerySet类具有两个公有属性用于内省：

ordered：如果QuerySet是排好序的则为True，否则为False。

db：如果现在执行，则返回使用的数据库。


三、返回新QuerySets的API

方法名	解释

filter()	过滤查询对象。

exclude()	排除满足条件的对象

annotate()	使用聚合函数

order_by()	对查询集进行排序

reverse()	反向排序

distinct()	对查询集去重

values()	返回包含对象具体值的字典的QuerySet

values_list()	与values()类似，只是返回的是元组而不是字典。

none()	创建空的查询集

all()	获取所有的对象

select_related()	附带查询关联对象





## 不返回QuerySets的API

get()	获取单个对象

create()	创建对象，无需save()

get_or_create()	查询对象，如果没有找到就新建对象

update_or_create()	更新对象，如果没有找到就创建对象

count()	统计对象的个数

latest()	获取最近的对象

earliest()	获取最早的对象

first()	获取第一个对象

last()	获取最后一个对象

aggregate()	聚合操作

exists()	判断queryset中是否有对象

update()	批量更新对象

delete()	批量删除对象


# 第二章　视图层
## URL路由基础

四、path转换器

默认情况下，Django内置下面的路径转换器：



str：匹配任何非空字符串，但不含斜杠/，如果你没有专门指定转换器，那么这个是默认使用的；

int：匹配0和正整数，返回一个int类型

slug：可理解为注释、后缀、附属等概念，是url拖在最后的一部分解释性字符。该转换器匹配任何ASCII字符以及连接符和下划线，比如building-your-1st-django-site；

uuid：匹配一个uuid格式的对象。为了防止冲突，规定必须使用破折号，所有字母必须小写，例如075194d3-6885-417e-a8a8-6c931e272f00。返回一个UUID对象；

path：匹配任何非空字符串，重点是可以包含路径分隔符’/‘。这个转换器可以帮助你匹配整个url而不是一段一段的url字符串。要区分path转换器和path()方法。

五、自定义path转换器

其实就是写一个类，并包含下面的成员和属性：



类属性regex：一个字符串形式的正则表达式属性；

to_python(self, value) 方法：一个用来将匹配到的字符串转换为你想要的那个数据类型，并传递给视图函数。如果转换失败，它必须弹出ValueError异常；

to_url(self, value)方法：将Python数据类型转换为一段url的方法，上面方法的反向操作。

例如，新建一个converters.py文件，与urlconf同目录，写个下面的类：

```
class FourDigitYearConverter:  
    regex = '[0-9]{4}'  
  
    def to_python(self, value):  
        return int(value)  
  
    def to_url(self, value):  
        return '%04d' % value  
```
写完类后，在URLconf 中注册，并使用它，如下所示，注册了一个yyyy：

```
from django.urls import register_converter, path  
from . import converters, views  
register_converter(converters.FourDigitYearConverter, 'yyyy')  
  
urlpatterns = [  
    path('articles/2003/', views.special_case_2003),  
    path('articles/<yyyy:year>/', views.year_archive),  
    ...  
]  
```
八、指定视图参数的默认值

```
urlpatterns = [  
    path('blog/', views.page),  
    path('blog/page<int:num>/', views.page),  
]  

# View (in blog/views.py)  

def page(request, num=1):  
    # Output the appropriate page of blog entries, according to num.  
    ...  
```
 
九、自定义错误页面

```
from app import views  
  
urlpatterns = [  
    path('admin/', admin.site.urls),  
]  

# 增加的条目  
handler400 = views.bad_request  
handler403 = views.permission_denied  
handler404 = views.page_not_found  
handler500 = views.error  
```
app/views.py文件中增加四个处理视图：

```
def bad_request(request):  
    return render(request, '400.html')  
  
  
def permission_denied(request):  
    return render(request, '403.html')  
  
  
def page_not_found(request):  
    return render(request, '404.html')  
  
  
def error(request):  
    return render(request, '500.html')  
```

## 路由转发
一、路由转发

```
urlpatterns = [  
    path('<page_slug>-<page_id>/', include([  
        path('history/', views.history),  
        path('edit/', views.edit),  
        path('discuss/', views.discuss),  
        path('permissions/', views.permissions),  
    ])),  
]  
```
三、向视图传递额外的参数

```
urlpatterns = [  
    path('blog/<int:year>/', views.year_archive, {'foo': 'bar'}),  
]  
```
四、传递额外的参数给include()

```
urlpatterns = [  
    path('blog/', include('inner'), {'blog_id': 3}),  
]  
```
## 反向解析和命名空间

一、反向解析URL

```
<a href="{% url 'news-year-archive' 2012 %}">2012 Archive</a>  
return HttpResponseRedirect(reverse('news-year-archive', args=(year,)))  
```
二、URL命名空间

```
urlpatterns = [  
    path('author-polls/', include('polls.urls', namespace='author-polls')),  
    path('publisher-polls/', include('polls.urls', namespace='publisher-polls')),  
]  
```
三、URL命名空间和include的URLconf

```
app_name = 'polls'  
urlpatterns = [  
urlpatterns = [  
    path('polls/', include('polls.urls')),  
]  
  
polls_patterns = ([  
    path('', views.IndexView.as_view(), name='index'),  
    path('<int:pk>/', views.DetailView.as_view(), name='detail'),  
], 'polls')  
urlpatterns = [  
    path('polls/', include(polls_patterns)),  
]  
```

## 视图函数及快捷方式

二、返回错误

```
my_object = get_object_or_404(MyModel, pk=1)  
5. get_list_or_404()  
```
