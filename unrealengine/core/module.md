# Module

关于虚幻引擎模块目前找到的最好的视频：

{% embed url="https://www.youtube.com/watch?t=470s%E2%80%8Bwww.youtube.com/watch?v=DqqQ_wiWYOw&t=470s&v=DqqQ_wiWYOw" %}

### 为什么要使用模块，使用模块有什么优点。

* 方便对代码进行重复使用。
* Better Code practices / encapsulation.
* 更快速的编译和链接时间
* 对于已加载内容的更好的控制。

### Use

* Code from your module is not exposed to other modules by default, you need to mark each function or class explicitly for export.
* 任何你不想让其他模组用的头文件可以放到 private 文件夹中。
* 如果你不想让其他的类依靠也就是使用你的primary game moudle。这个primary game模组里面就不需要public 和 private 文件夹。如果你不计化将这个模组中的内容暴露出去。你就不需要使用public 和 private 文件夹，直接将文件放在 source 文件夹里面。

只是导出到引擎/编辑器 C++ classes in other moules can't cast to it or call its functions.导出类型信息到其他模组，那么其他的模组可以： UCLASS(Blueprintable,MinimalAPI)

* Cast to the class
* Extend the class
* Use inline functions

将函数导出到其他模组 \[YourMoudleName]\_API为了能够将韩函数导出到其他的模组我们需要添加一个导出修饰符在函数的前面。 虚幻通过\`\[YourMoudleName]\_API创建了这个修饰符(specifier)为每个模组。我们可以直接在我们的类前面指定 类名\_API 将暴露类里面所有的内容。这允许其他的模组使用任何公共的属性或者函数。为了能够使用其他模组的类你需要：

* 在你的cpp或者头文件里面包含文件 整个路径从classes 或者 public开始 ，classes 现在已经过期了。

我们可以给模组添加公共依赖 public dependency 和私有依赖 private dependency公共依赖和私有依赖之间的差别 the difference between private and public dependency添加一个子模组作为公共依赖或者私有依赖：他将添加include 路径

<figure><img src="https://picx.zhimg.com/80/v2-d3ec5744afdf2f7d6ae0ac9b21a7766a_720w.png?source=d16d100b" alt=""><figcaption></figcaption></figure>

添加图片注释，不超过 140 字（可选）

<figure><img src="https://picx.zhimg.com/80/v2-f2117bdc73ef02eb68e3da313832bae4_720w.png?source=d16d100b" alt=""><figcaption></figcaption></figure>

添加图片注释，不超过 140 字（可选）

如果只有你的模组的cpp文件或者私有的头文件使用到了依赖的头文件。那么我们就应该使用私有依赖。私有依赖是首选的因为他们能够减少编译时间。当可以的时候你可以使用 向前声明。Missing moudle dependencies will produce compiler or linking errors.demystifying 阐明启发

### Implement

调用 IMPLEMENT\_MOUDLE 在任何cpp文件的声明之后。按理说可以在cpp文件的任何地方。The MoudleManager header is in Core moudle， 解释了为什么最小的模组也要依赖于Core moudleMain class is any class that extends IModuleInterface.a moudle main class is basically a class that shares the lifetime of the moudle itsself\


### Load

模组描述模组需要在.uproject 或者 .uplugin 文件里面描述。定义模组在什么时期加载并且加载的目标和平台是什么？

```
"Moudules":[   {       "Name": "FooBar",       "Type": "Runtime",       "LoadingPhase":"Default"   }]
```

<figure><img src="https://pic1.zhimg.com/80/v2-aafff0febc4e06e1fd8afd9b4f283017_720w.png?source=d16d100b" alt=""><figcaption></figcaption></figure>

添加图片注释，不超过 140 字（可选）

### Depend

只有在依赖链上的模组才会被编译。你可以将你的模组添加到chain上：

* 如果有其他的模组依赖于它。在他们的.build.cs 文件中：
* \[Private/Public]DependencyMoudleNames arrays.
* 更倾向于在没有内容依赖于你的模组的时候，这个时候模组是不编译的。
* 如果没有其他的模组依赖于它 在你的.Target,cs文件夹。
* ExtraMoudleNames 数组
* 当这个模组应该被编译但是没有任何模组依赖于它的时候
* 通常比较多的是 primary game moudle 和 costom editor moudles。

为了更好的掌握模组，我们还需要一些更加重要的内容：

* Precompiled Headers 预编译的头文件
* Include What You Use
* DefaultBuildSettings
* Moudule Logging
* Plugins

#### PCH

正常的头文件不会自己编译自己的。他们被include 在cpp文件内，编译也是在cpp文件中被编译。

* 它们被包含并且被编译在每一个cpp文件里面。
* Lots of duplicate compliing.
* 如果你总是包含相同的 x 头文件，为什么不只编译一次呢。
* 这个时候就用到了 PCHs.
* 定义一个头文件这个头文件里面包含了所有你最常用的头文件。
* 在其他文件编译之前被编译。
* 不会被再次编译除非 它包含的头文件被改变了。
* 然后所有模组里面的其他cpp文件都会被编译。
* 最适合于引擎头文件或者那些很少被改变的头文件。

有很多不同类型的 precompiled HeadersPrivate PCH

* A custom PCH 你为你的模组创建的。
* 在你的.Build.cs 文件中定义它。
* PrivatePCHHeaderFile = "FooBarPrivatePCH.h"
* 你自己不要去包含这个文件在你的头文件或者cpp文件里面
* UBT 将会自动的注入这个东西在你的模组中的所有已编译的头文件中。
* PCHs 应该被认为是一个优化层。
* 不要将其作为一个简单的 include all，只包含你用到的内容就可以。
* 尽管PCHs关闭了你的文件也会被编译。

Shared PreCompiled headers相对于定义你自己的PCH你可以使用一个共享的PCH。A shared PCH 是一个模组定义的一个PCH供其他的依赖模组使用的。存在于一些基础的很常用的虚幻模组当中。例如 UnrealEd，Engine，Slate，CoreUObject Core只有引擎的模组可以创建 shared PCHs.一个shared PCH 将只能够被编译一次。即使有很多的模组在使用它。Include what you use 可以简写为 IWYU\


#### Module Logging

能够为你的模组创建一个log的分类是一个很好的习惯。声明一个日志分类的类型：

* DECLARE\_LOG\_CATEGORY\_EXTERN(CAtegoryName,DefaultVerbosity,CompileTimeVerbosity);
* 声明一个类继承于 FLogCategory
* 最实用的作坊就是将其放在自己的头文件中。

Initialize the log category with your moudle 在你的模组里面初始化日志类型

* DEFINE\_LOG\_CATEGORY(CategoryName);
* 实例化一个日志分类类的实例类，在这个log suppression system的构造函数里面注册这个类。
* 将这个东西放在和 IMPLEMENT\_Moudle 的相同的地方。

Use the log category 使用这个`log`分类\




