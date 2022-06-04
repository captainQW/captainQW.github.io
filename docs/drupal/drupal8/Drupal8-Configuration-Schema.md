title: Drupal8 configuration schema
date: 2016-10-31 16:39:15
categories: Drupal8
tags: [configuration schema]
---

Drupal 8中configuration schema是用来描述配置文件的结构的. 它应用于:
定型配置以确保数据一致性
(see StorableConfigBase::castValue())

自动持久化实体属性的配置
 (see ConfigEntityBase::toArray())

自动生成用户界面的翻译配置
 (see the core module)

1、一个简单的示例
如下:
```php
config/install/my_module.settings.yml
type: warning
message: 'Hello!'
langcode: en

config/schema/my_module.schema.yml
my_module.settings:
  type: config_object
  mapping:
    type:
	  type: string
	  label: 'Message type'
	message:
	  type: label
	  label: 'Message text'
	langcode:
	  type: string
	  label: 'Language code'
```

需要说明的是，my_module.schema.yml我们可以认为是一个自定义的数据结构体，而my_module.settings.yml则是给这个自定义的数据结构体定义了默认值，
在my_module.schema.yml中我们可以看到这个数据结构体中定义了一个mapping结构体(可以简单的认为是一个数组类型)，这个数组有三个key，分别是type、message、langcode，
前面说了my_module.settings.yml是给my_module.schema.yml这个数据结构体定义了默认值，我们可以看到在my_module.settings.yml中type的默认值是warning,message的默认值
是'Hello!',langcode的默认值是en,值的注意的一点是，一个schema.yml文件可以定义多个数据结构体，但是每一个的数据结构体的root元素必须与settings.yml文件的文件名一致，
比如上面的两个文件settings.yml文件的文件名是my_module.settings.yml，my_module.schema.yml的root元素的名称就是my_module.settings。

2、schema中常用的数据类型
基本类型：
boolean，
integer，
float，
string，
uri，
email

列表类型：
mapping(知道key是什么)，
sequence(不知道key的具体值)

常见的子类型：
label: 短标签
text: 长文本
config_object: object根元素类型
config_entity: entity根元素类型

子类型化
所有configuration schema的子类型都是从现有的类型演化而来. 
早些时候的简单示例子类型config_object (mapping的一个子类型) 可以进一步自定义key。
configuration schema根节点的数据类型一般都是config_object或者config_entity。

3、关于动态类型[%parent]
确切的数据类型可能不会知道，而且还有类型取决于数据的可能性，configuration schema同样支持基于数据的类型。假设Message的类型可能取决于数据的类型：无论Message是一个列表
或者是一个简单的警告。让我们用 'multiple' 来定义列表情况,用 'warning' 来表示一个单行的警告。
```php
config/install/my_module.message.single.yml
type: warning
message: 'Hello!'
langcode: en

config/install/my_module.message.multiple.yml
type: multiple
message:
 -'Hello!'
 -'Hi!' 
langcode: en

config/schema/my_module.schema.yml
my_module.message.*:
 type: config_object 
 mapping:
  type:
   type: string
   label: 'Message type'
  message:
   type: my_module_message.[%parent.type]
 langcode:	
   type: string
   label: 'Language code'

my_module_message.warning:
 type: string

my_module_message.multiple:
 type: sequence
 label: 'Messages'
 sequence:
  type: string
  label: 'Message'
```

首先看my_module.schema.yml这个文件，'my_module.message.*'用了通配符定义结构体，所以它适用于一组配置名称。所以我们看到分别有my_module.message.single.yml跟my_module.message.multiple.yml
这两个配置文件。接着往下看my_module.schema.yml，'type: my_module_message.[%parent.type]' message的类型用了动态元素[%parent.type]来定义，所以在下面我们看到了my_module_message.warning跟
my_module_message.multiple两个类型定义，无论是my_module_message.warning还是my_module_message.multiple的前缀都是跟my_module_message.[%parent.type]的前缀一致的，另外需要注意的是内部节点的
名称要避免与根节点相同以免造成冲突。

4、关于动态类型 [type]
如果想要在已有的数据下面定义你自己不同的数据类型 那么[type]将会变的非常有用.
```php
config/install/my_module.message.single.yml
 message: 
  type: warning
  value: 'Hello!'
 langcode: en
 
config/install/my_module.message.multiple.yml
 message:
  type: multiple 
  value:
   -'Hello!'
   -'Hi!'
 langcode: en

config/schema/my_module.schema.yml
my_module.message.*:
 type: config_object
 mapping:
  message:
   type: my_module_message.[type]
  […]
  
my_module_message.warning:
 type: mapping
 […]
 
my_module_message.multiple:
 type: mapping
 […]
```

首先看my_module.schema.yml文件用[type]来动态定义message的类型，所以my_module.schema.yml这个文件中我们又看到了my_module_message.warning跟my_module_message.multiple这两个定义类型，
需要说明的是无论是my_module_message.warning还是my_module_message.multiple它们的类型都是mapping类型，那么它们对应的值也必须mapping类型，我们再来看my_module.message.single.yml跟
my_module.message.multiple.yml两个文件，message的值也是mapping类型。
你同样可以定义一个包含相同的key，例如’type’的 基础类型,并且继承这个基础的类型你可以扩展任何自定义类型的key。

5、关于动态类型[%key]

```php
config/install/my_module.messages.yml
messages: 
 'single:1': 'Hello!'
 'single:2': 'Hi!'
 'multiple:1':
  -'Good morning!'
  -'Good night!'
langcode: en
```

这是一个任意消息元素的列表.
```php
config/schema/my_module.schema.yml
 my_module.messages:
  type: config_object
  mapping:		
   messages:
    type: sequence
    label:'Messages'				
    sequence:				
     type: my_module_message.[%key] 
   langcode:
    type: string
    label: 'Language code'
 
 my_module_message.single:*:
  type: string
  label: 'Message'

 my_module_message.multiple:*:
  type: sequence
  label: 'Messages'
  sequence:
   type: string
   label: 'Message'	
```

首先看config/schema/my_module.schema.yml，messages的类型是sequence，而sequence的类型则是以my_module_message为前缀的动态类型，再往下看定义了my_module_message.single:*:跟 
my_module_message.multiple:*:两个结构类型，这两个类型对应的messages值分别是以'single:*'跟'multiple:*'为前缀的，所以我们在config/install/my_module.messages.yml这个文件中
看到了'single:*'跟'multiple:*'为前缀的的messages的默认值