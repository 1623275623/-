# FSlateApplication

## 前情提要

```
/** All the top-level windows owned by this application; they are tracked here in a platform-agnostic way. */
TArray< TSharedRef<SWindow> > SlateWindows;

/** All the virtual windows, which can be anywhere - likely inside the virtual world. */
TArray< TSharedRef<SWindow> > SlateVirtualWindows;

/** The currently active slate window that is a top-level window (full fledged window; not a menu or tooltip)*/
TWeakPtr<SWindow> ActiveTopLevelWindow;

/** List of active modal windows.  The last item in the list is the top-most modal window */
TArray< TSharedPtr<SWindow> > ActiveModalWindows;

/** These windows will be destroyed next tick. */
TArray< TSharedRef<SWindow> > WindowDestroyQueue;

/** The stack of menus that are open */
FMenuStack MenuStack;

/** The hit-test radius of the cursor. Default value is 0. */
float CursorRadius;
```

当我第一次看到代码的时候，心里也是充满了很多的问号？

*   什么是ModalWindow？

    GPT给出的解释是那些在最上层的窗口，而且这些窗口不能被忽略而且必须被优先处理。就比如说我们在使用软件的时候，有时候会报错，这些个窗口是不能忽略的，我们必须做出选择才能进行其他的操作。此时在电脑当中我们只能对该窗口进行操作，对其他窗口的操作是无效的。
*   VirtualWindow和普通的SlateWindow到底有什么区别？

    这个忘了，



## 基本使用

```
FSlateApplication::Get()->AddWindow()
```

```cpp
TSharedRef<SWindow> FSlateApplication::AddWindow( TSharedRef<SWindow> InSlateWindow, const bool bShowImmediately )
{
	// Add the Slate window to the Slate application's top-level window array.  Note that neither the Slate window
	// or the native window are ready to be used yet, however we need to make sure they're in the Slate window
	// array so that we can properly respond to OS window messages as soon as they're sent.  For example, a window
	// activation message may be sent by the OS as soon as the window is shown (in the Init function), and if we
	// don't add the Slate window to our window list, we wouldn't be able to route that message to the window.

	FSlateWindowHelper::ArrangeWindowToFront(SlateWindows, InSlateWindow);
	TSharedRef<FGenericWindow> NewWindow = MakeWindow( InSlateWindow, bShowImmediately );

	if( bShowImmediately )
	{
		InSlateWindow->ShowWindow();

		//@todo Slate: Potentially dangerous and annoying if all slate windows that are created steal focus.
		if( InSlateWindow->SupportsKeyboardFocus() && InSlateWindow->IsFocusedInitially() )
		{
			InSlateWindow->GetNativeWindow()->SetWindowFocus();
		}
	}

	return InSlateWindow;
}
```

我们在使用这个函数的时候会调用函数`FSlateWindowHelper::ArrangeWindowToFront(SlateWindows, InSlateWindow);` 我们一般的窗口就是放在SlateWindows这个数组当中，调用这个函数就是为的是将其放入到SlateWindows这个数组当中。

```cpp
void FSlateWindowHelper::ArrangeWindowToFront( TArray< TSharedRef<SWindow> >& Windows, const TSharedRef<SWindow>& WindowToBringToFront )
{
	Windows.Remove(WindowToBringToFront);

	if ((Windows.Num() == 0) || WindowToBringToFront->IsTopmostWindow())
	{
		Windows.Add(WindowToBringToFront);
	}
	else
	{
		bool PerformedInsert = false;

		int32 WindowIndex = Windows.Num() - 1;
		for (; WindowIndex >= 0; --WindowIndex)
		{
			// Topmost windows first, then non-regular windows (popups), then regular windows.
			if (!Windows[WindowIndex]->IsTopmostWindow() && (!WindowToBringToFront->IsRegularWindow() || Windows[WindowIndex]->IsRegularWindow()))
			{
				break;
			}
		}

		Windows.Insert(WindowToBringToFront, WindowIndex + 1);
	}
}
```



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
int32 SWindow::PaintSlowPath(const FSlateInvalidationContext& Context)
{
	HittestGrid->Clear();

	const FSlateRect WindowCullingBounds = GetClippingRectangleInWindow();
	const int32 LayerId = 0;
	const FGeometry WindowGeometry = GetWindowGeometryInWindow();

	int32 MaxLayerId = 0;

	//OutDrawElements.PushBatchPriortyGroup(*this);
	{
		
		MaxLayerId = Paint(*Context.PaintArgs, WindowGeometry, WindowCullingBounds, *Context.WindowElementList, LayerId, Context.WidgetStyle, Context.bParentEnabled);
	}

	//OutDrawElements.PopBatchPriortyGroup();



	return MaxLayerId;
}
```

```
```







## 研究一下各种事件的调用流程

我上面所说的各种事件，包括且不限于Mouse Move 等等 ButtonDown。这些个事件最终会到哪里？到FWindowsWindow。它是FGenericWindow的子类。

