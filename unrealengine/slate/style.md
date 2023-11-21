# Style

Style的主要作用就是服务于UI的样式。编辑器当中主流的Style就是FCoreStyle和FAppStyle。

组成这个系统的类有许多。其中管理所有的Style元素的类是FSlateStyleSet所有的样式相关的单例类都继承于这个类。这个类实现了接口。ISlateStyle中主要定义的接口是Get派的，Get各种各样的Style资源。这个接口还实现了一个函数

```cpp
static FName Join( FName A, const ANSICHAR* B )
	{
		if( B == nullptr )
		{
			return A;
		}
		else
		{
			return FName( *( A.ToString() + B ) );
		}
	}
```







FSlateSet有很多的Map如下所示里面保存了该样式单例类内含有的所有的样式。

```cpp
TMap< FName, TSharedRef< struct FSlateWidgetStyle > > WidgetStyleValues;

	/** float property storage. */
	TMap< FName, float > FloatValues;

	/** FVector2D property storage. */
	TMap< FName, FVector2f > Vector2DValues;

	/** Color property storage. */
	TMap< FName, FLinearColor > ColorValues;

	/** FSlateColor property storage. */
	TMap< FName, FSlateColor > SlateColorValues;

	/** FMargin property storage. */
	TMap< FName, FMargin > MarginValues;

	/* FSlateBrush property storage */
	FSlateBrush* DefaultBrush;
	TMap< FName, FSlateBrush* > BrushResources;

	/** SlateSound property storage */
	TMap< FName, FSlateSound > Sounds;

	/** FSlateFontInfo property storage. */
	TMap< FName, FSlateFontInfo > FontInfoResources;

	/** A list of dynamic brushes */
	mutable TMap< FName, TWeakPtr< FSlateDynamicImageBrush > > DynamicBrushes;

	/** A set of resources that were requested, but not found. */
	mutable TSet< FName > MissingResources;
```

然后我们每一个单例类还得注册到一个类中去，到现在我还不明白这么做的原因是什么？

这个类是FSlateStyleRegistry，里面的所有内容全部注册在：\`

```cpp
/** Repository is just a collection of shared style pointers. */
static TMap<FName, const ISlateStyle*> SlateStyleRepository;
```

> 关于StyleSet如何设置一个路径，然后自动获取该路径下的各种样式。

:question:在FSlateStyleSet这个类当中ContentRootDir 和 CoreContentDir在类中的实际作用是什么?

答案是这些类未在FSlateStyleSet中发挥真正的作用，其真正发挥作用的地方在其子类FSlateGameResources中。







