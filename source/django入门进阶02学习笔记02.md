# django入门进阶02学习笔记02

基于教程，刘江的博客教程Django教程:https://www.liujiangblog.com/course/django/87

第一章:模型层

## 1.1 模型和字段
FileField

ImageField

FilePathField

UUIDField



## 1.2 关系类型字段
一，一对多

外键要定义在‘多’的一方！

```
parent_comment = models.ForeignKey('self', on_delete=models.CASCADE)  
on_delete  
limit_choices_to  
related_name  
related_query_name  
to_field  
db_constraint  
swappable  
```
二、多对多（ManyToManyField）

symmetrical

默认情况下，Django中的多对多关系是对称的。

Django认为，如果我是你的朋友，那么你也是我的朋友，这是一种对称关系，Django不会为Person模型添加person_set属性用于反向关联。如果你不想使用这种对称关系，可以将symmetrical设置为False，这将强制Django为反向关联添加描述符。

through_fields

Membership模型中包含两个关联Person的外键，Django无法确定到底使用哪个作为和Group关联的对象。所以，在这个例子中，必须显式的指定through_fields参数，用于定义关系。

through_fields参数指定从中间表模型Membership中选择哪两个字段，作为关系连接字段。

db_table

ManyToManyField多对多字段不支持Django内置的validators验证功能。



null参数对ManyToManyField多对多字段无效！设置null=True毫无意义

三、一对一（OneToOneField）


## 1.3 字段的参数

null



该值为True时，Django在数据库用NULL保存空值。默认值为False。对于保存字符串类型数据的字段，请尽量避免将此参数设为True，那样会导致两种‘没有数据’的情况，一种是NULL，另一种是‘空字符串’。

blank

True时，字段可以为空。默认False。和null参数不同的是，null是纯数据库层面的，而blank是验证相关的，它与表单验证是否允许输入框内为空有关，与数据库无关。所以要小心一个null为False，blank为True的字段接收到一个空值可能会出bug或异常。

```
choices  
class Student(models.Model):  
    FRESHMAN = 'FR'  
    SOPHOMORE = 'SO'  
    JUNIOR = 'JR'  
    SENIOR = 'SR'  
    YEAR_IN_SCHOOL_CHOICES = (  
        (FRESHMAN, 'Freshman'),  
        (SOPHOMORE, 'Sophomore'),  
        (JUNIOR, 'Junior'),  
        (SENIOR, 'Senior'),  
    )  
    year_in_school = models.CharField(  
        max_length=2,  
        choices=YEAR_IN_SCHOOL_CHOICES,  
        default=FRESHMAN,  
    )  
class Person(models.Model):  
    SHIRT_SIZES = (  
    ('S', 'Small'),  
    ('M', 'Medium'),  
    ('L', 'Large'),  
    )  
    name = models.CharField(max_length=60)  
    shirt_size = models.CharField(max_length=1, choices=SHIRT_SIZES)  
p = Person(name="Fred Flintstone", shirt_size="L")  
p.shirt_size  
p.get_shirt_size_display()  
db_column  
db_index  
```
该参数接收布尔值。如果为True，数据库将为该字段创建索引。

```
db_tablespace  
default  
```
editable

如果设为False，那么当前字段将不会在admin后台或者其它的ModelForm表单中显示，同时还会被模型验证功能跳过。参数默认值为True。

error_messages

用于自定义错误信息。参数接收字典类型的值。字典的键可以是null、 blank、 invalid、 invalid_choice、 unique和unique_for_date其中的一个。

help_text

额外显示在表单部件上的帮助文本。使用时请注意转义为纯文本，防止脚本攻击。

primary_key

如果你没有给模型的任何字段设置这个参数为True，Django将自动创建一个AutoField自增字段，名为‘id’，并设置为主键。也就是id = models.AutoField(primary_key=True)。

如果你为某个字段设置了primary_key=True，则当前字段变为主键，并关闭Django自动生成id主键的功能。

另外，主键字段不可修改，如果你给某个对象的主键赋个新值实际上是创建一个新对象，并不会修改原来的对象。

unique

设为True时，在整个数据表内该字段的数据不可重复。

注意：对于ManyToManyField和OneToOneField关系类型，该参数无效。

unique_for_date

日期唯一。可能不太好理解。举个栗子，如果你有一个名叫title的字段，并设置了参数unique_for_date="pub_date"，那么Django将不允许有两个模型对象具备同样的title和pub_date。有点类似联合约束。

unique_for_month

同上，只是月份唯一。



unique_for_year

同上，只是年份唯一。

verbose_name

为字段设置一个人类可读，更加直观的别名。



对于每一个字段类型，除了ForeignKey、ManyToManyField和OneToOneField这三个特殊的关系类型，其第一可选位置参数都是verbose_name。如果没指定这个参数，Django会利用字段的属性名自动创建它，并将下划线转换为空格。

下面这个例子的verbose name是"person’s first name":



first_name = models.CharField("person's first name", max_length=30)

下面这个例子的verbose name是"first name":



first_name = models.CharField(max_length=30)

对于外键、多对多和一对一字字段，由于第一个参数需要用来指定关联的模型，因此必须用关键字参数verbose_name来明确指定。如下：



## 1.4 多对多中间表详解


## 1.5 模型的元数据Meta
abstract

app_label

base_manager_name

db_table

db_tablespace

default_manager_name

default_related_name

get_latest_by

managed

order_with_respect_to

ordering

permissions

default_permissions

proxy

required_db_features

required_db_vendor

select_on_save

indexes

unique_together

verbose_name

verbose_name_plural

label

label_lower


## 1.6 模型的继承
抽象基类,多表继承,代理模型

抽象基类中的abstract=True这个元数据不会被继承。也就是说如果想让一个抽象基类的子模型，同样成为一个抽象基类，那你必须显式的在该子模型的Meta中同样声明一个abstract = True；

有一些元数据对抽象基类无效，比如db_table，首先是抽象基类本身不会创建数据表，其次它的所有子类也不会按照这个元数据来设置表名。

警惕related_name和related_query_name参数

```
class Base(models.Model):
    m2m = models.ManyToManyField(
    OtherModel,
    related_name="%(app_label)s_%(class)s_related",
    related_query_name="%(app_label)s_%(class)ss",
    )

    class Meta:
        abstract = True
```
 
如果一个Place对象同时也是一个Restaurant对象，你可以使用小写的子类名，在父类中访问它，


Meta和多表继承

由于父类和子类都在数据库内有物理存在的表，父类的Meta类会对子类造成不确定的影响，因此，Django在这种情况下关闭了子类继承父类的Meta功能。这一点和抽象基类的继承方式有所不同。

但是，还有两个Meta元数据特殊一点，那就是ordering和get_latest_by，这两个参数是会被继承的。因此，如果在多表继承中，你不想让你的子类继承父类的上面两种参数，就必须在子类中显示的指出或重写

多表继承和反向关联

三、 代理模型

声明一个代理模型只需要将Meta中proxy的值设为True。

四、 多重继承

Django的模型体系支持多重继承，就像Python一样。如果多个父类都含有Meta类，则只有第一个父类的会被使用，剩下的会忽略掉。
一般情况，能不要多重继承就不要，尽量让继承关系简单和直接，避免不必要的混乱和复杂。

请注意，继承同时含有相同id主键字段的类将抛出异常。为了解决这个问题，你可以在基类模型中显式的使用AutoField字段。
或者使用一个共同的祖先来持有AutoField字段，并在直接的父类里通过一个OneToOne字段保持与祖先的关系，如下所示：
```
class Piece(models.Model):
    pass

class Article(Piece):
    article_piece = models.OneToOneField(Piece, on_delete=models.CASCADE, parent_link=True)
    ...

class Book(Piece):
    book_piece = models.OneToOneField(Piece, on_delete=models.CASCADE, parent_link=True)
    ...

class BookReview(Book, Article):
    pass
```
警告
在Python语言层面，子类可以拥有和父类相同的属性名，这样会造成覆盖现象。但是对于Django，如果继承的是一个非抽象基类，那么子类与父类之间不可以有相同的字段名！


比如下面是不行的！

```
class A(models.Model):
    name = models.CharField(max_length=30)

class B(A):
    name = models.CharField(max_length=30)
```
## 1.7 用包来组织模型

