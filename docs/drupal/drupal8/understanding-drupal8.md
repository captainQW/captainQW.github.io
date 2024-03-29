title: understanding Drupal8-Drupal8初窥
date: 2015-11-02 15:42:26
categories: Drupal8
tags: [drupal8]
---

如何学习drupal8？国内文档寥寥可数，后发现老外的一篇文章，可做入门，遂翻译如下。原创内容，转载请注明出处！

##一、结构
Drupal 8来了。这是一个革命！和Drupal6到Drupal7的演化不同，这是一个完全不同的架构和编码方式。我确信这些变化对专业化和现代化CMS是必不可少的。我相信转向面向对象将会吸引开发者，并将提升软件的整体质量。我感谢Drupal内核开发者的勇气以及为此所做的工作。
然而，对于现有（包括新的）Web开发者，学习drupal将是一个巨大的挑战。通过阅读这篇文章，您将更具优势。在你深入Drupal 8模块开发前，这篇文章会给你带去些许启示。在我写下这篇文章之前，我花了数天，通过一步步的调试、阅读代码和在线文章，深入地学习了symfony2 和 Drupal8。我是一个经验丰富的Drupal7开发者，但是对symfony一无所知，并且至今没有对Druapl8内核有所贡献。我相信，通过分享我所学到的，也能助你提升对它的理解。
在接下来的四周里，这篇文章将发布成四部分。在这第一部分，我们将一瞥Drupal 8 框架的总体架构，特别是与symfony2 组件的联系。我们将看到哪些symfony2 组件在Drupal中用到了以及如何使用。在下一个部分，我们将详细学习非常重要的类容器。在第三部分，我们将详细学习Drupal 8 的引导和路由。你将学到如何处理一个请求。在第四也是最后一部分，将会介绍drupal 8的其它一些令人兴奋的、全新的特性。
###1、Symfony2
Symfony框架被设计用来使开发者能够构建自定义的Web应用。Symfony不是一个CMS，因为它不能用来管理一个站点。相反，要构建应用，必须要写代码。为此，symfony提供了一些方法使开发更具效率。理论上，CMS可以基于Symfony开发，但事实上，Symfony的默认方式对Drupal来说并不灵活。

因此，Drupal 8只采用了Symfony的非常优秀的内核层，并且扩展它以支持Drupal模块。这促生了一个非常平衡的系统，它采用Symfony部分优秀的内核（而且未经修改），但是可以基于一个非常灵活的CMS层扩展它。
###2、组件
Symfony框架由数个组件构成。有些对系统是至关重要的，例如能理解HTTP协议并且为其他组件提供良好request和response对象的Http Foundation组件。其他的仅是辅助组件，例如校验数据的Validator组件（检测是否是合法的电子邮件地址/URL/数字/等等？）。系统的核心是Kernel组件。这个内核总的来说就是一个主类，它管理环境（类和程序集）并负责处理http请求。开发者可以通过扩展内核来扩展系统，例如AppKernal就是扩展了Kernal，并且在里面添加了自己的程序集。这些程序集可以构建一系列连贯的功能集合，非常类似于Drupal的模块。
Drupal只使用了Symfony的一部分组件，如下图所示：
![组件](https://static.verycloud.cn/sites/default/files/pic/image/20151104/2015110494853_67351.png)

###3、Drupal 8如何扩展Symfony
Drupal不扩展内核，而Symfony应用会。通过实现内核接口，Drupal提供了相似的功能类型。这是因为Drupal不共享Symfony的程序集方法。程序集提供了极好的方式来构建自定义的web应用，但是不适合构建需要灵活性和扩展性的CMS。DrupalKernal加载环境（可用的类和模块）的方式与Kernal稍微不同，但是和Symfony Kernal一样，都是把将http请求委托给HttpKernel。
与此同时，Drupal 8引入了自己的（第三方）组件和核心代码，示意概览如下图所示：
![Drupal 8如何扩展Symfony](https://static.verycloud.cn/sites/default/files/pic/image/20151104/2015110494817_42194.png)

###4、结论
这里总结了“开始学习Drupal 8”这篇文章。下周我们将开始学习类容器，可以理解成Drupal系统的骨架。在理解任何其他组件前你必须先了解它。


##二、服务容器
这是“理解Drupal 8”的第二部分。在第一部分，我们已经学习了Drupal 8的总体架构以及它与Symfony的关联。Symfony（和Drupal 8）包含着许多组件。在这篇文章中，你将会学习到什么是服务容器，以及在Drupal 8中怎么使用它。很重要的一点就是，在学习路由之前我们得先理解服务容器。

Symfony采用了服务器容器，它可以高效地管理应用中的services。这个概念也就是所谓的Dependency Injection。

服务容器是一个在处理请求之前由Kernal创建并包含的全局对象。它能被用来获取服务，动态加载服务。服务是用于完成特定任务的全局对象，例如邮件服务或数据库连接。一个服务对应着一个确切的类。服务容器非常重要，因为它包含可用服务，知道它们的联系和配置信息，甚至构造它们。

###1、依赖和参数
一个服务可能依赖于其他服务。Symfony文档举了个通讯管理服务依赖邮件服务来发送邮件的例子。服务容也管理着这些依赖。当创建一个服务时，它的依赖会通过类构造器中的参数进行传递。接口用来定义依赖服务要提供哪些方法，这样实现的这个服务才能在必要时与另一个服务进行交换。

###2、配置
服务容器中的服务可以通过多种方式进行配置：代码（所谓的扩展），XML，YAML，等等。Symfony混合着使用这些方式，但是把大多数核心服务配置留给了程序集代码。Drupal的服务配置则不同，大部分基于yaml文件。Drupal使用services.core.yml来关联所有与内核相关的配置。这个配置可以被模块和ServiceProvider类扩展。

YAML提供了可读的和灵活的语法。Drupal 8中的一个例子（摘自services.core.yml）：

```php
services:
  ...
  router_listener:
  class: Symfony\Component\HttpKernel\EventListener\RouterListener
  tags:
    - { name: event_subscriber }
  arguments: ['@router']
  ...
  router:
    class: Symfony\Cmf\Component\Routing\ChainRouter
    calls:
      - [setContext, ['@router.request_context']]
      - [add, ['@router.dynamic']]
  ...
```

这里定义了router_listener服务。对应的类要求能够加载这个监听器。arguments属性定义了RouterListener 构造器的第一个参数（URL或请求匹配器）应该是service id是router的服务：‘@router’。这个路由也被定义在配置文件中，而且是一个不同于Symfony中使用的‘类’。

###3、标签化的服务
标签被用来加载特定的标签化的服务。在实践中，drupal有时使用钩子方式。在以下例子中（node.services.yml），node模块为'add node'页面增加了一个访问检查，基于access_check服务模式，当访问时，将会检查访问是否被允许。

```php
services:
  ...
  access_check.node.add:
    class: Drupal\node\Access\NodeAddAccessCheck
    arguments: ['@plugin.manager.entity']
    tags:
      - { name: access_check }
        ...
```

###4、Compiler passes
Drupal是如何理解这些标签以及如何处理它们的呢？好，Symfony已经有另外一种动态配置服务容器的方法：Compiler pasess。服务容器实际上在用静态配置构建之后就被编译了。在这个期间，它允许对象实现CompilerPassInterface 类来更改配置。在Drupal中，CoreServiceProvider 注册了一些非常重要的Compiler pasess，例如RegisterAccessChecksPass，它会试图找到所有加了access_check标签的服务（见上例）然后把它添加到AccessManager（通过使用addMethodCall容器指令）中。在路由阶段，AccessManager 会检查访问权限，它将在其他类之间检查NodeAddAccessCheck。在drupal内核中还有几个其他的Compiler pasess，例如用于将路由参数转换成对象和转义字符串的Compiler passes。在实践中，你有时需要在模块中使用这些标签化的服务来增加自定义访问校验，转换路径参数。

###5、Swapping services
服务容器使灵活的服务配置成为可能。Drupal 8使用服务参数改变了Symfony使用这些参数实现自己功能的方式。例如，drupal需要一个不同于Symfony的url路由机制。与Symfony的(Symfony\Bundle\FrameworkBundle\Routing\Router)相比，它通过提供不同的路由服务(Symfony\Cmf\Component\Routing\ChainRouter)来实现。而这个
不需要改变router_listerner (Symfony\Component\HttpKernel\EventListener\RouterListener)类本身的任何一行代码。由于两个路由都实现了RequestMatcherInterface接口，而这个接口是RouterListener的第一个必要参数，所以这个是可以实现的。
 
###6、结论
在这一部分，我们学习了Drupal 8 的服务容器组件。现在，你对Drupal8不修改一行代码实现自己的服务替换Symfony2的服务应该有了更好的认识。这允许drupal内核维护者轻松地升级Symfony2。当学习Drupal 8时，你会经常翻阅core.services.yml文件查找正在使用的服务。

下周，我们将详细学习Drupal 8的控制流程和路由。


##三、路由

这是'理解drupal8'的第三部分，在前2个部分，我们学习了drupal8和Symfony的结构化差异。这部分我们将学习drupal8如何处理一个请求。首先，我们将看下引导阶段以及控制流程，然后我们将会学习事件订阅，这是个非常重要的概念，需要你在学习请求处理之前就理解它。

###1、流程控制(Flow of control)
接下来，我们将学习发起一个请求之后，drupal8是如何处理的。

####1)、bootstrap配置
①、读取settings.php文件（一般在sites/default/下），动态生成一些其它设置信息，保存到全局变量和Drupal\Component\Utility\Settings对象中。
②、执行类加载器(class loader)，加载类。
③、设置Drupal错误处理程序。
④、检测Drupal是否正确安装，如没有，则跳转到安装界面。
####2)、构建Drupal内核
####3)、初始化service container（从缓存或者重建）
####4)、把container添加到Drupal静态类
####5)、尝试加载缓存的静态页(类似Drupal7)
####6)、加载所有变量(variable_get) ？这个函数已经去掉了
####7)、加载其他必要的文件
####8)、注册stream wrappers(public://, private://, temp://，自定义)
####9)、创建HTTP请求对象（使用Symfony HttpFoundation）
####10)、DrupalKernel处理并返回结果
####11)、发送返回结果给客户端
####12)、中断请求

其中最值得探讨的是在请求处理阶段究竟发生了什么?要了解这点，我们先来学习事件监听。

###2、事件订阅(Event subscribers)
之前，我们探讨了Compiler passes。标签化服务一个最重要的用处就是，event_subscriber-tagged服务。这些服务应该实现了EventSubscriberInterface，并且基本上都是事件监听器。事件会声明一个getSubscribedEvents方法定义哪个事件应该映射到哪个处理方法，并设置一个属性来决定事件动作执行的顺序。在Drupal核心中只有少数事件在调用：
①、kernel.request
请求分发的最初阶段调用
②、kernel.response
请求响应的时候调用
③、routing.route_dynamic
允许模块加载额外的路由
④、routing.route_alter
允许在route collection中改变routes。主要用于核心添加一些校验和参数转换。

每个Drupal开发者都应该了解事件订阅。尤其是kernel.request，因为它替换了hook_init(Drupal7)。另外一个比较重要的事件是routing.route_dynamic事件。因为它可以创建动态路由，而一般路由是在yaml文件中静态配置的。在Drupal8之前，都是在hook_menu中实现，但现在hook_menu仅用于生成菜单项。如block模块在\Drupal\block\Routing\RouteSubscriber用 routing.route_dynamic注册路由用于区块配置页面。

你可能会奇怪事件订阅干嘛不用hook呢？因为这是一种很有效的方式，所有可用的事件监听都已经编译到了服务容器中，而且还被缓存着！同时，它也会让Drupal更加趋于面向对象。

###3、从请求到响应(From request to response)
当一个请求来时，系统开始引导，启动DrupalKernel，调用对应的方法，委派请求给Symfony2的HttpKernel做进一步处理。
HttpKernel分发kernel.request事件，一些订阅监听事件将按照顺序执行：
①、AuhtenticationSubscriber
加载session，设置全局用户
②、LanguageRequestSubscriber
检测当前语言
③、PathSubscriber
将url转换为系统路径(url别名等)
④、LegacyRequestSubscriber
允许设置一个自定义主题并初始化
⑤、MaintenanceModeSubscriber
如果是维护模式，显示维护页面
⑥、RouteListener
获取所有已加载的路由对象
⑦、AccessSubscriber
检查客户端的访问权限

RouterListener在路由工作的时候调用，获取路由服务返回的路由属性。Drupal的路由服务跟Symfony的不同：DynamicRouter在Symfony中仅仅以插件形式提供，见http://cmf.symfony.com/。与Symfony常规路由的主要区别在于，Drupal的DynamicRouter支持增强，我们将在下一节讨论。DynamicRouter类接下来会把查找当前路由的任务（类似于Drupal 7中的current_path）委派给NestedMatcher, 而NestedMatcher又会把这个任务委派给RouteProvider。RouteProvider会在router表中找到匹配该路由的路由，也包括在CMS中的已缓存的路由。这个路由表是由route builder创建，站点我们稍后再做介绍。路由按照路径长度进行匹配：最长的匹配路径被认为是正确的。然后，NestedMatcher要求UrlMatcher在路由集中选定那个特定的活跃路由。通常，能正确的通过http/https通过GET/POST方式访问的路由将被UrlMatcher优先匹配(最长匹配)。当然通常情况下，我们只会定义一个路由，所以UrlMatcher的作用很有限。最终的结果就是匹配到一个可用的活跃路由，如node.add_page。

接下来我们了解一下route filters，可能在实际项目中并不一定用到。通过添加一个打了标签名为route_filter的service，这些route filters将被NestedMatcher调用，在RouteProvider返回正确路由信息之后直接过滤路由集。Drupal核心仅对没有返回MIME类型的路由采用mechanism名做过滤操作，当然，还得显示指定"_format"。

匹配到请求路由后，DynamicRouter继续调用路由的增强属性，包含路由属性，转换路由参数。下一节将讲述这个过程。然后路由属性会返回给RouterListener，并设置在请求对象中。接下来，执行权限检查(AccessSubscriber)，最后，使用正确的参数调用控制器方法，控制器返回结果给客户端。

现在我们知道了请求的处理过程，接下来再来研究一下路由。

###4、路由
在Drupal7中，hook_menu用于注册页面回掉函数(page callbacks)、标题(titles)、参数(arguments)、访问要求(requirements)等，Drupal8中,采用了Symfony的路由，将更加灵活，也更复杂。

Drupal8中将不会再有页面回掉函数，取而代之的是控制器类的方法。这些有效的路由现在会被配置在模块目录下的{module}.routing.yml文件中。举个例子：用户退出登陆的配置：

```php
user.logout:
  path: '/user/logout'
 defaults:
    _controller: '\Drupal\user\Controller\UserController::logout'
  requirements:
    _user_is_logged_in: 'TRUE'

```

注意：任何路由都需要一个ID(user.logout)和路径(/user/logout)。在路径中也可包含参数，后面再来讨论。'default'非常重要，它用于控制请求路径匹配时需要做什么。'requirement'定义了是否应该处理请求，通常包括访问检查，Symfony替代了Drupal7的'access arguments'和'access callback'。

####常用'default'键：
①、_controller
返回一个预期的结果，通过调用特定的方法。
②、_content
如指定，基于请求的MIME类型设置_controller，并用特定的方法(通常是字符串或数组)填充返回值
③、_form
如指定，_controller设置给HtmlFormController::content响应特定的表单。这个form是一个类名，实现了FormInterface，而且通常继承自FormBase。当然表单的构建也是面向对象的。
④、_entity_form
如指定，_controller设置给HtmlEntityFormController::content返回特定的实体表单(如{entity_type}.{add|edit|delete})
⑤、_entity_list
如指定，_controller设置给HtmlFormController::content，_content设置给EntityListController::listing，基于实体类型列表控制器返回一个实体列表，使用期望实体的EntityListController提供一个实体列表。例:_entity_list: view_mode返回一个视图模式的渲染数组。
⑥、_entity_view
如指定，_controller设置HtmlFormController::content，_content设置EntityViewController::view ，在给定的视图模式下，通过路径和渲染模式下查找一个实体。如: _entity_view:node.teaser。 在视图模式下将返回一个{node}的渲染数组。
⑦、_title
页面的标题(字符串)
⑧、_title_callback
页面的标题(方法回调)

####常用'requirement'键：
①、_permission
指定当前用户需要拥有的特定权限
②、_role
指定当前用户需要拥有的特定角色
③、_method
允许的HTTP方法，如GET、POST等
④、_schema
一般为http或https，并且请求的scheme和定义的schema要一致。当使用Drupal::url()输出url时也会带入这个属性。
⑤、_node_add_access
定义某些node type的访问控制
⑥、_entity_access
实例的访问控制
⑦、_format
格式化MIME类型

上面所列的大多数(但不是所有)的requirement'是由access checkers'进行校验，这个我们稍后介绍。实际项目中，你可能需要创建一个自定义的access checkers,因为Drupal 7的'access callback'已不再可用。像node模块中的'_node_add_access',通过定制的权限控制(NodeAddAccessCheck)来处理。

AccessManager执行权限控制,订阅kernel.requet的事件。调用权限检测，必须实现AccessInterface类。在之前，我们已经了解到权限检测会被注册为打了标签access_check的services。权限检测会有一个方法去检测客户端是否有权限访问指定的路由，如果访问被拒绝,会抛出一个“拒绝访问”页面。

####路径参数(path parameters)
在实践中你经常需要参数去控制页面回调。下面的路由定义实例中，路径包含一个名为'node'的参数。每一个路由存储相应的选项。控制器方法接收这些参数作为参数。所以在这种情况下,NodeController方法有一个参数:$node。一个路由可以有多个参数,但是他们的名字应该是唯一的。
```php
node.view:
  path: '/node/{node}'
  defaults:
    _content: '\Drupal\node\Controller\NodeController::page'
    _title_callback: '\Drupal\node\Controller\NodeController::pageTitle'
  requirements:
    _entity_access: 'node.view'
```

参数传递的值默认就是url中的值(一个字符串)，但通常会通过parameter converter进行转换。在上面的示例中,NodeController不会得到node id，而是一个加载的node实体。 参数转换是由订阅着kernel.request事件的ParamConverterManager完成。ParamConverterManager包含可所有注册了的ParamConverterInterface(标签名为paramconverter的服务)的实现。当处理一个请求时,ParamConverterManager会遍历活跃路由的参数并调用参数转换器的转换方法。如果可能的话，一个'string'参数会被替换为一个完整的对象。请注意，如果控制器方法的参数没有明确的类型暗示，将会传递未发生转换的参数。

EntityConverter是Drupal核心中唯一一个提供参数转换的类，但是在Drupal中使用得很广。如果一个参数名匹配到了一个实体类型('node', 'user'等)，它就会被自动转换为实体对象。它的值被认为是实体的id。如果实体不存在或转换失败，则返回404。

你也可以指定一些值作为默认的可选参数，这只适用于参数不被转换的情况。

###4、路由生成(Route building)
路由，显然是在请求中完成的，它使用之前构建的'router'表。RouteBuilder service负责填充路由表(例如,当清理缓存后)。当一个请求来时，这个路由表用于查找对应的路由并完成它。这么做是出于性能的考虑。RouteBuilder通过收集所有的已配置的静态路由(yml-files)并创建一个包含它们的集合。它也调用route.route_dynamics事件，在这个事件中，事件订阅者还可以注册额外的路由(见上面的示例block模块)。然后调用routing.route_alter事件，这个事件可能有多个订阅者。另外一些特别重要的如访问控制和参数转换。

我们刚刚提到的访问检测。当处理一个请求时，访问检测会检测客户端是否有权限访问。当然，并不是所有的路由都需要权限。实际上，在构建路由的时候，就会决定每个路由的访问权限。这项工作由监听router.route_alter事件的AccessManager完成，包含静态和动态的访问控制。StaticAccessCheckInterface有一个方法appliesTo,用于返回一个requirement数组。动态访问控制通过applies方法来测试每个路由。AccessManager在router表中添加了一个选项“_access_checks”，这常用于处理一个请求时做路由的权限控制。注意，每一个路由至少需要一个访问检测，否则永远不会访问路由!

我们也提到参数转换器，也就是一个参数如何被转换器转换。每个参数，可能会有一个适用的转换器(或没有)。就像访问控制，这个适用的转换器会在ParamConverterManager创建路由时被搜索到。这个服务会检查每一个存在的路由参数，以及对应参数转化器的applies方法，并在router表中添加一个选项“converter”。这通常用于处理时，转换参数。

###5、总结(Conclusion)
在本节中,我们已经了解了如何处理请求。同时,你现在应该了解Drupal 8的路由以及它是如何工作的!事实上,你已经知道了从请求到响应这一基本流程。然而，Drupal8中仍然有一些有趣的概念存在，这部分内容将在下个部分也是最后一部分里面进行探讨。好了，我们下周再来讨论。

##四、Drupal8其它概念

在过去的几周我们已经了解了Drupal的结构和如何处理请求。我们还没有了解如何安装配置Drupal8。在你深入具体的核心模块代码之前,你应该了解一些重要的新的或变更的Drupal8概念。我在本文的最后一部分将描述它们。

###1、配置(Configuration)
Drupal8的配置系统做了很多令人兴奋的改进。最值得注意的是,现在配置存储在一个YAML-files中，目录结构一般为'/sites/{name}/files/config_{HASH}/active'。文件名应该准确描述该配置文件，例如,“node.settings.yml”是节点模块的设置，“views.view.content.yml”是一个完整的机器名称为'content'视图定义!

你可能认为配置文件非常不灵活。但在实践中,您可以使用YAML-syntax构建信息的关联数组。此外,Drupal提供一个Config对象可以用来改变配置文件，而不必手动编辑这样一个文件!当您通过Drupal UI创建一个新的字段，节点类型，区块类型或视图时，系统将会创建一个新的文件。内容(nodes)和数据依然存储在数据库中，否则你将无法建立快速查询。

一个模块可以包含conf子目录。当启用了一个模块，目录下的所有配置文件会被复制到网站的配置目录下，当文件添加完毕（而且缓存已经情况），配置会被系统检测到并且自动生效。通过这种方式,一个模块可以提供额外的node types, image styles, views等无需实现Drupal7中的诸如hook_node_info的钩子。

配置文件可以很容易地复制到另一个地方，这有利于在最后阶段更改配置!

配置文件也可能有一个关联的schema，详细描述了一个view/block/image style/的配置应该是什么样的。这样可以更好的验证、翻译配置。

###2、实体(Entities)
Drupal7引入了实体，使用实体作为主要的抽象内容，是Drupal底层架构最大的变化。例如,字段可被连接，可以定义访问方法，可以关联视图等。实体现在变的非常强大。在Drupal8中, 实体已经变的对于CMS来说更加核心。不仅内容数据(节点nodes、分类taxonomy terms、用户users)都是实体,而且non-fieldable比如视图(views)、数据操作(actions)、菜单(menus)、图像(image styles)等都是扩展实体类，也包节点类型和词汇等实体。

使用attributes定义一个实体的属性。例如，这里可以指定一个实体是否fieldable或者它应该作为另一个实体的一个集合。实体依赖于控制器，不一定需要保存在配置文件中。如果控制器继承ConfigStorageController,将被保存为配置文件。虽然配置文件有好处，这意味着您不能使用数据库查询它们。如果你想搜索它们,你要加载所有项目并使用for循环去检索。如果存储在数据库中，例如节点和用户,就可以使用FieldableDatabaseStorageController。节点、用户、分类都是一样。但结果就是，这些内容不能通过配置文件进行传递。对我来说,现在还不清楚是否有可能有一个fieldable配置存储控制器!

如果要加深实体的理解，我建议你看一看ImageStyle类,这是一个纯粹的基于配置的实体，taxonomy term,它是一个字段化基于数据库存储的实体，还有Vocabulary，它是一个实体集。

###3、插件(Plugins)
插件可以看作是钩子的面向对象版本。他们用于模块化包括自定义模块，图像效果等。

学习插件有个很好的示例就是BlockBase抽象类。为了添加一个新的自定义block时，就必须继承这个类，这么做类似于论坛模块的ActiveTopicsBlock类。一些方法不一定必须覆盖其访问检查、视图建立、改变块的形式等。

插件管理器用来定义插件类型，指定命名空间列表以及用于存放插件的子目录。通过这种方式,任何模块可以添加插件。插件通过注释类的方式进行设置。此外，根据不同的插件类型，注释中必须要指定一些配置属性。

###4、类(Drupal class)
Drupal 8核心代码主要是面向对象的。然而，模块中的钩子代码仍然是面向过程的。这里就出现了疑问,因为Symfony是完全面向对象的！在Symfony中，如果您需要依赖于另一个service创建一些功能,您将创建一个新的服务并且在服务容器定义所需的必须依赖项。但如果你想用Drupal的钩子实现一个特定的service,将创建一个静态Drupal类。它可以通过使用Drupal::service(“{service_id}”),或着通过使用特定的像Drupal::request(),Drupal:currentUser(),Drupal:entityManager()的服务访问器类获程序流程中的services。后者的优势是可以为你提供更好的代码补全功能。此外,它提供了一些方便像Drupal::url()的辅助方法，相当于Drupal7的url()方法。

下面是一个采用Drupal class的例子，位于node.module中的node_access方法:

```php
function node_access($op, $node, $account = NULL, $langcode = NULL) {
  $access_controller = \Drupal::entityManager()->getAccessController('node');
  ...
  return $access_controller->access($node, $op, $langcode, $account);
}
```

###5、总结(Conclusion)
在本文中,我们已经了解了Drupal8的结构，以及与Symfony2对比。我们还学习了服务容器管理着所有服务,而服务可以进行配置。之后我们学习了路由和处理请求的过程。最后我们介绍了Drupal8中一些重要的新概念:Drupal类,插件和配置Drupal8。

我希望所有这些会帮助你探索和了解Drupal8。我希望你能够了解Drupal8模块开发，并对这个伟大的项目有所贡献。


原文出处：https://cipix.nl/understanding-drupal-8-part-1-general-structure-framework