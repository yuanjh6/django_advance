# django进阶06数据库事务
## 锁  
1.1：乐观锁：  
概念：同一条数据很少会因为并发修改而产生冲突，适用于读多写少的场景。  
实现方式：读取一个字段，执行处理逻辑，当需要更新数据时，再次检查该字段是否和第一次读取一致。如果一致，更新数据，否则不更新，重新读取后再提交。  
1.2：悲观锁：  
概念：当一条数据正在被修改时，不允许其他任何关于这条数据的操作。  
实现方式：读取一个字段之后，加锁，不允许其他任何读、写操作。执行处理逻辑，更新数据完毕后，释放锁。  
1.3：二者比较：  
乐观锁的开销远低于悲观锁  
原因：当 A 锁定了 a 资源，需要 b 资源。而 b 被 B 锁定，正在等待 a 资源。此时，导致出现死锁。也可以通过设置超时来处理这个问题。  
悲观锁可以有效降低冲突后，重试的次数  
乐观锁可以提高响应速度  
在并发比较少时，建议使用乐观锁，减少加锁、释放锁的开销，在并发比较高的时候，建议使用悲观锁。  
  
## 事务  
Django默认每条数据库操作都会被立即提交到数据库。  
会导致一个问题，若有一系列的数据库操作构成，要么全部执行，要么全部都不执行。  
这时需要事务，将一系列数据库操作设置为一个事务，提交给数据库执行。  
Django提供atomic装饰器以开启事务，acomic使用一个参数来指定数据库的名字，若不设置，django会使用系统默认的数据库。  
2.1：整个View函数开启事务：  
```
from django.db import transaction  
  
@transaction.atomic  
def view(request):  
	"""整个view都开启了事务"""  
    func()  
```
2.2：部分函数开启事务：  
```
from django.db import transaction  
  
def view(request):  
    func()  
	  
	# 下面开始执行事务操作,若出现异常操作，会回退到这里  
    with transaction.atomic():  
	# 开启了事务  
        atmice_func()  
```
2.3：不要再事务中处理异常：  
```
from django.db import transaction  
  
def view(request):  
    func()  
    try:  
        with transaction.commit_on_success():  
            do_more_stuff_1() # in transaction  
            try:  
                do_more_stuff_2() # not in transaction  
            except:  
                pass  
            do_more_stuff_3() # in transaction  
    except:  
        pass  
```
当退出原子块时，Django 会查看它是否正常退出或者是否有异常来确定是否提交或者回滚。  
如果你捕获并处理原子块中的异常，可能会隐藏Django中发生问题的事实。  
  
## F函数更新运算  
通常更新数据库的操作，需要将对象读取到内存。在内存中进行修改之后，再写回数据库。  
在内存中的操作，如果存在同时操作的情况，会导致运算逻辑错误。  
F() 函数的作用就是直接生成 SQL 语句，不必将需要更新的对象读取到内存。避免了并发导致的数据不一致问题。  
```
from django.db.models import F  
  
course_obj = Course.objects.get(pk=1)  
course_obj.purchased_quantity = F('purchased_quantity') + 1  
course_obj.save()  
```
## 利用select_for_update函数  
```
select_for_update 使用的是悲观锁，这是数据库层面的，解决并发取数据后再修改的问题方法。  
  
from django.db import transaction  
from django.http import HttpResponse  
  
# 类视图 (并发，悲观锁)  
class MyView(View):  
      
    @transaction.atomic  
    def post(self, request):  
        # select * from 表名 where id=1 for update;    
        # for update 就表示锁,只有获取到锁才会执行查询,否则阻塞等待。  
        obj = 模型类名.objects.select_for_update().get(id=1)  
          
        # 等事务提交后，会自动释放锁。  
          
        return HttpResponse('ok')  
```

## 其他  
django 中操作数据库 均是对 模型类进行CRUD 操作  

| 功能 | mysql命令 | django命令 |
| --- | --- | --- |
| 开启事务 | begin | sid = transaction.savapoint() |
| 提交事务 | commit | transaction.savepoint_commit(sid) |
| 回滚事务 | rollbok | tramsaction\_savepoint\_rollback(sid) |

## 参考  
Django 中事务的处理:https://www.jianshu.com/p/23ccd5f254bf  
Django-框架保证并发时数据一致性:https://blog.csdn.net/Fe_cow/article/details/90267103  
