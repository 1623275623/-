---
cover: >-
  https://images.unsplash.com/photo-1593720213411-697ba0ac0162?crop=entropy&cs=srgb&fm=jpg&ixid=M3wxOTcwMjR8MHwxfHNlYXJjaHwxfHxjb25maWd1cmF0aW9ufGVufDB8fHx8MTcwMDE4Nzg4Nnww&ixlib=rb-4.0.3&q=85
coverY: 0
---

# 🔧 Configuration

### 配置文件的分类

**General**

* `Engine`
* `Game`
* `Input`
* `DeviceProfiles`
* `GameUserSettings`
* `Scalability`
* `RuntimeOptions`
* `InstallBundle`
* `Hardware`
* `GameplayTags`

**Editor-Only**

* `Editor`
* `EditorPerProjectUserSettings`
* `EditorSettings`
* `EditorKeyBindings`
* `EditorLayout`

**Desktop-Only**

* `Compat`
* `Lightmass`

在相同分类下的配置文件是以层级的结构来组织的。如果有相同的键值对存在相同的分类下，那么后者的值会覆盖掉前者的值。

存储在引擎目录下的配置文件作用于使用该引擎的所有项目。在项目文件夹下的配置文件只作用于特定的项目。位于某个平台下的配置文件也仅作用于特定的平台。

1. `Engine/Config/Base.ini`
2. `Engine/Config/Base<CATEGORY>.ini`
3. `Engine/Config/<PLATFORM>/Base<PLATFORM><CATEGORY>.ini`
4. `Engine/Platforms/<PLATFORM>/Config/Base<PLATFORM><CATEGORY>.ini`
5. `<PROJECT_DIRECTORY>/Config/Default<CATEGORY>.ini`
6. `Engine/Config/<PLATFORM>/<PLATFORM><CATEGORY>.ini`
7. `Engine/Platforms/<PLATFORM>/Config/<PLATFORM><CATEGORY>.ini`
8. `<PROJECT_DIRECTORY>/Config/<PLATFORM>/<PLATFORM><CATEGORY>.ini`
9. `<PROJECT_DIRECTORY>/Platforms/<PLATFORM>/Config/<PLATFORM><CATEGORY>.ini`
10. `<LOCAL_APP_DATA>/Unreal Engine/Engine/Config/User<CATEGORY>.ini`
11. `<MY_DOCUMENTS>/Unreal Engine/Engine/Config/User<CATEGORY>.ini`
12. `<PROJECT_DIRECTORY>/Config/User<CATEGORY>.ini`

{% hint style="info" %}
对于配置文件层级相关方面的任何信息 `Engine/Source/Runtime/Core/Public/Misc/ConfigHierarchy.h`
{% endhint %}

### Use Configuration Variables in Code

我们可以自动应用config variable 到 `UPROPERTIES` 和 `USTRUCT` 或者手动的从config manager 当中读取它们。



### Apply Configuration Settings to Variables

#### Automatically

你可以定义一个类去自动的从层级配置文件当中加载数值。

#### Section Format

为了能够在你的模块当中自动的加载配置设置`[Section]` 的格式会像下面这样。

`[/Script/ModuleName.ClassName]`

上面的内容的意思是：

* `ModuleName` 是className所在模块内定义的模块的名字。
* `ClassName` 是问题中类的名字。

{% hint style="info" %}
`ClassName` 是没有U或者A前缀的。
{% endhint %}

Steps to Automatically Load Config Variables

假设你有一个名字是`MyGameModule` 的模块类的名字是`AMyConfigActor` 并且这个类中有一个变量是你想要在配置文件中修改的。

* 需要在`UCLASS` 当中指定读取的配置文件的类型.

```
UCLASS(config=Game)
class AMyConfigActor : public UObject
```

* 将任何你想在配置文件中配置的变量声明为`Config` ：

```cpp
UPROPERTY(Config)
int32 MyConfigVariable;
```

* 设置前面的变量在你选择的分类下的任何配置文件的层级当中。例如前面你选的是游戏分类，下面的配置可以在项目目录下的DefacultGame.ini文件中设置。

```ini
[/Script/MyGameModule.MyConfigActor]
MyConfigVariable = 3
```

你的类应该看起来像这样:

```cpp
UCLASS(config=Game)
class AMyConfigActor : public UObject
{
    GENERATED_BODY()

    UPROPERTY(Config)
    int32 MyConfigVariable;
}
```

前面讲解的是自动的方式，这里我们还有手动获取配置变量的方式。

#### Manually

配置文件会加载所有的在配置文件中的声明，无论实际的配置变量是否在C++中存在。这意味着你可以查询任何部分的配置变量。例如你有下面的配置文件:`DefaultGame.ini` ：

```
[MyCategoryName]
MyVariable = 2
```

你可以使用下面的代码将这个值读入任何的文件。

```cpp
int MyConfigVariable;
GConfig->GetInt(TEXT("MyCategoryName"), TEXT("MyVariable"), MyConfigVariable, GGameIni);
```

`MyConfigVariable` 现在的值是2.

手动读取选项

#### fuinctions

下面的函数可以在头文件

`Engine/Source/Runtime/Core/Public/Misc/ConfigCacheIni.h` \
中找到。

* `GetBool`
* `GetInt`
* `GetInt64`
* `GetFloat`
* `GetDouble`
* `GetString`
* `GetText`
* `GetArray`



## Edit Configuration Settings

我们可以通过下面的两种方式修改配置文件的信息：

1. 直接修改配置文件
2. 通过Unreal Editor中的Project Settings来修改暴露在Project Settings中的值。

#### 在代码中保存配置设置





## Related Console Commands

可以通过控制台命令\`GetIni来获取任何配置文件中的值。





| **Type**           | **Section**                                  | **Description**                                                                                                          |
| ------------------ | -------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| Rendering          | `[/Script/Engine.RendererSettings]`          | Any console variable starting with `r.`                                                                                  |
| Rendering Override | `[/Script/Engine.RendererOverrideSettings]`  | Specifically for the console variable `r.SupportAllShaderPermutations`                                                   |
| Streaming          | `[/Script/Engine.StreamingSettings]`         | Any console variable starting with `s.`                                                                                  |
| Garbage Collection | `[/Script/Engine.GarbageCollectionSettings]` | Any console variable starting with `gc.`                                                                                 |
| Network Settings   | `[/Script/Engine.NetworkSettings]`           | Only for the console variables `n.VerifyPeer`, `p.EnableMultiplayerWorldOriginRebasing`, and `NetworkEmulationProfiles`. |
| Cooker Settings    | `[/Script/UnrealEd.CookerSettings]`          | Any console variable starting with `cook.`                                                                               |

### Useful Source Files for More Information <a href="#usefulsourcefilesformoreinformation" id="usefulsourcefilesformoreinformation"></a>

下面的引擎文件提供了关于配置系统及其组件的更多信息

* ConfigCacheIni
* CoreGlobals
* ConfigHierarchy
