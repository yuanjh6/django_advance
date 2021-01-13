# django入门进阶06静态文件和模板
## 静态文件和模板

静态文件:css,js,image,如果作为纯粹的web应用来看，静态文件的响应并不属于web应用范畴，**因为静态文件不涉及业务逻辑，也不需开发业务代码**。但几乎100%的web应用都支持对静态文件的直接访问。为何？主要是web应用基本上必然依赖css,js,img等静态资源，我们不可能固执的开发一个纯粹的web应用，**只支持从url里进入视图函数，也只能从视图函数返回内容**（简单来说所有请求路径都必须体现在url_route和view视图中）。而要求用户独立开启静态资源的请求处理服务。所以，先把静态资源服务和包含业务逻辑的web应用独立认识，二者并不相同，但强相关。**其本身是独立于应用的**

模板:在代码中response渲染中使用的，可以看做view视图的组成部分.所以没有独立url配置，**依赖view视图存在**，属于应用一部分，包含了业务逻辑（需要渲染），这一点也可以从配置上看出来。


## django配置静态文件

**STATIC_ROOT**

所有的静态文静聚合的目录,

STATIC_ROOT要写成绝对地址,在这里,比如我的项目mysite是/home/mysite/ 那么STATIC_ROOT 为 /home/mysite/collect_static/

```
python manage.py collectstatic # 把所有的static文件都复制到STATIC_ROOT文件夹下
```
可见,STATIC_ROOT 是在部署的时候才发挥作用。简单来说，开发时可能按应用分散开发，**所以css,js等都是按应用分散到各自应用目录下**，但是在部署时，**不大可能在前置的web服务器中配置多个分散的应用目录**（而且也会带来高耦合问题）。所以需要“整合”到一个目录，然后nginx的服务器指向此目录即可。



**STATICFILES_DIRS**

开发时静态资源路径，前面说过部署时会使用STATIC_ROOT作为静态资源路径。但开发时一般不会有独立的静态资源服务器，所以允许分散性配置，同时，这个也是生成STATIC_ROOT里文件的“源路径”。

STATICFILES_DIRS里文件夹又份两种，
app应用独立占用：一种就是在每个app里面新建一个static文件夹,将静态文件放到里面,在加载静态文件时,比如要在模板中用到静态文件,django会自动在每个app里面搜索static文件夹(所以,不要把文件夹的名字写错哦, 否则django就找不到你的文件夹了)

项目（多个或所有应用）共享：就是在所有的app文件外面,建立一个公共的文件夹, 因为有些静态文件不是某个app独有的,那么就可以把它放到一个公共文件夹里面,方便管理(注意,建立一个公共的静态文件的文件夹只是一种易于管理的做法,但是不是必须的,app是可以跨app应用静态文件的,因为最后所有的静态文件都会在STATIC_ROOT里面存在) 那现在的问题是如何让django知道你把一些静态文件放到app以外的公共文件夹中呢,那就需要配置STATICFILES_DIRS了

```
STATICFILES_DIRS = ( os.path.join(BASE_DIR, 'common_static'), ) 
```
STATICFILES_DIRS告诉django,首先到STATICFILES_DIRS里面寻找静态文件,其次再到各个app的static文件夹里面找(注意, django查找静态文件是惰性查找,查找到第一个,就停止查找了)


**STATIC_URL**

那么到此为止,静态文件的机制就可以运作了,但是有一个问题,我能不能通过url直接访问我在项目中的静态文件呢,答案肯定是啦,但是,注意,你是在浏览器是访问,你不可能输入你的静态文件的本地绝对地址吧,比如我的一种图片的本地地址为 /home/mysite/common_static/myapp/photo.png 那么别人不可能在浏览器上直接输入: http://192.168.1.2:8000/home/mysite/common_static/myapp/photo.png 这样子,浏览器会报错, 没有该页面 那么django是如何让浏览器也可以访问服务器上的静态文件呢,前面已经说了,直接访问服务器本地的地址是不行的,那就需要一个映射,django利用STATIC_URL来让浏览器可以直接访问静态文件,比如:

```
STATIC_URL = '/static/' 
```
那么可以在浏览器上输入: http://192.168.1.2:8000/static/common_static/myapp/photo.png 那么就相当与访问/home/mysite/common_static/myap/photo.png


开发环境下静态文件都是通过Django自带的web服务器来处理的（这样会更方面）。如果把DEBUG设置成False，那么Django自带的web服务器自然不处理静态文件了，静态文件都交给nginx，apache来处理吧（这样会更高效）。

另外，Django提供了一个findstatic命令来查找指定的静态文件所在的目录，例如：

```
D:\TestDjango>python manage.py findstatic Chrome.jpg ('D:/TestDjango/TestDjango/templates',)
```

## django配置模板

模板相关配置

```
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'templates')], # 公共共享文件夹
        'APP_DIRS': True,# 检索app里面的templates文件夹
    }
]
```
模板相关配置只有2项，实际这2项目，如果**和静态文件比对起来，只对应了1项**，就是STATICFILES_DIRS

所以:TEMPLATES（原始）并不支持从url直接respones，而是需要经过url_route,view处理，view_response(html)，

所以**模板不需要独立路由配置**，其路由有具体调用其的view决定，也就没有STATIC_URL类似的配置。当然也不存在静态资源统一保存的需求，也就不需要STATIC_ROOT类似的配置。


**Django 模板查找机制**

Django 查找模板的过程是在每个 app 的 templates 文件夹中找（而不只是当前 app 中的代码只在当前的 app 的 templates 文件夹中找）。

各个 app 的 templates 形成一个文件夹列表，Django 遍历这个列表，一个个文件夹进行查找，当在某一个文件夹找到的时候就停止，所有的都遍历完了还找不到指定的模板的时候就是 Template Not Found （过程类似于Python找包）。这样设计有利当然也有弊，有利是的地方是一个app可以用另一个app的模板文件，弊是有可能会找错了。所以我们使用的时候在 templates 中建立一个 app 同名的文件夹，这样就好了。

这就需要把每个**app中的 templates 文件夹中再建一个 app 的名称**，仅和该app相关的模板放在 app/templates/app/ 目录下面



**不使用view直接跳转**

```
urlpatterns = [ path('about/', TemplateView.as_view(template_name="about.html")), ]
```
## 参考

Django 模板:https://code.ziqiangxuetang.com/django/django-template.html

django---不使用view，直接从Url转到html：https://blog.csdn.net/weixin_30325487/article/details/97544472

Django关于访问静态文件总结：https://blog.csdn.net/WaitForFree/article/details/39815507

django 静态文件之配置说明(Django中STATIC_URL、STATIC_ROOT、STATICFILES_DIRS区别关系)：https://blog.csdn.net/alxandral_brother/article/details/52202270

