#  MySQL索引介绍

## 概念

MySQL官方对索引的定义为：索引（Index）是帮助MySQL高效获取数据的数据结构。可以得到索引的本质：索引是排好序的快速查找数据结构。一般来说索引本身也很大，不可能全部存储在内存中，因此索引往往以索引文件的形式存储在磁盘上。

## 优劣

索引并不是”银弹“。

### 优势

所有的MySql列类型(字段类型)都可以被索引，也就是可以给任意字段设置索引。
大大加快数据的查询速度。       
提高数据检索的效率，降低数据排序的成本

### 劣势

实际上索引也是一张表，该表保存了主键与索引字段，并指向实体表的记录，所以索引列也是要占用空间的。
虽然索引大大提高了查询速度，同时却会降低更新表的速度，如对表进行INSERT、UPDATE和DELETE。因为更新表时，MySQL不仅要保存数据，还要保存一下索引文件每次更新添加了索引列的字段，都会调整因为更新所带来的键值变化后的索引信息。
索引只是提高效率的一个因素，如果你的MySQL有大数据量的表，就需要花时间研究建立最优秀的索引，或者优化查询。

## 使用原则

对经常更新的表就避免对其设置过多的索引，对经常用于查询的字段应该创建索引。
数据量小的表最好不要使用索引，因为由于数据较少，可能查询全部数据花费的时间比遍历索引的时间还要短，索引就可能不会产生优化效果。
在一个列上(字段上)不同值较少的不要建立索引，比如在学生表的"性别"字段上只有男，女两个不同值。相反的，在一个字段上不同值较多的可是建立索引。

下面是更详细使用索引的指导

### 哪些情况需要创建索引

- 主键自动建立唯一索引
- 频繁作为查询条件的字段应该创建索引
- 查询中与其它表关联的字段，外键关系建立索引
- 频繁更新的字段不适合创建索引，因为每次更新不单单是更新了记录，还会更新索引，加重IO负担
- where条件里用不到的字段不创建索引
- 单键/组合索引的选择问题，who？（在高并发下倾向创建组合索引）
- 查询中排序的字段，排序字段若通过索引去访问将大大提高排序速度
- 查询中统计或者分组字段

### 哪些情况不需要创建索引

- 表记录太少
- 经常增删改的表：虽然提高了查询速度，同时却会降低更新表的速度，如对表进行INSERT、UPDATE和DELETE。因为更新表时，MySQL不仅要保存数据，还要保存一下索引文件。
- 数据重复且分布平均的表字段，因此应该只为最经常查询和最经常排序的数据列建立索引。注意，如果某个数据列包含许多重复的内容，为它建立索引就没有太大的实际效果。

# 索引分类与创建

MySQL中分为：普通索引，唯一索引，[主键](https://so.csdn.net/so/search?q=主键&spm=1001.2101.3001.7020)索引，组合索引，和全文索引。

## 普通索引

是最基本的索引，它没有任何限制，允许在定义索引的列中插入重复值和空值，纯粹为了查询数据更快一点。。它有以下几种创建方式：

```mysql
# 1.直接创建索引
CREATE INDEX index_name ON table(column(length))
 
# 2.修改表结构的方式添加索引
ALTER TABLE table_name ADD INDEX index_name ON (column(length))
 
# 3.创建表的时候同时创建索引
CREATE TABLE `table` (
    `id` int(11) NOT NULL AUTO_INCREMENT ,
    `title` char(255) CHARACTER NOT NULL ,
    `content` text CHARACTER NULL ,
    `time` int(10) NULL DEFAULT NULL ,
    PRIMARY KEY (`id`),
    INDEX index_name (title(length))
)
 
# 4.删除索引
DROP INDEX index_name ON table
```

## 唯一索引

与前面的普通索引类似，不同的就是：索引列的值必须唯一，但允许有空值。如果是组合索引，则列值的组合必须唯一。它有以下几种创建方式：

```mysql
# 1.创建唯一索引
CREATE UNIQUE INDEX indexName ON table(column(length))
 
# 2.修改表结构
ALTER TABLE table_name ADD UNIQUE indexName ON (column(length))
 
# 3.创建表的时候直接指定
CREATE TABLE `table` (
    `id` int(11) NOT NULL AUTO_INCREMENT ,
    `title` char(255) CHARACTER NOT NULL ,
    `content` text CHARACTER NULL ,
    `time` int(10) NULL DEFAULT NULL ,
     UNIQUE indexName (title(length))
);
```

唯一索引和主键索引唯一的区别：主键索引不能为null

## 主键索引

是一种特殊的唯一索引，一个表只能有一个主键，不允许有空值。一般是在建表的时候同时创建主键索引：

```mysql
CREATE TABLE `table` (
    `id` int(11) NOT NULL AUTO_INCREMENT ,
    `title` char(255) NOT NULL ,
     PRIMARY KEY (`id`)
);
```

## 组合索引

指多个字段上创建的索引，只有在查询条件中使用了创建索引时的第一个字段，索引才会被使用。使用组合索引时遵循最左前缀集合

```mysql
#添加组合索引
ALTER TABLE `table` ADD INDEX name_city_age (id,name,age); 
 
#创建组合索引
CREATE TABLE tab3(
    id INT(4) NOT NULL,
    name CHAR(20) NOT NULL,
    age INT(3) NOT NULL,
    info VARCHAR(255),
    INDEX multiIdx(id,name,age)
);
```

### 最左前缀

组合索引遵从了最左前缀，利用索引中最左边的列集来匹配行，这样的列集称为最左前缀。例如，这里由id、name和age3个字段构成的索引，索引行中就按id/name/age的顺序存放，索引组合中的字段可以是(id，name，age)、(id，name)或者(id)。如果要查询的字段不构成最左面的前缀原则，那么就不会用索引，比如，age或者（name，age）组合就不会使用索引查询。

##  全文索引

主要用来查找文本中的关键字，而不是直接与索引中的值相比较。fulltext索引跟其它索引大不相同，它更像是一个搜索引擎，而不是简单的where语句的参数匹配。fulltext索引配合match against操作使用，而不是一般的where语句加like。它可以在create table，alter table ，create index使用，不过目前只有char、varchar，text 列上可以创建全文索引。值得一提的是，在数据量较大时候，现将数据放入一个没有全局索引的表中，然后再用CREATE index创建fulltext索引，要比先为一张表建立fulltext然后再将数据写入的速度快很多。
```mysql
# 1.创建表的适合添加全文索引
CREATE TABLE `table` (
    `id` int(11) NOT NULL AUTO_INCREMENT ,
    `title` char(255) CHARACTER NOT NULL ,
    `content` text CHARACTER NULL ,
    `time` int(10) NULL DEFAULT NULL ,
    PRIMARY KEY (`id`),
    FULLTEXT (content)
);
 
# 2.修改表结构添加全文索引
ALTER TABLE article ADD FULLTEXT index_content(content)
 
# 3.直接创建索引
CREATE FULLTEXT INDEX index_content ON article(content)
```



# 索引底层原理——B+树

