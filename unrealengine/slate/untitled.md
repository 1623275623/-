# Untitled

现在随着学习的不断的深入自己的认识也逐渐变得更加的深刻起来。

在编辑器开发的过程当中，主要出现了三个主要的角色。

* 按钮
* 命令
* 样式

在一个插件当中，上面三块工作又三个单例类来进行专门的负责。我从中发现了虚幻引擎这是使用了设计模式当中的命令模式，《游戏设计模式》一书当中的第一章有对命令模式进行详细的介绍。实现了三者之间的相互解耦。

下面就来简单阐述一下命令和按钮部分。当我们开发简单的小工具的时候，UI按钮可能会直接绑定对应的命令操作。这样就不整齐。我们的命令执行函数为了方便就写在了UI类里。这样UI类就显得十分的臃肿。所以对其进行解耦。

首先介绍其中一个最重要的类：FUICommandInfo。它的功能就是描述一个命令。主要是从UI层面对一个命令进行描述，当中并不含有该命令具体执行的内容，包括命令具体执行的函数或者Delegate。



到此，可以说完全看懂了一个插件的代码的各个部分。这个插件有很多主要的大主管。首先是样式大主管，这些代码有很强的参考和借鉴的意义。这些主管都是单例类。

## 样式

```cpp
class FTestContentStyle
{
public:

	static void Initialize();

	static void Shutdown();

	/** reloads textures used by slate renderer */
	static void ReloadTextures();

	/** @return The Slate style set for the Shooter game */
	static const ISlateStyle& Get();

	static FName GetStyleSetName();

private:

	static TSharedRef< class FSlateStyleSet > Create();

private:

	static TSharedPtr< class FSlateStyleSet > StyleInstance;
};
```

基本上插件当中用到的样式都来自于这里。下面是相关的Cpp文件。

<pre class="language-cpp" data-overflow="wrap" data-line-numbers data-full-width="true"><code class="lang-cpp"><strong>#define RootToContentDir Style->RootToContentDir
</strong>
TSharedPtr&#x3C;FSlateStyleSet> FTestContentStyle::StyleInstance = nullptr;

void FTestContentStyle::Initialize()
{
	if (!StyleInstance.IsValid())
	{
		StyleInstance = Create();
		FSlateStyleRegistry::RegisterSlateStyle(*StyleInstance);
	}
}

void FTestContentStyle::Shutdown()
{
	FSlateStyleRegistry::UnRegisterSlateStyle(*StyleInstance);
	ensure(StyleInstance.IsUnique());
	StyleInstance.Reset();
}

FName FTestContentStyle::GetStyleSetName()
{
	static FName StyleSetName(TEXT("TestContentStyle"));
	return StyleSetName;
}


const FVector2D Icon16x16(16.0f, 16.0f);
const FVector2D Icon20x20(20.0f, 20.0f);

TSharedRef&#x3C; FSlateStyleSet > FTestContentStyle::Create()
{
	TSharedRef&#x3C; FSlateStyleSet > Style = MakeShareable(new FSlateStyleSet("TestContentStyle"));
	Style->SetContentRoot(IPluginManager::Get().FindPlugin("TestContent")->GetBaseDir() / TEXT("Resources"));

	Style->Set("TestContent.PluginAction", new IMAGE_BRUSH_SVG(TEXT("PlaceholderButtonIcon"), Icon20x20));
	return Style;
}

void FTestContentStyle::ReloadTextures()
{
	if (FSlateApplication::IsInitialized())
	{
		FSlateApplication::Get().GetRenderer()->ReloadTextureResources();
	}
}

const ISlateStyle&#x26; FTestContentStyle::Get()
{
	return *StyleInstance;
}

</code></pre>

和样式相关的有很多类。上面是一个很好的参考例子，以后自己想要实现相似的功能，代码完全可以按照上面的来实现。

:question: 现在自己还不知道为什么样式要进行注册。自定义的单例样式类都会注册到一个全局的单例类里面去。然后在`shutdown`的时候都会结束注册。

上面的`FSlateStyleSet` 可以说是样式的数据库，里面有各种样式的Map表。建立了各种样式的FName 和 样式之间的联系。可以通过Get函数获取。下面代码的第12行就是获取相应的样式。

<pre class="language-cpp" data-overflow="wrap" data-line-numbers><code class="lang-cpp"><strong>void MakeUICommand_InternalUseOnly( FBindingContext* This, TSharedPtr&#x3C; FUICommandInfo >&#x26; OutCommand, const TCHAR* InSubNamespace, const TCHAR* InCommandName, const TCHAR* InCommandNameUnderscoreTooltip, const ANSICHAR* DotCommandName, const TCHAR* FriendlyName, const TCHAR* InDescription, const EUserInterfaceActionType CommandType, const FInputChord&#x26; InDefaultChord, const FInputChord&#x26; InAlternateDefaultChord)
</strong>{
	static const FString UICommandsStr(TEXT("UICommands"));
	const FString Namespace = InSubNamespace &#x26;&#x26; FCString::Strlen(InSubNamespace) > 0 ? UICommandsStr + TEXT(".") + InSubNamespace : UICommandsStr;

	FUICommandInfo::MakeCommandInfo(
		This->AsShared(),
		OutCommand,
		InCommandName,
		FInternationalization::ForUseOnlyByLocMacroAndGraphNodeTextLiterals_CreateText( FriendlyName, *Namespace, InCommandName ),
		FInternationalization::ForUseOnlyByLocMacroAndGraphNodeTextLiterals_CreateText( InDescription, *Namespace, InCommandNameUnderscoreTooltip ),
		FSlateIcon( This->GetStyleSetName(), ISlateStyle::Join( This->GetContextName(), DotCommandName ) ),
		CommandType,
		InDefaultChord,
		InAlternateDefaultChord
	);
}
</code></pre>

下面来分析一下UI\_Command的参数；

\`\`

{% code title="" overflow="wrap" %}
```cpp
void MakeUICommand_InternalUseOnly( 
FBindingContext* This,
TSharedPtr< FUICommandInfo >& OutCommand,
const TCHAR* InSubNamespace,
const TCHAR* InCommandName,
const TCHAR* InCommandNameUnderscoreTooltip,
const ANSICHAR* DotCommandName,
const TCHAR* FriendlyName,
const TCHAR* InDescription,
const EUserInterfaceActionType CommandType,
const FInputChord& InDefaultChord,
const FInputChord& InAlternateDefaultChord)
{
```
{% endcode %}

<table><thead><tr><th width="303"></th><th></th></tr></thead><tbody><tr><td>UI_COMMAND</td><td><p></p><pre><code>void MakeUICommand_InternalUseOnly
</code></pre></td></tr><tr><td>PluginAction</td><td><p></p><pre><code>FBindingContext* This
</code></pre></td></tr><tr><td>"TestContent"</td><td></td></tr><tr><td>"Execute TestContent action"</td><td></td></tr><tr><td>EUserInterfaceActionType::Button</td><td></td></tr><tr><td></td><td></td></tr><tr><td></td><td></td></tr></tbody></table>

```
UI_COMMAND(, , , , FInputChord());
```

<pre><code><strong>UI_COMMAND( 
</strong>CommandId,                                           PluginAction                                   
FriendlyName,                                        "TestContent"
InDescription,                                       "Execute TestContent action"
CommandType,                                         EUserInterfaceActionType::Button
InDefaultChord,                                      FInputChord()                           
... ) \
MakeUICommand_InternalUseOnly( this,
CommandId,
<strong>TEXT(LOCTEXT_NAMESPACE),
</strong>TEXT(#CommandId),
TEXT(#CommandId)，
TEXT("_ToolTip"),
"." #CommandId,
TEXT(FriendlyName),
TEXT(InDescription),
CommandType,
InDefaultChord,
## __VA_ARGS__ );
</code></pre>

下面是命令主管。

```svg
FTestContentCommands()
	: TCommands<FTestContentCommands>(
	TEXT("TestContent"),
	NSLOCTEXT("Contexts", "TestContent", "TestContent Plugin"),
	NAME_None,
	FTestContentStyle::GetStyleSetName()
	)
{
}
```

```cpp
/** Construct a set of commands; call this from your custom commands class. */
TCommands( 
const FName InContextName,
const FText& InContextDesc,
const FName InContextParent,
const FName InStyleSetName)
   : FBindingContext( InContextName, InContextDesc, InContextParent, InStyleSetName )
{
}
```

上面两个代码片段阐述了上面代码的参数的对应关系。

这个时候Commands类的Context类的名字和描述包括SetName都已经被设置了。

如果一个插件的名字是“TestContent”，那么Commands Context的名字就是 TestContent。Style的名字是TestContentStyle 也就是PluginName + Style。

```cpp
Style->Set("TestContent.PluginAction", new IMAGE_BRUSH_SVG(TEXT("PlaceholderButtonIcon"), Icon20x20));
```

上面代码的名字部分 是 “TestContent.PluginAction”.

所以必须记住PluginName + CommandInfo name； 当我们想要为命令设置 Style的时候。当我们自己写代码的时候，只要顺着这个模子写就不会出什么大问题。



那么这三个类是如何被组织起来一起工作的呢？

当模块加载的时候，第一个首先被调用的肯定就是模块的`StartupModule` 函数，我们从最基本的插件当中了解到的是首先进行样式类和命令类的创建和初始化工作。

```cpp

FTestContentStyle::Initialize();
FTestContentStyle::ReloadTextures();

FTestContentCommands::Register();

PluginCommands = MakeShareable(new FUICommandList);

```

那么在模组类中的配置就是一个成员变量和一个函数。

```cpp
 TSharedPtr<class FUICommandList> PluginCommands;
```

```cpp
void RegisterMenus();
```







