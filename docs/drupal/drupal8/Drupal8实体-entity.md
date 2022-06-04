title: Drupal8实体(entity)
date: 2017-02-28 20:36:33
categories: Drupal8
tags: [entity]
---

Drupal 的 Entity 是一个很重要的概念，暂时把 Entity 翻译成“实体”吧。正确的理解 Entity，有助于理解 Drupal 信息管理结构，对于使用 Drupal 会有很大的帮助，下面将对这个 Entity 做一个通俗的解释。

## 先简单介绍一下 Entity：
什么是 Entity？Entity 包括 Nodes（节点）, Users（用户）, Taxonomy Terms（术语）, Comments（评论），它们都是实体，只是命名和用途不一样而已。下面翻译一下 Drupal 官网找到的几行定义文字：
An entity type is a base class
一个实体类型是基础类别
A bundle is an extended class
一个集合是扩展类别
A field is a class member, property, variable or field instance (depending on your naming preference)
一个字段是一个类别的成员、属性、元素、或者字段实例（取决于你的命名喜好）
An entity is an object or instance of a base or extended class
一个实体是一个物体，或者是基础类别的实例，或者是扩展类别的实例
更多的就不翻译了，免得看得一头雾水。如果你能理解，下面的内容也不用看了。

## 关于 Entity 的通俗解释：
这是根据我对 Entity 的理解来解释的，我觉得我对 Entity 理解得差不多了，不至于对大家造成多大的误导。
好了，让我们来打个比方，请充分发挥你的联想能力：
把 Entity Type（实体类型） 看做事物——事物有很多种，包括生物、非生物等，比如竹子、蝴蝶、雨花石、空气等。
把 Bundle（集合） 看成某一类具体的事物，比如竹类——竹子其实也分很多种，比如水竹、毛竹、紫竹、苦竹等
把 Entity（实体）看成某一种具体的事物，比如毛竹——毛竹其实其实是由很多部分组成的，比如主干、竹枝、竹叶、根须等
把 Field（字段）看成组成某一具体事物的一部分，比如竹叶
这样也许比较好理解吧。你也可以把 Bundle 看成蝴蝶这个大类，Entity 相当于大鸟翼蝶什么的，字段相当于翅膀、触须什么的。以此类推。

## Content Type（内容类型）：
在括号内加解释太乱了，分两行吧，一行翻译成中文，一行不翻译。我个人觉得不翻译的话好理解一点，但是对英语不熟悉的网友看英语可能不好理解。
翻译的：内容类型其实就是节点这种实体的集合，默认包括标题、内容两个字段。评论也是一个实体，它和节点这种实体是关联的关系。
不翻译：Content Type 其实就是 node 这种 Entity 的 Bundle，默认包括 Title、Body 两个 Filed。

## 应用：
理解了 Entity（实体）之后，自然就是应用了。可以用 Entity Construction Kit (ECK)  这个模块创建新的Entity（实体），或者简单的，就添加新的 Content Type（内容类型）吧。
内容所指的其实不仅仅是文章，图片、音乐、电影、文件、书籍、应用等等都是内容。你想做什么站点就怎么去定义它，它有什么属性就添加几个字段。
把内容理解成某些东西就对了，只要它是信息数据都能用 Drupal 来管理。怎么样？知道 Drupal 可以做很多种网站了吧？其实前面说的主要还是资讯类的网站，更多类型的网站该怎么做，就要发挥你的思维力了。思路决定出路，见识改变思维。