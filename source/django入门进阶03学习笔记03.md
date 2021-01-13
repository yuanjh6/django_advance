# django入门进阶03学习笔记03
第一章:模型层

## 1.8 查询操作

一、创建对象

```
>>> from blog.models import Blog  
>>> b = Blog(name='Beatles Blog', tagline='All the latest Beatles news.')  
>>> b.save()  

b = Blog.objects.create(name='Beatles Blog', tagline='All the latest Beatles news.')  
```

二、保存对象

1. 保存外键和多对多字段

```
>>> entry = Entry.objects.get(pk=1)  
>>> cheese_blog = Blog.objects.get(name="Cheddar Talk")  
>>> entry.blog = cheese_blog  
>>> entry.save()  
```
多对多字段的保存稍微有点区别，需要调用一个add()方法，而不是直接给属性赋值，但它不需要调用save方法。如下例所示：

```
>>> from blog.models import Author  
>>> joe = Author.objects.create(name="Joe")  
>>> entry.authors.add(joe)  
```
同时添加多个对象到多对多的字段

```
>>> entry.authors.add(john, paul, george, ringo)  
```

三、检索对象

通过模型的Manager获得QuerySet，每个模型至少具有一个Manager，默认情况下，它被称作objects，可以通过模型类直接调用它，但不能通过模型类的实例调用它，以此实现“表级别”操作和“记录级别”操作的强制分离。

1. 检索所有对象

2. 过滤对象

```
filter(**kwargs)：返回一个根据指定参数查询出来的QuerySet  
exclude(**kwargs)：返回除了根据指定参数查询出来结果的QuerySet  
```
链式过滤

被过滤的QuerySets都是唯一的

```
>>> q1 = Entry.objects.filter(headline__startswith="What")  
>>> q2 = q1.exclude(pub_date__gte=datetime.date.today())  
>>> q3 = q1.filter(pub_date__gte=datetime.date.today())  
```
例子中的q2和q3虽然由q1得来，是q1的子集，但是都是独立自主存在的。同样q1也不会受到q2和q3的影响。

3. 检索单一对象

注意：使用get()方法和使用filter()方法然后通过[0]的方式分片，有着不同的地方。看似两者都是获取单一对象。但是，如果在查询时没有匹配到对象，那么get()方法将抛出DoesNotExist异常。

在使用get()方法查询时，如果结果超过1个，则会抛出MultipleObjectsReturned异常

4. 其它QuerySet方法

5. QuerySet使用限制

注意：不支持负索引！例如 Entry.objects.all()[-1]是不允许的

通常情况，切片操作会返回一个新的QuerySet，并且不会被立刻执行。但是有一个例外，那就是指定步长的时候，查询操作会立刻在数据库内执行，如下：

```
>>> Entry.objects.all()[:10:2]  
```
若要获取单一的对象而不是一个列表（例如，SELECT foo FROM bar LIMIT 1），可以简单地使用索引而不是切片。例如，下面的语句返回数据库中根据标题排序后的第一条Entry：

```
>>> Entry.objects.order_by('headline')[0]  
```
它相当于：

```
>>> Entry.objects.order_by('headline')[0:1].get()  
```
注意：如果没有匹配到对象，那么第一种方法会抛出IndexError异常，而第二种方式会抛出DoesNotExist异常。



也就是说在使用get和切片的时候，要注意查询结果的元素个数。

6. 字段查询

字段查询其实就是filter()、exclude()和get()等方法的关键字参数。 其基本格式是：field__lookuptype=value，注意其中是双下划线。 例如：

```
>>> Entry.objects.filter(pub_date__lte='2006-01-01') 
#　相当于：  
SELECT * FROM blog_entry WHERE pub_date <= '2006-01-01';  
```
有一个例外，那就是ForeignKey字段，你可以为其添加一个“_id”后缀（单下划线）。这种情况下键值是外键模型的主键原生值。例如：

```
>>> Entry.objects.filter(blog_id=4)  
```
exact：默认类型。

iexact：不区分大小写。

contains：表示包含的意思！大小写敏感！

icontains：contains的大小写不敏感模式。

startswith和endswith :以什么开头和以什么结尾。大小写敏感！

istartswith和iendswith是不区分大小写的模式。

7. 跨越关系查询

Django提供了强大并且直观的方式解决跨越关联的查询，它在后台自动执行包含JOIN的SQL语句。要跨越某个关联，只需使用关联的模型字段名称，并使用双下划线分隔，直至你想要的字段（可以链式跨越，无限跨度）。

跨越多值的关系查询

最基本的filter和exclude的关键字参数只有一个，这种情况很好理解。但是当关键字参数有多个，且是跨越外键或者多对多的情况下，那么就比较复杂，让人迷惑了。我们看下面的例子：

```
Blog.objects.filter(entry__headline__contains='Lennon', entry__pub_date__year=2008)  
```
这是一个跨外键、两个过滤参数的查询。此时我们理解两个参数之间属于-与“and”的关系，也就是说，过滤出来的BLog对象对应的entry对象必须同时满足上面两个条件。这点很好理解。也就是说上面要求至少有一个entry同时满足两个条件。

但是，看下面的用法：

```
Blog.objects.filter(entry__headline__contains='Lennon').filter(entry__pub_date__year=2008)  
```
把两个参数拆开，放在两个filter调用里面，按照我们前面说过的链式过滤，这个结果应该和上面的例子一样。可实际上，它不一样，Django在这种情况下，将两个filter之间的关系设计为-或“or”，这真是让人头疼。

多对多关系下的多值查询和外键foreignkey的情况一样。

但是，更头疼的来了，exclude的策略设计的又和filter不一样！

```
Blog.objects.exclude(entry__headline__contains='Lennon',entry__pub_date__year=2008,)  
```
这会排除headline中包含“Lennon”的Entry和在2008年发布的Entry，中间是一个-和“or”的关系！



那么要排除同时满足上面两个条件的对象，该怎么办呢？看下面：

```
Blog.objects.exclude(  
entry=Entry.objects.filter(  
    headline__contains='Lennon',  
    pub_date__year=2008,  
),  
)  
```
（有没有很坑爹的感觉？所以，建议在碰到跨关系的多值查询时，尽量使用Q查询）

8. 使用F表达式引用模型的字段

9. 主键的快捷查询方式：pk

pk就是primary key的缩写。通常情况下，一个模型的主键为“id”，所以下面三个语句的效果一样：

```
>>> Blog.objects.get(id__exact=14) # Explicit form  
>>> Blog.objects.get(id=14) # __exact is implied  
>>> Blog.objects.get(pk=14) # pk implies id__exact  
```
10. 在LIKE语句中转义百分符号和下划线

在原生SQL语句中%符号有特殊的作用。Django帮你自动转义了百分符号和下划线，你可以和普通字符一样使用它们，如下所示：

```
>>> Entry.objects.filter(headline__contains='%')  
# 它和下面的一样  
# SELECT ... WHERE headline LIKE '%\%%';  
```
11. 缓存与查询集

要想高效的利用查询结果，降低数据库负载，你必须善于利用缓存。看下面的例子，这会造成2次实际的数据库操作，加倍数据库的负载，同时由于时间差的问题，可能在两次操作之间数据被删除或修改或添加，导致脏数据的问题：

```
>>> print([e.headline for e in Entry.objects.all()])  
>>> print([e.pub_date for e in Entry.objects.all()])  
```
为了避免上面的问题，好的使用方式如下，这只产生一次实际的查询操作，并且保持了数据的一致性：

```
>>> queryset = Entry.objects.all()  
>>> print([p.headline for p in queryset]) # 提交查询  
>>> print([p.pub_date for p in queryset]) # 重用查询缓存  
```
何时不会被缓存

有一些操作不会缓存QuerySet，例如切片和索引。这就导致这些操作没有缓存可用，每次都会执行实际的数据库查询操作。例如：

```
>>> queryset = Entry.objects.all()  
>>> print(queryset[5]) # 查询数据库  
>>> print(queryset[5]) # 再次查询数据库  
```
但是，如果已经遍历过整个QuerySet，那么就相当于缓存过，后续的操作则会使用缓存，例如：

```
>>> queryset = Entry.objects.all()  
>>> [entry for entry in queryset] # 查询数据库  
>>> print(queryset[5]) # 使用缓存  
>>> print(queryset[5]) # 使用缓存  
```
下面的这些操作都将遍历QuerySet并建立缓存：

```
>>> [entry for entry in queryset]  
>>> bool(queryset)  
>>> entry in queryset  
>>> list(queryset)  
```
注意：简单的打印QuerySet并不会建立缓存，因为__repr__()调用只返回全部查询集的一个切片。

```
Poll.objects.get(  
Q(question__startswith='Who'),  
Q(pub_date=date(2005, 5, 2)) | Q(pub_date=date(2005, 5, 6))  
)  
# 它相当于  
# SELECT * from polls WHERE question LIKE 'Who%'  
AND (pub_date = '2005-05-02' OR pub_date = '2005-05-06')  
```
当关键字参数和Q对象组合使用时，Q对象必须放在前面，如下例子：

```
Poll.objects.get(  
Q(pub_date=date(2005, 5, 2)) | Q(pub_date=date(2005, 5, 6)),question__startswith='Who',)  
```
如果关键字参数放在Q对象的前面，则会报错。


五、比较对象

要比较两个模型实例，只需要使用python提供的双等号比较符就可以了。在后台，其实比较的是两个实例的主键的值。下面两种方法是等同的：

```
>>> some_entry == other_entry  
>>> some_entry.id == other_entry.id  
```
如果模型的主键不叫做“id”也没关系，后台总是会使用正确的主键名字进行比较，例如，如果一个模型的主键的名字是“name”，那么下面是相等的：

```
>>> some_obj == other_obj  
>>> some_obj.name == other_obj.name  
```

六、删除对象

删除对象使用的是对象的delete()方法。该方法将返回被删除对象的总数量和一个字典，字典包含了每种被删除对象的类型和该类型的数量。如下所示：

```
>>> e.delete()  
(1, {'weblog.Entry': 1})  
```
也可以批量删除。每个QuerySet都有一个delete()方法，它能删除该QuerySet的所有成员。例如：

```
>>> Entry.objects.filter(pub_date__year=2005).delete()  
(5, {'webapp.Entry': 5})  
```
当Django删除一个对象时，它默认使用SQL的ON DELETE CASCADE约束，也就是说，任何有外键指向要删除对象的对象将一起被删除。例如：

```
b = Blog.objects.get(pk=1)  
# 下面的动作将删除该条Blog和所有的它关联的Entry对象  
b.delete()  
```
这种级联的行为可以通过的ForeignKey的on_delete参数自定义。

注意，delete()是唯一没有在管理器上暴露出来的方法。这是刻意设计的一个安全机制，用来防止你意外地请求类似Entry.objects.delete()的动作，而不慎删除了所有的条目。如果你确实想删除所有的对象，你必须明确地请求一个完全的查询集，像下面这样：

```
Entry.objects.all().delete()  
```
七、复制模型实例

```
blog = Blog(name='My blog', tagline='Blogging is easy')  
blog.save() # blog.pk == 1  
#  
blog.pk = None  
blog.save() # blog.pk == 2  
```

八、批量更新对象

```
# 更新所有2007年发布的entry的headline  
Entry.objects.filter(pub_date__year=2007).update(headline='Everything is the same')  
```

九、关系的对象

1. 一对多（外键）

正向查询:

反向查询:

使用自定义的反向管理器:

处理关联对象的其它方法:

2. 多对多

3. 一对一

4. 反向关联是如何实现的？

5. 通过关联对象进行查询


十、使用原生SQL语句


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


