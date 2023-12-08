# API

### 基础内容

### 实用API 总结

#### LoadObject

```cpp
/** 
 * Load an object. 
 * @see StaticLoadObject()
 */
template< class T > 
inline T* LoadObject( UObject* Outer, const TCHAR* Name, const TCHAR* Filename=nullptr, uint32 LoadFlags=LOAD_None, UPackageMap* Sandbox=nullptr, const FLinkerInstancingContext* InstancingContext=nullptr )
{
	return (T*)StaticLoadObject( T::StaticClass(), Outer, Name, Filename, LoadFlags, Sandbox, true, InstancingContext );
}
```

然后直接调用函数`StaticLoadObject()` 该函数内部的代码如下：

```
FUObjectThreadContext& ThreadContext = FUObjectThreadContext::Get();
	if ((GSyncLoadUsingAsyncLoaderCount == 0) && ThreadContext.IsRoutingPostLoad && IsInAsyncLoadingThread())
	{
		UE_LOG(LogUObjectGlobals, Warning, TEXT("Calling StaticLoadObject(\"%s\", \"%s\", \"%s\") during PostLoad of %s is illegal and will crash in a cooked runtime"), 
			*GetFullNameSafe(ObjectClass),
			*GetFullNameSafe(InOuter),
			InName,
			*GetFullNameSafe(ThreadContext.CurrentlyPostLoadedObjectByALT));
	}
```

有一说一上面的内容说实话我也不清楚具体是在干什么，但是可以肯定的是因为我们符合一些条件然后输出了警告。

`UObject* Result = StaticLoadObjectInternal(ObjectClass, InOuter, InName, Filename, LoadFlags, Sandbox, bAllowObjectReconciliation, InstancingContext);`

然后就有调用上面的函数继续加载。

如果上面的函数没有加载到东西，Result是一个空，那么不出意外的是会进入下面的If当中。

```
if (!Result)
	{
		FString ObjectName = InName;
		ResolveName(InOuter, ObjectName, true, true, LoadFlags & LOAD_EditorOnly, InstancingContext);

		if (InOuter == nullptr || FLinkerLoad::IsKnownMissingPackage(FName(*InOuter->GetPathName())) == false)
		{
			// we haven't created or found the object, error
			FFormatNamedArguments Arguments;
			Arguments.Add(TEXT("ClassName"), ObjectClass ? FText::FromString(ObjectClass->GetName()) : NSLOCTEXT("Core", "None", "None"));
			Arguments.Add(TEXT("OuterName"), InOuter ? FText::FromString(InOuter->GetPathName()) : NSLOCTEXT("Core", "None", "None"));
			Arguments.Add(TEXT("ObjectName"), FText::FromString(ObjectName));
			const FString Error = FText::Format(NSLOCTEXT("Core", "ObjectNotFound", "Failed to find object '{ClassName} {OuterName}.{ObjectName}'"), Arguments).ToString();
			SafeLoadError(InOuter, LoadFlags, *Error);

			if (InOuter && !InOuter->HasAnyFlags(RF_WasLoaded))
			{
				// Stop future repeated warnings
				FLinkerLoad::AddKnownMissingPackage(FName(*InOuter->GetPathName()));
			}
		}
	}
```

那上面的if干了什么事情呢？

### UObject 系列

\


\


#### FindObject

```cpp
/**
 * Find an optional object.
 * @see StaticFindObject()
 */
template< class T > 
inline T* FindObject( UObject* Outer, const TCHAR* Name, bool ExactClass=false )
{
	return (T*)StaticFindObject( T::StaticClass(), Outer, Name, ExactClass );
}
```

第一个参数是要指定实在那个UPackage中寻找，第二个参数是要指定寻找对象的名字。下面看`StaticFindObject`

```cpp
UObject* StaticFindObject( UClass* ObjectClass, UObject* InObjectPackage, const TCHAR* OrigInName, bool bExactClass )
{
	INC_DWORD_STAT(STAT_FindObject);

	// Resolve the object and package name.
	/*如果我们输入函数的包名是 ANY_PACKAGE_DEPRECATED 那么 bAnyPackage 就是 true 否则就是false
	* ObjectPackage 如果 bAnyPackage 是true 就是 nullptr 否则就是 对应的包
	*/
	const bool bAnyPackage = InObjectPackage == ANY_PACKAGE_DEPRECATED;  
	UObject* ObjectPackage = bAnyPackage ? nullptr : InObjectPackage;

	UObject* MatchingObject = nullptr;
#if WITH_EDITOR
	// If the editor is running, and T3D is being imported, ensure any packages referenced are fully loaded.
	if ((GIsEditor == true) && (GIsImportingT3D == true))
	{
		MatchingObject = LoadObjectWhenImportingT3D(ObjectClass, OrigInName);
		if (MatchingObject)
		{
			return MatchingObject;
		}
	}
#endif	//#if !WITH_EDITOR

	FName ObjectName;
	
	/*下面的内容是根据 bAnyPackage 的是与否采取的两套不同的处理策略 
	* 如果 bAnyPackage 是 false 那么就 使用 ResolveName 将最内部的对象名字和包名找出来 用于下一步的寻找
	* 如果我们 bAnyPackage 是 true就先将 InName中前面的对象名去掉 通过函数 ConstructorHelpers::StripObjectClass(InName);完成
	* 然后对处理后将处理后的ObjectName传入 函数 StaticFindObject
	*/
	// Don't resolve the name if we're searching in any package
	if (!bAnyPackage)
	{
		FString InName = OrigInName;
		if (!ResolveName(ObjectPackage, InName, false, false))
		{
			return nullptr;
		}
		ObjectName = FName(*InName, FNAME_Add);
	}
	else
	{
		FString InName = OrigInName;
		ConstructorHelpers::StripObjectClass(InName);

		ObjectName = FName(*InName, FNAME_Add);
	}
	PRAGMA_DISABLE_DEPRECATION_WARNINGS
	return StaticFindObjectFast(ObjectClass, ObjectPackage, ObjectName, bExactClass, bAnyPackage);
	PRAGMA_ENABLE_DEPRECATION_WARNINGS
}
```

StaticFindObjectFast -> StaticFindObjectFastInternal ->

```cpp
UObject* StaticFindObjectFastInternal(const UClass* ObjectClass, const UObject* ObjectPackage, FName ObjectName, bool bExactClass, bool bAnyPackage, EObjectFlags ExcludeFlags, EInternalObjectFlags ExclusiveInternalFlags)
{
	INC_DWORD_STAT(STAT_FindObjectFast);

	check(ObjectPackage != ANY_PACKAGE_DEPRECATED); // this could never have returned anything but nullptr

	// If they specified an outer use that during the hashing
	FUObjectHashTables& ThreadHash = FUObjectHashTables::Get();
	UObject* Result = StaticFindObjectFastInternalThreadSafe(ThreadHash, ObjectClass, ObjectPackage, ObjectName, bExactClass, bAnyPackage, ExcludeFlags, ExclusiveInternalFlags);
	return Result;
}

```

在上面的函数中是第一次祭出了 FUobjectHashTables 是为全局的UObject管理表；然后调用函数 `StaticFindObjectFastInternalThreadSafe` 并将全局表作为参数传入。

```cpp
UObject* StaticFindObjectFastInternalThreadSafe(FUObjectHashTables& ThreadHash, const UClass* ObjectClass, const UObject* ObjectPackage, FName ObjectName, bool bExactClass, bool bAnyPackage, EObjectFlags ExcludeFlags, EInternalObjectFlags ExclusiveInternalFlags)
{
	ExclusiveInternalFlags |= EInternalObjectFlags::Unreachable;

	// If they specified an outer use that during the hashing
	UObject* Result = nullptr;
	if (ObjectPackage != nullptr)
	{
		int32 Hash = GetObjectOuterHash(ObjectName, (PTRINT)ObjectPackage);
		FHashTableLock HashLock(ThreadHash);
		for (TMultiMap<int32, uint32>::TConstKeyIterator HashIt(ThreadHash.HashOuter, Hash); HashIt; ++HashIt)
		{
			uint32 InternalIndex = HashIt.Value();
			UObject* Object = static_cast<UObject*>(GUObjectArray.IndexToObject(InternalIndex)->Object);
			if
				/* check that the name matches the name we're searching for */
				((Object->GetFName() == ObjectName)

				/* Don't return objects that have any of the exclusive flags set */
				&& !Object->HasAnyFlags(ExcludeFlags)

				/* check that the object has the correct Outer */
				&& Object->GetOuter() == ObjectPackage

				/** If a class was specified, check that the object is of the correct class */
				&& (ObjectClass == nullptr || (bExactClass ? Object->GetClass() == ObjectClass : Object->IsA(ObjectClass)))
				
				/** Include (or not) pending kill objects */
				&& !Object->HasAnyInternalFlags(ExclusiveInternalFlags))
			{
				checkf(!Object->IsUnreachable(), TEXT("%s"), *Object->GetFullName());
				if (Result)
				{
					UE_LOG(LogUObjectHash, Warning, TEXT("Ambiguous search, could be %s or %s"), *GetFullNameSafe(Result), *GetFullNameSafe(Object));
				}
				else
				{
					Result = Object;
				}
#if (UE_BUILD_SHIPPING || UE_BUILD_TEST)
				break;
#endif
			}
		}

#if WITH_EDITOR
		// if the search fail and the OuterPackage is a UPackage, lookup potential external package
		if (Result == nullptr && ObjectPackage->IsA(UPackage::StaticClass()))
		{
			Result = StaticFindObjectInPackageInternal(ThreadHash, ObjectClass, static_cast<const UPackage*>(ObjectPackage), ObjectName, bExactClass, ExcludeFlags, ExclusiveInternalFlags);
		}
#endif
	}
	else
	{
		FObjectSearchPath SearchPath(ObjectName);

		const int32 Hash = GetObjectHash(SearchPath.Inner);
		FHashTableLock HashLock(ThreadHash);

		FHashBucket* Bucket = ThreadHash.Hash.Find(Hash);
		if (Bucket)
		{
			for (FHashBucketIterator It(*Bucket); It; ++It)
			{
				UObject* Object = (UObject*)*It;

				if
					((Object->GetFName() == SearchPath.Inner)

					/* Don't return objects that have any of the exclusive flags set */
					&& !Object->HasAnyFlags(ExcludeFlags)

					/*If there is no package (no InObjectPackage specified, and InName's package is "")
					and the caller specified any_package, then accept it, regardless of its package.
					Or, if the object is a top-level package then accept it immediately.*/
					&& (bAnyPackage || !Object->GetOuter())

					/** If a class was specified, check that the object is of the correct class */
					&& (ObjectClass == nullptr || (bExactClass ? Object->GetClass() == ObjectClass : Object->IsA(ObjectClass)))

					/** Include (or not) pending kill objects */
					&& !Object->HasAnyInternalFlags(ExclusiveInternalFlags)

					/** Ensure that the partial path provided matches the object found */
					&& SearchPath.MatchOuterNames(Object->GetOuter()))
				{
					checkf(!Object->IsUnreachable(), TEXT("%s"), *Object->GetFullName());
					if (Result)
					{
						UE_LOG(LogUObjectHash, Warning, TEXT("Ambiguous path search, could be %s or %s"), *GetFullNameSafe(Result), *GetFullNameSafe(Object));
					}
					else
					{
						Result = Object;
					}
#if (UE_BUILD_SHIPPING || UE_BUILD_TEST)
					break;
#endif
				}
			}
		}
	}
	// Not found.
	return Result;
}
```

上面的函数根据If分为两个部分，进行区分的条件还是是否指定有效的UPackage。但是大体的步骤是一样的，先看没有进行明确指定UPakcage的版本。大体流程是根据Name计算出Hash，根据Hash在表中查找除Bucket，这个是全局表为了提升性能独有的结构，所有具有相同Hash的对象都在这个Bucket里面。然后通过for循环遍历所有BUcket里面的对象找寻目标对象，如果找到就返回。寻找的条件依次为：

1. 检查对象姓名
2. ExcludeFlags 找的对象应该不包含含有ExcludeFlags 的对象
3. 如果指定了类 UClass 要检测找的对象是是参数穿进来的类型
4. 排除所有含有 EInternalObjectFlags 的对象
5. 对对象的父对象也就是Outer对象进行比较

如果上面的条件全部都符合就返回该对象

如何判断某个对象是否属于某个类呢？

直接使用 == 判断 `&& (ObjectClass == nullptr || (bExactClass ? Object->GetClass() == ObjectClass : Object->IsA(ObjectClass)))`

其中bExactClass是一个bool值的参数意思是 是否要严格匹配参数中的Class

如果bExactClass是true 那么需要做的就是 所查对象的类型和参数穿进来的类型要完全匹配。如果bExactClass是false 表示的意思就是 只要属于穿进来的类的子类也可以接受。通过IsA函数来进行判断。

\


#### LoadObject函数

首先是查找，如果实在是找不到最终会调用到LoadPackage 这一部分就要牵涉到资源管理的内容了。

\


### SpawnActor

```
AActor* UWorld::SpawnActor( UClass* Class, FVector const* Location, FRotator const* Rotation, const FActorSpawnParameters& SpawnParameters )
{
	FTransform Transform;
	if (Location)
	{
		Transform.SetLocation(*Location);
	}
	if (Rotation)
	{
		Transform.SetRotation(FQuat(*Rotation));
	}

	return SpawnActor(Class, &Transform, SpawnParameters);
}
```

且看这到底是如何一步步执行的。

\


#### 深入理解CDO

CDO 是 Class Default Object

从源码角度来看，这个过程开始于：ProcessNewlyLoadedUObjects -UObjectLoadAllCompiledInDefaultProperties -> UObject\* GetDefaultObject(bool bCreateIfNeeded = true) const -> InternalCreateDefaultObjectWrapper() -> const\_cast\<UClass\*>(this)->CreateDefaultObject();

```
UObject* UClass::CreateDefaultObject()
{
	if ( ClassDefaultObject == NULL )
	{
		ensureMsgf(!bLayoutChanging, TEXT("Class named %s creating its CDO while changing its layout"), *GetName());

		UClass* ParentClass = GetSuperClass();
		UObject* ParentDefaultObject = NULL;
		if ( ParentClass != NULL )
		{
			UObjectForceRegistration(ParentClass);
			ParentDefaultObject = ParentClass->GetDefaultObject(); // Force the default object to be constructed if it isn't already
			check(GConfig);
			if (GEventDrivenLoaderEnabled && EVENT_DRIVEN_ASYNC_LOAD_ACTIVE_AT_RUNTIME)
			{ 
				check(ParentDefaultObject && !ParentDefaultObject->HasAnyFlags(RF_NeedLoad));
			}
		}

		if ( (ParentDefaultObject != NULL) || (this == UObject::StaticClass()) )
		{
			// If this is a class that can be regenerated, it is potentially not completely loaded.  Preload and Link here to ensure we properly zero memory and read in properties for the CDO
			if( HasAnyClassFlags(CLASS_CompiledFromBlueprint) && (PropertyLink == NULL) && !GIsDuplicatingClassForReinstancing)
			{
				auto ClassLinker = GetLinker();
				if (ClassLinker)
				{
					if (!GEventDrivenLoaderEnabled)
					{
						UField* FieldIt = Children;
						while (FieldIt && (FieldIt->GetOuter() == this))
						{
							// If we've had cyclic dependencies between classes here, we might need to preload to ensure that we load the rest of the property chain
							if (FieldIt->HasAnyFlags(RF_NeedLoad))
							{
								ClassLinker->Preload(FieldIt);
							}
							FieldIt = FieldIt->Next;
						}
					}
					
					StaticLink(true);
				}
			}

			// in the case of cyclic dependencies, the above Preload() calls could end up 
			// invoking this method themselves... that means that once we're done with  
			// all the Preload() calls we have to make sure ClassDefaultObject is still 
			// NULL (so we don't invalidate one that has already been setup)
			if (ClassDefaultObject == NULL)
			{
				// RF_ArchetypeObject flag is often redundant to RF_ClassDefaultObject, but we need to tag
				// the CDO as RF_ArchetypeObject in order to propagate that flag to any default sub objects.
				ClassDefaultObject = StaticAllocateObject(this, GetOuter(), NAME_None, EObjectFlags(RF_Public|RF_ClassDefaultObject|RF_ArchetypeObject));
				check(ClassDefaultObject);
				// Register the offsets of any sparse delegates this class introduces with the sparse delegate storage
				for (TFieldIterator<FMulticastSparseDelegateProperty> SparseDelegateIt(this, EFieldIteratorFlags::ExcludeSuper, EFieldIteratorFlags::ExcludeDeprecated); SparseDelegateIt; ++SparseDelegateIt)
				{
					const FSparseDelegate& SparseDelegate = SparseDelegateIt->GetPropertyValue_InContainer(ClassDefaultObject);
					USparseDelegateFunction* SparseDelegateFunction = CastChecked<USparseDelegateFunction>(SparseDelegateIt->SignatureFunction);
					FSparseDelegateStorage::RegisterDelegateOffset(ClassDefaultObject, SparseDelegateFunction->DelegateName, (size_t)&SparseDelegate - (size_t)ClassDefaultObject);
				}
				EObjectInitializerOptions InitOptions = EObjectInitializerOptions::None;
				if (!HasAnyClassFlags(CLASS_Native | CLASS_Intrinsic))  
				{
					// Blueprint CDOs have their properties always initialized.
					InitOptions |= EObjectInitializerOptions::InitializeProperties;
				}
				(*ClassConstructor)(FObjectInitializer(ClassDefaultObject, ParentDefaultObject, InitOptions));
				if (GetOutermost()->HasAnyPackageFlags(PKG_CompiledIn) && !GetOutermost()->HasAnyPackageFlags(PKG_RuntimeGenerated))
				{
					TCHAR PackageName[FName::StringBufferSize];
					TCHAR CDOName[FName::StringBufferSize];
					GetOutermost()->GetFName().ToString(PackageName);
					GetDefaultObjectName().ToString(CDOName);
					NotifyRegistrationEvent(PackageName, CDOName, ENotifyRegistrationType::NRT_ClassCDO, ENotifyRegistrationPhase::NRP_Finished, nullptr, false, ClassDefaultObject);
				}
				ClassDefaultObject->PostCDOContruct();
			}
		}
	}
	return ClassDefaultObject;
}
```

关于上面一直出现的两个ClassFlags的意思到底是什么？

> CLASS\_Native指的是在C++里定义的类，用来和蓝图类区分。CLASS\_Intrinsic指的是告诉UHT不要帮我生成反射代码，我要自己写，一般是引擎内部的类才会用到。

`(*ClassConstructor)(FObjectInitializer(ClassDefaultObject, ParentDefaultObject, InitOptions));`

```cpp
#define DEFINE_FORBIDDEN_DEFAULT_CONSTRUCTOR_CALL(TClass) \
	static_assert(false, "You have to define " #TClass "::" #TClass "() or " #TClass "::" #TClass "(const FObjectInitializer&). This is required by UObject system to work correctly.");

#define DEFINE_DEFAULT_CONSTRUCTOR_CALL(TClass) \
	static void __DefaultConstructor(const FObjectInitializer& X) { new((EInternal*)X.GetObj())TClass; }

#define DEFINE_DEFAULT_OBJECT_INITIALIZER_CONSTRUCTOR_CALL(TClass) \
	static void __DefaultConstructor(const FObjectInitializer& X) { new((EInternal*)X.GetObj())TClass(X); }
```

上面的ClassConstructor这个函数指针的设定是在GetPrivateStaticClassBody()这个函数里面设定的。

```
template<class T>
void InternalConstructor( const FObjectInitializer& X )
{ 
	T::__DefaultConstructor(X);
}
```

然后这里的\_\_DefaultConstructor 是有上面的宏定义的。至于具体那个类使用那个下面有更加详细的阐述。

[NewObject过程剖析](https://zhuanlan.zhihu.com/p/357510279)

CDO是在引擎初始化的时候在为每一个类创建UClass的时候创建的，然后它很自然的就会执行构造函数去设置变量的默认值。这也是为什么你不能在一个类的构造函数里写有关游戏逻辑的代码。

CDO 在引擎中很多地方都会被用到。

Blueprint generated classes (BPGC) are not always loaded at all times. Using FindObject with a blueprint class path might return null if the package is not loaded at this time. If nothing references your uasset, chances are it will never be loaded.

But as long as your uasset package exists you can load it by path using LoadObject (or other more recent sync/async loading techniques), eg :

```
UClass* MyClass = LoadObject<UClass>(nullptr, TEXT("/Game/Path/BP_Thing.BP_Thing_C");
```

BPGCs and their CDOs are serialized into uasset packages when you compile blueprint in editor or package the game.\
Upon loading package, the Linker resolves dependencies and de-serializes package contents, thus creating the BPGC and CDO contained within.

Objects contained in the package are created in FLinkerLoad::CreateExport, but at this point they are just an empty shell. Their data is deserialized from disk at a later point when FLinkerLoad::Preload is called.

That is of course assuming you are using traditional sync loading. I’m not familiar with the more recent loading techniques but they probably rely on async loader in AsyncLoading.cpp or AsyncLoading2.cpp, which seem to add a whole new layer on top of LinkerLoad.

[CDO](https://link.zhihu.com/?target=https%3A//forums.unrealengine.com/t/what-is-cdo/310820/3%3Fu%3Donline\_learner\_qf3r1)

[CDO part 2](https://link.zhihu.com/?target=https%3A//forums.unrealengine.com/t/what-is-cdo-part-2/349582/3)

> FObjectInitializer is Object that hold information what to do after construction

FObjectInitializer 是一个对象拥有在构造之后做什么的对象

CDO的主要功能是持有默认值，所以如果你想要获取你的类的默认值，你应该从UClass里获取CDO然后获取那个变量。

CDO还可以在属性设置方面给我们提供帮助。

### Template

判断两个类是否有父子关系 也不一定非要是

```
if(TIsDerivedFrom<AActor,UObject>::Value)
{
	PRINT_LOG("Yes");
}
```

这里根据结果Value返回的就是bool 的 true 或者 false

比较两个类型是否相同，最新版官方推荐的是：

```
std::is_same<>
TIsRValueReferenceType<T>::Value
TIsReferenceType
TIsLValueReferenceType
TIsFunction
TNameOf 获取类型的 TEXT("int")格式
TIsBitwiseConstructible

/**
 * Tests if a type T is bitwise-constructible from a given argument type U.  That is, whether or not
 * the U can be memcpy'd in order to produce an instance of T, rather than having to go
 * via a constructor.
 *
 * Examples:
 * TIsBitwiseConstructible<PODType,    PODType   >::Value == true  // PODs can be trivially copied
 * TIsBitwiseConstructible<const int*, int*      >::Value == true  // a non-const Derived pointer is trivially copyable as a const Base pointer
 * TIsBitwiseConstructible<int*,       const int*>::Value == false // not legal the other way because it would be a const-correctness violation
 * TIsBitwiseConstructible<int32,      uint32    >::Value == true  // signed integers can be memcpy'd as unsigned integers
 * TIsBitwiseConstructible<uint32,     int32     >::Value == true  // and vice versa
 */

TRemoveConst

/** Gets the Nth type in a template parameter pack. N must be less than sizeof...(Types) */
template <int32 N, typename... Types>
struct TNthTypeFromParameterPack;

TIsPointer 
TIsPolymorphic
TIsSigne 
TIsTrivial 拥有没用的构造函数 
TIsArray 
TIsAbstract
TIsArithmetic
TIsFloatingPoint
TIsPODTypetemplate <typename T>
struct TIsTrivial
{
	enum { Value = TAnd<TIsTriviallyDestructible<T>, TIsTriviallyCopyConstructible<T>, TIsTriviallyCopyAssignable<T>>::Value };
}; 

/**
* Traits class which tests if a type is a core variant type (e.g. FVector, which supports FVector3f/FVector3d float/double variants.
* Can be used to determine if the provided type is a core variant type in general:
*  e.g. TIsUECoreVariant<FColor>::Value == false
*		 TIsUECoreVariant<FVector>::Value == true
*  and also to determine if it is a variant type of a  particular component type:
*	e.g TIsUECoreVariant<FVector3d, float>::Value == false
*		TIsUECoreVariant<FVector3d, double>::Value == true
*/

TIsUECoreVariant
template <typename T>
struct TIsTrivial
{
	enum { Value = TAnd<TIsTriviallyDestructible<T>, TIsTriviallyCopyConstructible<T>, TIsTriviallyCopyAssignable<T>>::Value };
};
```

\


### Console Command

```cpp
	// Add the stats to the list, note this is also the order that they get rendered in if active.
#if !UE_BUILD_SHIPPING
	EngineStats.Add(FEngineStatFuncs(TEXT("STAT_Version"), TEXT("STATCAT_Engine"), FText::FromString(TEXT("Display build version string which includes the Changelist number.")), FEngineStatRender::CreateUObject(this, &UEngine::RenderStatVersion), FEngineStatToggle(), bIsRHS));
#endif // !UE_BUILD_SHIPPING
	EngineStats.Add(FEngineStatFuncs(TEXT("STAT_NamedEvents"), TEXT("STATCAT_Engine"), FText::GetEmpty(), FEngineStatRender(), FEngineStatToggle::CreateUObject(this, &UEngine::ToggleStatNamedEvents), bIsRHS));
	EngineStats.Add(FEngineStatFuncs(TEXT("STAT_VerboseNamedEvents"), TEXT("STATCAT_Engine"), FText::GetEmpty(), FEngineStatRender(), FEngineStatToggle::CreateUObject(this, &UEngine::ToggleStatVerboseNamedEvents), bIsRHS));
	EngineStats.Add(FEngineStatFuncs(TEXT("STAT_FPS"), TEXT("STATCAT_Engine"), FText::FromString(TEXT("Display average FPS.")), FEngineStatRender::CreateUObject(this, &UEngine::RenderStatFPS), FEngineStatToggle::CreateUObject(this, &UEngine::ToggleStatFPS), bIsRHS));
	EngineStats.Add(FEngineStatFuncs(TEXT("STAT_Summary"), TEXT("STATCAT_Engine"), FText::FromString(TEXT("Display UsedPhysical memory stat.")), FEngineStatRender::CreateUObject(this, &UEngine::RenderStatSummary), FEngineStatToggle(), bIsRHS));
	EngineStats.Add(FEngineStatFuncs(TEXT("STAT_Unit"), TEXT("STATCAT_Engine"), FText::FromString(TEXT("Display frame times for Game Thread, Render Thread and GPU.  Also displays draw calls and primitive counts.")), FEngineStatRender::CreateUObject(this, &UEngine::RenderStatUnit), FEngineStatToggle::CreateUObject(this, &UEngine::ToggleStatUnit), bIsRHS));
	EngineStats.Add(FEngineStatFuncs(TEXT("STAT_DrawCount"), TEXT("STATCAT_Engine"), FText::FromString(TEXT("Display draw call counts per render pass.")), FEngineStatRender::CreateUObject(this, &UEngine::RenderStatDrawCount), FEngineStatToggle(), bIsRHS));
	EngineStats.Add(FEngineStatFuncs(TEXT("STAT_Hitches"), TEXT("STATCAT_Engine"), FText::FromString(TEXT("Display a scrolling hitch time when frame hitches occur.")), FEngineStatRender::CreateUObject(this, &UEngine::RenderStatHitches), FEngineStatToggle::CreateUObject(this, &UEngine::ToggleStatHitches), bIsRHS));
	EngineStats.Add(FEngineStatFuncs(TEXT("STAT_AI"), TEXT("STATCAT_Engine"), FText::FromString(TEXT("Display number of APlayerControllers and the number that were rendered to screen.")), FEngineStatRender::CreateUObject(this, &UEngine::RenderStatAI), FEngineStatToggle(), bIsRHS));
	EngineStats.Add(FEngineStatFuncs(TEXT("STAT_Timecode"), TEXT("STATCAT_Engine"), FText::FromString(TEXT("Display a time from a timecode provider.")), FEngineStatRender::CreateUObject(this, &UEngine::RenderStatTimecode), FEngineStatToggle(), bIsRHS));
	EngineStats.Add(FEngineStatFuncs(TEXT("STAT_FrameCounter"), TEXT("STATCAT_Engine"), FText::FromString(TEXT("Display the global, steadily increasing frame counter.")), FEngineStatRender::CreateUObject(this, &UEngine::RenderStatFrameCounter), FEngineStatToggle(), bIsRHS));
	EngineStats.Add(FEngineStatFuncs(TEXT("STAT_ColorList"), TEXT("STATCAT_Engine"), FText::FromString(TEXT("Display a list of common FColors and their names.")), FEngineStatRender::CreateUObject(this, &UEngine::RenderStatColorList), FEngineStatToggle()));
	EngineStats.Add(FEngineStatFuncs(TEXT("STAT_Levels"), TEXT("STATCAT_Engine"), FText::FromString(TEXT("Display a list of names of all the loaded levels.")), FEngineStatRender::CreateUObject(this, &UEngine::RenderStatLevels), FEngineStatToggle()));
	EngineStats.Add(FEngineStatFuncs(TEXT("STAT_Detailed"), TEXT("STATCAT_Engine"), FText::FromString(TEXT("Display detailed frame and fps timings.")), FEngineStatRender(), FEngineStatToggle::CreateUObject(this, &UEngine::ToggleStatDetailed)));
#if !UE_BUILD_SHIPPING
	EngineStats.Add(FEngineStatFuncs(TEXT("STAT_UnitMax"), TEXT("STATCAT_Engine"), FText::FromString(TEXT("Same as stat unit, but with max values also displayed.")), FEngineStatRender(), FEngineStatToggle::CreateUObject(this, &UEngine::ToggleStatUnitMax)));
	EngineStats.Add(FEngineStatFuncs(TEXT("STAT_UnitGraph"), TEXT("STATCAT_Engine"), FText::FromString(TEXT("Displays a frame time stats graphed over time.")), FEngineStatRender(), FEngineStatToggle::CreateUObject(this, &UEngine::ToggleStatUnitGraph)));
	EngineStats.Add(FEngineStatFuncs(TEXT("STAT_UnitTime"), TEXT("STATCAT_Engine"), FText::GetEmpty(), FEngineStatRender(), FEngineStatToggle::CreateUObject(this, &UEngine::ToggleStatUnitTime)));
	EngineStats.Add(FEngineStatFuncs(TEXT("STAT_Raw"), TEXT("STATCAT_Engine"), FText::GetEmpty(), FEngineStatRender(), FEngineStatToggle::CreateUObject(this, &UEngine::ToggleStatRaw)));
	EngineStats.Add(FEngineStatFuncs(TEXT("STAT_ParticlePerf"), TEXT("STATCAT_Engine"), FText::GetEmpty(), FEngineStatRender::CreateUObject(this, &UEngine::RenderStatParticlePerf), FEngineStatToggle::CreateUObject(this, &UEngine::ToggleStatParticlePerf), bIsRHS));
	EngineStats.Add(FEngineStatFuncs(TEXT("STAT_TSR"), TEXT("STATCAT_Engine"), FText::FromString(TEXT("Same as stat unit, but with additional TSR settings.")), FEngineStatRender(), FEngineStatToggle::CreateUObject(this, &UEngine::ToggleStatTSR)));
#endif // !UE_BUILD_SHIPPING
```

\


### 迭代器

//遍历世界中所有的PlayerController

```cpp
for (FConstPlayerControllerIterator Iterator = GWorld->GetPlayerControllerIterator(); Iterator; ++Iterator)
				{
					APlayerController* PlayerController = Iterator->Get();
					if (PlayerController)
					{
						if (ULocalPlayer* LocalPlayer = Cast<ULocalPlayer>(PlayerController->Player))
						{
							LocalPlayer->Exec(GWorld, *Cmd, *GLog);
							bExecuted = true;
						}
					}
				}
```

### ActorIterator

```cpp
for(TActorIterator<AActor> Iter(GEditor->GetEditorWorldContext().World());Iter;++Iter)
	{
		TArray<UStaticMeshComponent*> MeshComponents;
		Iter->GetComponents<UStaticMeshComponent>(MeshComponents);
		for(auto MeshComp: MeshComponents)
		{
			AllMeshComponents.Add(MeshComp);
		}
	}
```

\


\


```cpp
if (UClass* ClassToMatch = Cast<UClass>(ObjectToMatch))
{
	// Call it on all instances of the class
	int32 NumInstancesFound = 0;
        int32 NumInstanceCallsSucceeded = 0;
	for (FThreadSafeObjectIterator It(ClassToMatch); It; ++It)
	{
		UObject* const Obj = *It;
	        UWorld const* const ObjWorld = Obj->GetWorld();
		if (ObjWorld == InWorld)
		{
		        const bool bSucceeded = Obj->CallFunctionByNameWithArguments(Cmd, Ar, nullptr, true);
			++NumInstancesFound;
			NumInstanceCallsSucceeded += bSucceeded ? 1 : 0;
		}
	}

		Ar.Logf(TEXT("Called '%s' on %d instance(s) of class '%s' (%d succeeded)"), Cmd, NumInstancesFound, *ClassToMatch->GetPathName(), NumInstanceCallsSucceeded);
}
```

\


下面这个Iterator特别的实用。

```cpp
for(TObjectIterator<ATestActor> Iterator;Iterator;++Iterator)
{
       PRINT_LOG("The Actor Name is %s.",*Iterator->GetName());
}
```

\


\


\


\


### 关于如何使用反射：

使用反射操作类成员变量：

```cpp
for(TFieldIterator<FProperty> Iterator();Iterator;++Iterator)
{
     FString Name = Iterator->GetName().ToString();
}
```

使用反射操作类中的函数：

### &#x20;

\


\


虚幻引擎中 Timer的使用：

\


\


\


\


\


扩展内容：

如何在C++中为虚幻引擎添加控制台命令：

我们可以在很多的GamePlay框架中的类中添加比如说、HUD、Character、GameMode中。

在C++中：

```
UFUNCTION(Exec)
void YourFunction();
```
