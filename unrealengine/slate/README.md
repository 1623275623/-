# Slate

很长时间我可能都没有搞清楚到底如何进行Slate开发，整个虚幻引擎的UI框架的逻辑到底是什么？UI是如何被渲染的？但是当我对Slate和PrimitiveComponent这两个部分有一定的了解之后，我发现了一些很相似的地方。

比如说我们都是需要继承父类的某个函数去完成某项工作，比如说Slate的每个控件都会继承Onpaint函数，在这个函数内我们可以做很多绘制，引擎也提供给我们很多绘制的函数。下面是Onpaint函数

```cpp
int32 SButton::OnPaint(const FPaintArgs& Args, const FGeometry& AllottedGeometry, const FSlateRect& MyCullingRect, FSlateWindowElementList& OutDrawElements, int32 LayerId, const FWidgetStyle& InWidgetStyle, bool bParentEnabled) const
```

这个函数有很多参数。但是简单了解之后发现绘制的元素其实都在`FSlateWindowElementList` 这个里面。

而PrimitiveComponent也是有一个负责专门绘制的函数。

```cpp
virtual void GetDynamicMeshElements(const TArray<const FSceneView*>& Views, const FSceneViewFamily& ViewFamily, uint32 VisibilityMap, class FMeshElementCollector& Collector) const;
```

在这个函数当中`FMeshElementCollector` 这个函数负责收集需要渲染的信息。最近遇到了一个bug就是使用`Collector->GetDPI(ViewIndex);` 的时候DPI中的View是空的是`NULL` 自己目前也不知道具体的原因是什么？



其中在一个博客上看到说Slate有三个十分重要的类：

* FSlateApplication 全局单例所有的UI调度中心
* SWindow 顶层窗口，持有跨平台窗口实例 FGenericWindow，提供窗口相关属性。
* &#x20;Swidget ：小部件，划分窗口区域，处理自身区域内的交互性事件。

## 基本使用

创建一个简单的SWindow窗口的示例代码如下：

```
auto Window = SNew(SWindow)                         //创建窗口
    .ClientSize(FVector2D(600, 600))                //设置窗口大小
    [                                               //填充窗口内容
        SNew(SHorizontalBox)                        //创建水平盒子
        + SHorizontalBox::Slot()                    //添加子控件插槽
        [                                           
            SNew(STextBlock)                        //创建文本框
            .Text(FText::FromString("Hello"))       //设置文本框内容
        ]
        + SHorizontalBox::Slot()                    //添加子控件插槽
        [
            SNew(STextBlock)                        //创建文本框
            .Text_Lambda([](){                      //设置文本框内容   
                return FText::FromString("Slate");
            })      
        ]
    ];
FSlateApplication::Get().AddWindow(Window, true);   //注册该窗口，并立即显示
```



## Tool Menus 用于对新菜单和按钮的注册

我现在对于这个东西还是一脸懵的状态。







#### Notification 的基本使用

```cpp
H:\UE\UE_5.2\Engine\Source\Runtime\Engine\Private\UnrealClient.cpp
auto Message = NSLOCTEXT("UnrealClient", "HighResScreenshotSavedAs", "High resolution screenshot saved as");
FNotificationInfo Info(Message);  //创建通知对象
Info.bFireAndForget = true;
Info.ExpireDuration = 5.0f;
Info.bUseSuccessFailIcons = false;  
Info.bUseLargeFont = false; 
		
const FString HyperLinkText = FPaths::ConvertRelativePathToFull(CachedScreenshotName);
Info.Hyperlink = FSimpleDelegate::CreateStatic([](FString SourceFilePath) 
{
		FPlatformProcess::ExploreFolder(*(FPaths::GetPath(SourceFilePath)));
}, HyperLinkText);
Info.HyperlinkText = FText::FromString(HyperLinkText);
		
FSlateNotificationManager::Get().AddNotification(Info);
UE_LOG(LogClient, Log, TEXT("%s %s"), *Message.ToString(), *HyperLinkText);
```





