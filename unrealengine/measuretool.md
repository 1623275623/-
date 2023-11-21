# MeasureTool

其中一个难度不是很高又十分值得学习的插件项目是安宁的Measure Tool.这是一个简单的测量工具。

有一个主工具类：MeasureToolEdMode，三个工具类ObjectRulerTool、PointerRulerTool、AnnotationTool。还有一个主要负责工具UI的类是MeasureToolEdModeToolKit.h这个类的具体原理自己是搞不太清楚的。

那么首先我想探讨这个问题是这个工具是被谁调用的，是在哪里被调用的，这些函数㐊如何被执行的。

其中总的一个位置是在FEditorViewportClient这个类里面和编辑器Mode相关的是`TSharedPtr<FEditorteModeTools> ModeTools;` 特别的有规律EditorMode也需要在使用的时候进行注册，如果在不使用的情况进行取消注册。

当我们定睛去仔细观察这个FEditorModeTools的时候发现它里面保存的模式的数组是UEdMode，但是我们的工具类继承的类是FEdMode。

那这个UEdMode和FEdMode 之间是如何进行转换的？

为什么要进行转换？

两个类之间的区别在哪里？

有这么一个类作为一个纽带联系这FEdMode 和 UEdMode.



<img alt="" class="gitbook-drawing">
