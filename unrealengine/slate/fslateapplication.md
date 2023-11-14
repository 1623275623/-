# FSlateApplication

## 研究一下UI的渲染流程

到底我们写的那些组件是如何进到渲染流程的呢，下面我们就来深入的挖一挖。

首先从FSlateApplication开始，如果以后完全懂了Application的Tick是被谁调用的再填坑吧。

```cpp
void FSlateApplication::Tick(ESlateTickType TickType)

TickAndDrawWidgets(DeltaTime);

DrawWindows();

PrivateDrawWindows();

DrawWindowAndChildren( ActiveModalWindow.ToSharedRef(), DrawWindowArgs );

for( TArray< TSharedRef<SWindow> >::TConstIterator CurrentWindowIt( SlateWindows ); CurrentWindowIt; ++CurrentWindowIt )
{
	const TSharedRef<SWindow>& CurrentWindow = *CurrentWindowIt;
	if ( CurrentWindow->GetType() == EWindowType::ToolTip )
	{
		DrawWindowAndChildren(CurrentWindow, DrawWindowArgs);
	}
}


MaxLayerId = WindowToDraw->PaintWindow(
					GetCurrentTime(),
					GetDeltaTime(),
					WindowElementList,
					FWidgetStyle(),
					WindowToDraw->IsEnabled());
					
在PaintWindows 内：
FSlateInvalidationContext Context(OutDrawElements, InWidgetStyle);
	Context.bParentEnabled = bParentEnabled;
	// Fast path at the window level should only be enabled if global invalidation is allowed
	Context.bAllowFastPathUpdate = bAllowFastUpdate && GSlateEnableGlobalInvalidation;
	Context.LayoutScaleMultiplier = FSlateApplicationBase::Get().GetApplicationScale() * GetDPIScaleFactor();
	Context.PaintArgs = &PaintArgs;
	Context.IncomingLayerId = 0;
	Context.CullingRect = GetClippingRectangleInWindow();

	// Always set the window geometry and visibility
	PersistentState.AllottedGeometry = GetWindowGeometryInWindow();
	PersistentState.CullingBounds = GetClippingRectangleInWindow();
	if (!GetVisibilityAttribute().IsBound())
	{
		SetVisibility(GetWindowVisibility());
	}


	FSlateInvalidationResult Result = PaintInvalidationRoot(Context);
		
		CachedMaxLayerId = PaintSlowPath(Context);
```

```cpp
```

```
```



## 研究一下各种事件的调用流程

我上面所说的各种事件，包括且不限于

