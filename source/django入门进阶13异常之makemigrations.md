# django入门进阶13异常之makemigrations
makemigrations是django中的常用操作，但是坑也比较多。

坑的主要原因，使用django的manage.py makemigrations，django会加载整个项目，而不仅仅是models.py。而这会引发一系列问题。

## 01，初次初始化时使用了未（来得及）创建的表

比如：缓存类型对象查询到表，报错，进而导致无法执行makemigrations（无法创建对象表）类似先有鸡先有蛋矛盾。

比如:

```
views.py
member_buffer=Member.objects.all()
```
本意是为了当做类似缓存使用（就这意思，代码对没对不重要），但是makemigrations之前是没有表Member的，

要创建表Member就好扫描到views.py，然后就会报错。陷入悖论之中

解决：先注释掉views.py中的这一类查询，然后makemigrations，最后再恢复回来


## 02，非守护线程维持进程导致无法正确退出

和第一个问题类似，不过藏的更深，表现为执行makemigration后，命令未正常结束，而是卡在那里。

```
views.py
class Handler():
    def __init__(self):
        xxxxx
        thread.Thread().start()
handler=Handler()
```
handler此时是个单例，所以Handler里面的__init__会被执行到。意味着如果__init__存在 thread.Thread().start()线程启动语句的话，那么也会被执行，创建出子线程。这会导致makemigrations结束后，无法自动退出，光标在最后一行输出后闪动（实际makemigrations以及成功完成），由于子线程有锁，会阻塞父进程的退出（父进程认为事情没干完，所以也阻塞在那里）。

这个问题迷惑性较强，一般第一反应都是models.py写的有问题。其实和models.py没关系。

本质原因是代码不规范导致的。__init__中是初始化部分，最好别包含线程启动等实际性执行操作。可以用start函数统一启动，这样也丰富编码规范。


## 03，django.db.utils.OperationalError: no such table.

```
python  manage.py  makemigrations  
python   manage.py  migrate  
```
数据库中有一个django_migrations数据表,找到你需要创建数据表的那个name,然后delete,再运行上面两个文件即可解决报错问题.

凡是makemigrations相关问题，有解决的套路

1，如果只修改了部分app则makemigrations和migrate都指定具体app，避免全部app的扫描和更新。降低失败概率。

2，如果修改的app生成表失败，或者和预想不一致，则删除django_migrations数据表中有问题app相关记录，重新makemigrations+migrate生成（数据会丢失，线上别这么干就行了）

参考:https://blog.csdn.net/guyunzh/article/details/79487495    