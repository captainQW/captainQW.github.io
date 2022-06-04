title: sublime text创建快捷注释信息
date: 2015-07-14 10:18:10
categories: linux
tags: [sublime text]
---

* 该方式适用于sublime text2和3。

## 创建代码片段
点击菜单栏里的 Tools 菜单，然后点击 New Snippet，之后会在新的 tab 页创建一个代码片段模板，编辑代码如下
```bash
<snippet>
	<content><![CDATA[
/**
 * 定义函数
 *
 * 函数功能描述
 * 
 * @author panjun.liu <lpj163@gmail.com>
 *
 * @param int $param1      参数释义
 * @param string $param2   参数释义
 *
 * @return integer|string|array
*/
]]></content>
	<!-- Optional: Set a tabTrigger to define how to trigger the snippet -->
 	<tabTrigger>phpdoc</tabTrigger>
	<!-- Optional: Set a scope to limit where the snippet will trigger -->
	<!-- <scope>source.python</scope> -->
</snippet>
```
> 注：
> tabTrigger 此节是可选的配置，默认是被注释掉的。默认 hello 的意思是：如果你在某个文档里输入了单词 hello ，然后按下 Tab 键，接着 hello 就会被替换为1中定义的代码片段。再次按下Tab键，接着snippet会被替换为2中定义的代码片段。

保存到当前用户主目录下的\Sublime Text 3\Packages\User目录中，命名为phpdoc.sublime-snippet(注意：后缀必须为.sublime-snippet)。

## 绑定快捷键

可以将上述的操作绑定到一个快捷键，在不键入任何文本的情况下，直接按快捷键插入代码片段。

点击菜单栏的 Preferences 的子菜单 Key Binding – User，在打开的文件的方括号内部粘贴如下配置：

```bash
[
	{ "keys": ["phpdoc"], "command": "insert_snippet", "args": {"name": "Packages/User/phpdoc.sublime-snippet"} }
]
```

现在简单介绍一下这段配置：

1. "keys": ["phpdoc"] 这个定义了触发此命令的快捷键。

2. "command": "insert_snippet" 这个是需要触发的命令的名字。

3. "args": {"name": "Packages/User/phpdoc.sublime-snippet"} 这个是需要传入到上述命令的参数。这里把代码片段文件的相对路径传递过去。

保存配置文件，现在就可以用快捷键插入代码片段了。

至此，输入phpdoc，然后按tab键就会输出你定义这段注释。