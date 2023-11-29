# Page 1

<img src="../../.gitbook/assets/file.excalidraw (1).svg" alt="" class="gitbook-drawing">

关于**FMultiBlock**

```cpp

// We're friends with SMultiBoxWidget so that it can call MakeWidget() directly
friend class SMultiBoxWidget;  

/** Direct processing of actions. Will use these actions if there is not UICommand associated with this block that handles actions*/
FUIAction DirectActions;

/** The action associated with this block (can be null for some actions) */
const TSharedPtr< const FUICommandInfo > Action;

/** The list of mappings from command info to delegates that should be called. This is here for quick access. Can be null for some widgets*/
const TSharedPtr< const FUICommandList > ActionList;

/** Optional extension hook which is used for debug display purposes, so users can see what hooks are where */
FName ExtensionHook;

/** Type of MultiBlock */
EMultiBlockType Type;

/** Name to identify a widget for tutorials */
FName TutorialHighlightName;

FName StyleNameOverride;

/** Whether this block can be searched */
bool bSearchable;

/** Whether this block is part of the heading blocks for a section */
bool bIsPartOfHeading;
```

```cpp
class FBaseMenuBuilder
{
    /** True if clicking on a menu entry closes itself only and its children and not the entire stack */
	bool bCloseSelfOnly;
}
```



```cpp
class FMenuBuilder
{
    	/** Current extension hook name for sections to determine where sections begin and end */
	FName CurrentSectionExtensionHook;
	
	/** Any pending section's heading text */
	FText CurrentSectionHeadingText;
	
	/** True if there is a pending section that needs to be applied */
	bool bSectionNeedsToBeApplied;

	/** Whether this menu is searchable */
	bool bSearchable;

	/** Whether the search algorithm should walk down this menu sub-menu(s) (if the menu is searchable in first place). */
	bool bRecursivelySearchable;

	/** Whether menu is currently being edited */
	bool bIsEditing;
}
```

从继承链中发现主要起关键作用的还是最上面的父类。



```cpp

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Tool Menus")
FName Name;

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Tool Menus")
FToolMenuOwner Owner;

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Tool Menus")
EMultiBlockType Type;

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Tool Menus")
EUserInterfaceActionType UserInterfaceActionType;

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Tool Menus")
FName TutorialHighlightName;

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Tool Menus")
FToolMenuInsert InsertPosition;

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Tool Menus")
bool bShouldCloseWindowAfterMenuSelection;

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Tool Menus")
TObjectPtr<UToolMenuEntryScript> ScriptObject;

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Tool Menus")
FName StyleNameOverride;

FToolMenuEntrySubMenuData SubMenuData;

FToolMenuEntryToolBarData ToolBarData;

FToolMenuEntryWidgetData WidgetData;

UE_DEPRECATED(5.0, "Use MakeCustomWidget instead")
FNewToolMenuWidget MakeWidget;

/** Optional delegate that returns a widget to use as this menu entry */
FNewToolMenuCustomWidget MakeCustomWidget;

TAttribute<FText> Label;
TAttribute<FText> ToolTip;
TAttribute<FSlateIcon> Icon;


private:
friend class UToolMenus;
friend class UToolMenuEntryExtensions;
friend class FPopulateMenuBuilderWithToolMenuEntry;

FToolUIActionChoice Action;

FToolMenuStringCommand StringExecuteAction;

TSharedPtr< const FUICommandInfo > Command;
TSharedPtr< const FUICommandList > CommandList;

FNewToolMenuSectionDelegate Construct;
FNewToolMenuDelegateLegacy ConstructLegacy;

bool bAddedDuringRegister;

UPROPERTY()
bool bCommandIsKeybindOnly;
```



<img alt="" class="gitbook-drawing">

### 创建子菜单 | Create SubMenu

自己大致总结的有两种添加子菜单的形式。一种是通过menu的形式添加子菜单。另一种方式是通过Section添加子菜单。而且子类还有很多。







