---
cover: >-
  https://images.unsplash.com/photo-1560520031-3a4dc4e9de0c?crop=entropy&cs=srgb&fm=jpg&ixid=M3wxOTcwMjR8MHwxfHNlYXJjaHw1fHxQcm9wZXJ0eXxlbnwwfHx8fDE3MDExODIxMzF8MA&ixlib=rb-4.0.3&q=85
coverY: 0
---

# FProperty

我写这个文章的原因就是自己很菜，不会用啊，虽然简单了解了虚幻的反射体系但是自己还有很多问题需要解决？那么我想用下面这篇文章来解决的问题是：

1. 如何将枚举转发为字符串？
2. 如何能够使用反射来修改并且便利UE容器，这里的容器包括TMap、TArray、TSet。
3. 可以使用反射获取成员变量的UProperty里面的metaData信息。
4. 使用FProperty获取成员变量类型是类或者是结构体的变量。
5. 探索极致骚操作
6. 听说FProperty和蓝图的变量有一腿，想知道是怎么回事.

废话不多说我们现在来看这些操作是如何用代码实现的，完成这些之后再看一看FProperty的原理到底是什么？结合着虚幻先贤们的步伐走一走。

### 使用FProperty访问基础变量：

```
int* content = Property->ContainerPtrToValuePtr<int>(SelectedActor);		PRINT("the member content is %d",*content);//因为我们获取到的是指针所以我们也可以通过指针对值进行修改
```

### 使用FProperty便利数组TArray：

```
FArrayProperty* ArrayProperty = CastField<FArrayProperty>(Property);		void* Num = ArrayProperty->ContainerPtrToValuePtr<void>(SelectedActor);		TArray<int>* Array = static_cast<TArray<int>*>(Num);		for(int i=0;i<5;++i)		{			UE_LOG(LogTemp,Error,TEXT("the num is %d"),(*Array)[i]);		}		UE_LOG(LogTemp,Error,TEXT("The array Num is %d"),Array->Num());
```

### 使用FProperty便利TSet

```
				{					FProperty* Property = MapHelper.GetKeyProperty();					if(Property->IsA<FNameProperty>())					{						FName* Name = Property->ContainerPtrToValuePtr<FName>(MapHelper.GetPairPtr(Index));						PRINT("the result is %s.",*(Name->ToString()));					}				}			}							PRINT("The Map Num is %d",Num);			}		}	}
```

上面的代码都是自己知道自己访问的类型的情况下，如果自己不知道类型或者自己访问的变量可能是任意的类型的话真的头很大。针对上面这个问题目前自己知道的一个唯一办法就是使用FPorperty 中的 IsA<检测类型>()来判断该FProperty是不是某类型；下面是实际代码案例

```
if(MyProperty->IsA<FArrayProperty>()){}
```

那么困惑我很久的一个问题是：

## FProperty的原理是什么？

当我在探究一些变量的时候发现，在创建相对应的FProperty对象的时候到最后都会到FProperty这里，而且往往最关键的一步就是设置Offset的值。这个Offset的值是通过宏

```cpp
#define STRUCT_OFFSET( struc, member )  offsetof(struc, member)

#ifdef __clang__
#define STRUCT_OFFSET( struc, member )	__builtin_offsetof(struc, member)
#else
#define STRUCT_OFFSET( struc, member )	offsetof(struc, member)
#endif

#if defined _MSC_VER && !defined _CRT_USE_BUILTIN_OFFSETOF
    #ifdef __cplusplus
        #define offsetof(s,m) ((::size_t)&reinterpret_cast<char const volatile&>((((s*)0)->m)))
    #else
        #define offsetof(s,m) ((size_t)&(((s*)0)->m))
    #endif
#else
    #define offsetof(s,m) __builtin_offsetof(s,m)
#endif


```

上面这内容自己是完全看不懂的，老实说，但是这就是获取一个变量在一个类中的偏移值的具体手法。

下面我们将结合各种网上的学习资料来解决这个问题。





\
\
