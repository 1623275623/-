# getPrivateStaticClass()

<pre class="language-cpp"><code class="lang-cpp"><strong>void GetPrivateStaticClassBody(
</strong>	const TCHAR* PackageName,
	const TCHAR* Name,
	UClass*&#x26; ReturnClass,
	void(*RegisterNativeFunc)(),
	uint32 InSize,
	uint32 InAlignment,
	EClassFlags InClassFlags,
	EClassCastFlags InClassCastFlags,
	const TCHAR* InConfigName,
	UClass::ClassConstructorType InClassConstructor,
	UClass::ClassVTableHelperCtorCallerType InClassVTableHelperCtorCaller,
	FUObjectCppClassStaticFunctions&#x26;&#x26; InCppClassStaticFunctions,
	UClass::StaticClassFunctionType InSuperClassFn,
	UClass::StaticClassFunctionType InWithinClassFn
	)
{
	
	
GetPrivateStaticClassBody( \
			StaticPackage(), \															     	const TCHAR* PackageName,
			(TCHAR*)TEXT(#TClass) + 1 + ((StaticClassFlags &#x26; CLASS_Deprecated) ? 11 : 0), \  	const TCHAR* Name,
			Z_Registration_Info_UClass_##TClass.InnerSingleton, \								UClass*&#x26; ReturnClass,
			StaticRegisterNatives##TClass, \													void(*RegisterNativeFunc)(),
			sizeof(TClass), \																	uint32 InSize,
			alignof(TClass), \																	uint32 InAlignment,
			TClass::StaticClassFlags, \															EClassFlags InClassFlags,
			TClass::StaticClassCastFlags(), \													EClassCastFlags InClassCastFlags,
			TClass::StaticConfigName(), \														const TCHAR* InConfigName,
			(UClass::ClassConstructorType)InternalConstructor&#x3C;TClass>, \						UClass::ClassConstructorType InClassConstructor,	
			(UClass::ClassVTableHelperCtorCallerType)InternalVTableHelperCtorCaller&#x3C;TClass>, \  UClass::ClassVTableHelperCtorCallerType InClassVTableHelperCtorCaller,
			UOBJECT_CPPCLASS_STATICFUNCTIONS_FORCLASS(TClass), \								FUObjectCppClassStaticFunctions&#x26;&#x26; InCppClassStaticFunctions,
			&#x26;TClass::Super::StaticClass, \														UClass::StaticClassFunctionType InSuperClassFn,
			&#x26;TClass::WithinClass::StaticClass \													UClass::StaticClassFunctionType InWithinClassFn
		); \
		


inline static const TCHAR* StaticPackage() \
{ \
	return TPackage; \
} \

Z_Registration_Info_UClass_##TClass.InnerSingleton, \
inline static UClass* StaticClass() \
	{ \
		return GetPrivateStaticClass(); \
	} \
	
	
StaticRegisterNatives##TClass, \

static constexpr EClassFlags StaticClassFlags=EClassFlags(TStaticFlags); \



/** Returns the static cast flags for this class */ \
	inline static EClassCastFlags StaticClassCastFlags() \
	{ \
		return TStaticCastFlags; \
	} \



/**
 * A macro called from the IMPLEMENT_CLASS macro that allows the compiler to report to the UClass constructor
 * the class-specific overrides of UnrealEngine's list of reflected UObject static functions.
 */
#define UOBJECT_CPPCLASS_STATICFUNCTIONS_FORCLASS(TClass) \
	FUObjectCppClassStaticFunctions \
	( \
		UOBJECT_CPPCLASS_STATICFUNCTIONS_ALLCONFIGS(TClass) \
		UOBJECT_CPPCLASS_STATICFUNCTIONS_WITHEDITORONLYDATA(TClass) \
	)
	
	
	FUObjectCppClassStaticFunctions(
		  FUObjectCppClassStaticFunctions::AddReferencedObjectsType(&#x26;ATestActorRotation::AddReferencedObjects)
		, FUObjectCppClassStaticFunctions::DeclareCustomVersionsType(&#x26;ATestActorRotation::DeclareCustomVersions) \
		, FUObjectCppClassStaticFunctions::AppendToClassSchemaType(&#x26;ATestActorRotation::AppendToClassSchema) \
		, FUObjectCppClassStaticFunctions::DeclareConstructClassesType(&#x26;ATestActorRotation::DeclareConstructClasses)
	);

void AActor::AddReferencedObjects(UObject* InThis, FReferenceCollector&#x26; Collector)
{
	AActor* This = CastChecked&#x3C;AActor>(InThis);
	Collector.AddStableReferenceSet(&#x26;This->OwnedComponents);
#if WITH_EDITOR
	if (This->CurrentTransactionAnnotation.IsValid())
	{
		This->CurrentTransactionAnnotation->AddReferencedObjects(Collector);
	}
#endif
	Super::AddReferencedObjects(InThis, Collector);
}
	
		/**
	 * Call Ar.UsingCustomVersion for every CustomVersion that might be serialized by this class when saving.
	 * This duplicates CustomVersions declared in Serialize; Serialize still needs to declare them.
	 * Used to track which customversions will be used by a package when it is resaved.
	 * Not yet exhaustive; add CustomVersions as necessary to remove EditorDomain warnings about missing versions. */
	static void DeclareCustomVersions(FArchive&#x26; Ar, const UClass* SpecificSubclass);
	/**
	 * Append config values or settings that can change how instances of the class are cooked, including especially
	 * values that determine how version upgraded are conducted. Can also append a unique guid when necessary to
	 * invalidate previous results because serialization changed and no custom version was udpated.
	 */
	static void AppendToClassSchema(FAppendToClassSchemaContext&#x26; Context);
	/**
	 * Declare classes that can be constructed by this class during loading. This declaration is implicitly transitive;
	 * the caller is responsible for following the graph of ConstructClasses to find all transitive ConstructClasses.
	 * 
	 * @param OutConstructClasses Output list of classes that this class can construct.
	 * @param SpecificSubclass The class on which DeclaredConstructClasses was called. This can differ from the current
	 *        class because each class calls Super::DeclareConstructClasses.
	 */
	static void DeclareConstructClasses(TArray&#x3C;FTopLevelAssetPath>&#x26; OutConstructClasses, const UClass* SpecificSubclass);
	
	

	
	
	void UObject::DeclareCustomVersions(FArchive&#x26; Ar, const UClass* SpecificSubclass)
{
	// DeclareCustomVersions is called on the default object for each class
	// We first Serialize the object, which catches all the UsingCustomVersion statements
	// class authors have added unconditionally in their Serialize function
	SpecificSubclass->GetDefaultObject()->Serialize(Ar);

	// To further catch CustomVersions used by non-native structs that are in an array or don't
	// otherwise exist on the default object, Construct an instance of the struct and serialize
	// it for every struct property in the Class.
	// Since structs can contain other structs, we do a tree search of the fields.
	struct FStackData
	{
		const UStruct* Struct;
		FProperty* NextProperty;
	};
	TArray&#x3C;FStackData> StructStack;
	StructStack.Add(FStackData{ SpecificSubclass, SpecificSubclass->PropertyLink });
	TArray&#x3C;uint8> AllocationBuffer;
	while (!StructStack.IsEmpty())
	{
		FStackData&#x26; StackData = StructStack.Last();
		FProperty*&#x26; Property = StackData.NextProperty;
		bool bPushedStack = false;
		while (Property)
		{
			FProperty* InnerProperty = Property;
			Property = Property->PropertyLinkNext;
			FArrayProperty* ArrayProperty = CastField&#x3C;FArrayProperty>(InnerProperty);
			if (ArrayProperty)
			{
				InnerProperty = ArrayProperty->Inner;
			}
			FStructProperty* StructProperty = CastField&#x3C;FStructProperty>(InnerProperty);
			if (StructProperty)
			{
				UStruct* Struct = StructProperty->Struct;
				if (StructStack.ContainsByPredicate(
					[Struct](const FStackData&#x26; InStackData) { return InStackData.Struct == Struct; }))
				{
					// A cycle in the declarations (struct FA { FB B; }; struct FB { FA A; };)
					// This is invalid, but avoid an infinite loop by skipping the nested struct.
					continue;
				}
				// We handle structs that are direct members (not a pointer)
				// UObjects and structs cannot have a UObject as a direct member.
				// We rely on not having to handle it; we can construct Structs in our earliest calls,
				// but constructing a UObject during startup would cause problems.
				check(!Struct->IsA&#x3C;UClass>());
				UScriptStruct* ScriptStruct = Cast&#x3C;UScriptStruct>(Struct);
				if (ScriptStruct)
				{
					// Construct an instance and collect CustomProperties from it via Serialize
					int32 Size = ScriptStruct->GetPropertiesSize();
					int32 Alignment = ScriptStruct->GetMinAlignment();
					AllocationBuffer.SetNumUninitialized(Align(Size, Alignment) + Alignment);
					uint8* StructBytes = Align(AllocationBuffer.GetData(), Alignment);
					ScriptStruct->InitializeStruct(StructBytes);
					ScriptStruct->SerializeItem(Ar, StructBytes, nullptr);
					ScriptStruct->DestroyStruct(StructBytes);
				}
				StructStack.Add(FStackData{ Struct, Struct->PropertyLink });
				bPushedStack = true;
				break;
			}
		}
		if (!bPushedStack)
		{
			StructStack.Pop(false /* bAllowShrinking */);
		}
	}
}

void UObject::AppendToClassSchema(FAppendToClassSchemaContext&#x26; Context)
{
}

void UObject::DeclareConstructClasses(TArray&#x3C;FTopLevelAssetPath>&#x26; OutConstructClasses, const UClass* SpecificSubclass) 
{
}



#define UOBJECT_CPPCLASS_STATICFUNCTIONS_ALLCONFIGS(TClass) \
	FUObjectCppClassStaticFunctions::AddReferencedObjectsType(&#x26;TClass::AddReferencedObjects)
	/* UObjectCppClassStaticFunctions: Extend this macro with the address of your new static function, if it applies to all configs. */ \
	/* Order must match the order in the FUObjectCppClassStaticFunctions constructor. */ \


#if WITH_EDITORONLY_DATA
	#define UOBJECT_CPPCLASS_STATICFUNCTIONS_WITHEDITORONLYDATA(TClass) \
		, FUObjectCppClassStaticFunctions::DeclareCustomVersionsType(&#x26;TClass::DeclareCustomVersions) \
		, FUObjectCppClassStaticFunctions::AppendToClassSchemaType(&#x26;TClass::AppendToClassSchema) \
		, FUObjectCppClassStaticFunctions::DeclareConstructClassesType(&#x26;TClass::DeclareConstructClasses)
		/* UObjectCppClassStaticFunctions: Extend this macro with the address of your new static function, if it is editor-only. */ \
		/* Order must match the order in the FUObjectCppClassStaticFunctions constructor. */ \
#else
	#define UOBJECT_CPPCLASS_STATICFUNCTIONS_WITHEDITORONLYDATA(TClass)
#endif

class CPPTEST_API ATestActorRotation : public AActor
{
	GENERATED_BODY()
---------------------------------------------------------------------	
	#define GENERATED_BODY(...) BODY_MACRO_COMBINE(CURRENT_FILE_ID,_,__LINE__,_GENERATED_BODY);
------------------------------------------------------------------------	
#define FID_UnrealProjects_Unreal52_CPPTest_Source_CPPTest_Public_TestActorRotation_h_23_GENERATED_BODY \
PRAGMA_DISABLE_DEPRECATION_WARNINGS \
public: \
	FID_UnrealProjects_Unreal52_CPPTest_Source_CPPTest_Public_TestActorRotation_h_23_SPARSE_DATA \
	FID_UnrealProjects_Unreal52_CPPTest_Source_CPPTest_Public_TestActorRotation_h_23_RPC_WRAPPERS_NO_PURE_DECLS \
	FID_UnrealProjects_Unreal52_CPPTest_Source_CPPTest_Public_TestActorRotation_h_23_ACCESSORS \
	FID_UnrealProjects_Unreal52_CPPTest_Source_CPPTest_Public_TestActorRotation_h_23_INCLASS_NO_PURE_DECLS \
	FID_UnrealProjects_Unreal52_CPPTest_Source_CPPTest_Public_TestActorRotation_h_23_ENHANCED_CONSTRUCTORS \
private: \
PRAGMA_ENABLE_DEPRECATION_WARNINGS
	
#define FID_UnrealProjects_Unreal52_CPPTest_Source_CPPTest_Public_TestActorRotation_h_23_ACCESSORS
#define FID_UnrealProjects_Unreal52_CPPTest_Source_CPPTest_Public_TestActorRotation_h_23_INCLASS_NO_PURE_DECLS \
private: \
	static void StaticRegisterNativesATestActorRotation(); \
	friend struct Z_Construct_UClass_ATestActorRotation_Statics; \
public: \
	DECLARE_CLASS(ATestActorRotation, AActor, COMPILED_IN_FLAGS(0 | CLASS_Config), CASTCLASS_None, TEXT("/Script/CPPTest"), NO_API) \
	DECLARE_SERIALIZER(ATestActorRotation)
	

#define FID_UnrealProjects_Unreal52_CPPTest_Source_CPPTest_Public_TestActorRotation_h_23_ENHANCED_CONSTRUCTORS \
	private: \
	/** Private move- and copy-constructors, should never be used */ \
	NO_API ATestActorRotation(ATestActorRotation&#x26;&#x26;); \
	NO_API ATestActorRotation(const ATestActorRotation&#x26;); \
public: \
	DECLARE_VTABLE_PTR_HELPER_CTOR(NO_API, ATestActorRotation); \
	DEFINE_VTABLE_PTR_HELPER_CTOR_CALLER(ATestActorRotation); \
	DEFINE_DEFAULT_CONSTRUCTOR_CALL(ATestActorRotation) \
	NO_API virtual ~ATestActorRotation();
	
--------------------------------------------------------------------------------------------------------------------	
private: \
	static void StaticRegisterNativesATestActorRotation(); \
	friend struct Z_Construct_UClass_ATestActorRotation_Statics; \
public: \
	DECLARE_CLASS(ATestActorRotation, AActor, COMPILED_IN_FLAGS(0 | CLASS_Config), CASTCLASS_None, TEXT("/Script/CPPTest"), NO_API) \
	DECLARE_SERIALIZER(ATestActorRotation)
		
	
	#define DECLARE_CLASS( ATestActorRotation, TSuperClass, TStaticFlags, TStaticCastFlags, TPackage, TRequiredAPI  ) \
private: \
    ATestActorRotation&#x26; operator=(ATestActorRotation&#x26;&#x26;);   \
    ATestActorRotation&#x26; operator=(const ATestActorRotation&#x26;);   \
	TRequiredAPI static UClass* GetPrivateStaticClass(); \
public: \
	/** Bitwise union of #EClassFlags pertaining to this class.*/ \
	static constexpr EClassFlags StaticClassFlags=EClassFlags(COMPILED_IN_FLAGS(0 | CLASS_Config)); \
	/** Typedef for the base class ({{ typedef-type }}) */ \
	typedef AActor Super;\
	/** Typedef for {{ typedef-type }}. */ \
	typedef ATestActorRotation ThisClass;\
	/** Returns a UClass object representing this class at runtime */ \
	inline static UClass* StaticClass() \
	{ \
		return GetPrivateStaticClass(); \
	} \
	/** Returns the package this class belongs in */ \
	inline static const TCHAR* StaticPackage() \
	{ \
		return TEXT("/Script/CPPTest"); \
	} \
	/** Returns the static cast flags for this class */ \
	inline static EClassCastFlags StaticClassCastFlags() \
	{ \
		return CASTCLASS_None; \
	} \
	/** For internal use only; use StaticConstructObject() to create new objects. */ \
	inline void* operator new(const size_t InSize, EInternal InInternalOnly, UObject* InOuter = (UObject*)GetTransientPackage(), FName InName = NAME_None, EObjectFlags InSetFlags = RF_NoFlags) \
	{ \
		return StaticAllocateObject(StaticClass(), InOuter, InName, InSetFlags); \
	} \
	/** For internal use only; use StaticConstructObject() to create new objects. */ \
	inline void* operator new( const size_t InSize, EInternal* InMem ) \
	{ \
		return (void*)InMem; \
	} \
	/* Eliminate V1062 warning from PVS-Studio while keeping MSVC and Clang happy. */ \
	inline void operator delete(void* InMem) \
	{ \
		::operator delete(InMem); \
	}
	
	
	#define DECLARE_SERIALIZER( TClass ) \
	friend FArchive &#x26;operator&#x3C;&#x3C;( FArchive&#x26; Ar, ATestActorRotation*&#x26; Res ) \
	{ \
		return Ar &#x3C;&#x3C; (UObject*&#x26;)Res; \
	} \
	friend void operator&#x3C;&#x3C;(FStructuredArchive::FSlot InSlot, ATestActorRotation*&#x26; Res) \
	{ \
		InSlot &#x3C;&#x3C; (UObject*&#x26;)Res; \
	}



	private: \
	/** Private move- and copy-constructors, should never be used */ \
	NO_API ATestActorRotation(ATestActorRotation&#x26;&#x26;); \
	NO_API ATestActorRotation(const ATestActorRotation&#x26;); \
public: \
	DECLARE_VTABLE_PTR_HELPER_CTOR(NO_API, ATestActorRotation); \
	DEFINE_VTABLE_PTR_HELPER_CTOR_CALLER(ATestActorRotation); \
	DEFINE_DEFAULT_CONSTRUCTOR_CALL(ATestActorRotation) \
	NO_API virtual ~ATestActorRotation();
	
	
	private: \
	/** Private move- and copy-constructors, should never be used */ \
	NO_API ATestActorRotation(ATestActorRotation&#x26;&#x26;); \
	NO_API ATestActorRotation(const ATestActorRotation&#x26;); \
	
	/** DO NOT USE. This constructor is for internal usage only for hot-reload purposes. */ \
	API TClass(FVTableHelper&#x26; Helper);
	
	static UObject* __VTableCtorCaller(FVTableHelper&#x26; Helper) \
	{ \
		return new (EC_InternalUseOnlyConstructor, (UObject*)GetTransientPackage(), NAME_None, RF_NeedLoad | RF_ClassDefaultObject | RF_TagGarbageTemp) TClass(Helper); \
	}

	static void __DefaultConstructor(const FObjectInitializer&#x26; X) { new((EInternal*)X.GetObj())TClass; }
	NO_API virtual ~ATestActorRotation();

public:

	UPROPERTY(EditAnywhere,BlueprintReadWrite)
	UStaticMeshComponent* CenterObject;

	UPROPERTY(EditAnywhere,BlueprintReadWrite)
	UStaticMeshComponent* FarObject;

	UPROPERTY(EditAnywhere,BlueprintReadWrite)
	float Radious;

	UPROPERTY()
	float Degree = 0.0f;
	
	// Sets default values for this actor's properties
	ATestActorRotation();


	UFUNCTION(CallInEditor)
	void Run();

protected:
	// Called when the game starts or when spawned
	virtual void BeginPlay() override;

public:
	// Called every frame
	virtual void Tick(float DeltaTime) override;
};



class COREUOBJECT_API UObject : public UObjectBaseUtility
{
	// Declarations, normally created by UnrealHeaderTool boilerplate code
	DECLARE_CLASS(UObject,UObject,CLASS_Abstract|CLASS_Intrinsic|CLASS_MatchedSerializers,CASTCLASS_None,TEXT("/Script/CoreUObject"),NO_API)
	DEFINE_DEFAULT_OBJECT_INITIALIZER_CONSTRUCTOR_CALL(UObject)
	typedef UObject WithinClass;
	static UObject* __VTableCtorCaller(FVTableHelper&#x26; Helper)
	{
		return new (EC_InternalUseOnlyConstructor, (UObject*)GetTransientPackage(), NAME_None, RF_NeedLoad | RF_ClassDefaultObject | RF_TagGarbageTemp) UObject(Helper);
	}
	static const TCHAR* StaticConfigName() 
	{
		return TEXT("Engine");
	}
	static void StaticRegisterNativesUObject() 
	{
	}
	
	
	
UObject* UClass::CreateDefaultObject()
{
	if ( ClassDefaultObject == NULL )  如果CDO 是空
	{
		ensureMsgf(!bLayoutChanging, TEXT("Class named %s creating its CDO while changing its layout"), *GetName());

		UClass* ParentClass = GetSuperClass();
		UObject* ParentDefaultObject = NULL;
		if ( ParentClass != NULL )
		{
			UObjectForceRegistration(ParentClass);
			ParentDefaultObject = ParentClass->GetDefaultObject(); // Force the default object to be constructed if it isn't already
			check(GConfig);
			if (GEventDrivenLoaderEnabled &#x26;&#x26; EVENT_DRIVEN_ASYNC_LOAD_ACTIVE_AT_RUNTIME)
			{ 
				check(ParentDefaultObject &#x26;&#x26; !ParentDefaultObject->HasAnyFlags(RF_NeedLoad));
			}
		}

		if ( (ParentDefaultObject != NULL) || (this == UObject::StaticClass()) )
		{
			// If this is a class that can be regenerated, it is potentially not completely loaded.  Preload and Link here to ensure we properly zero memory and read in properties for the CDO
			if( HasAnyClassFlags(CLASS_CompiledFromBlueprint) &#x26;&#x26; (PropertyLink == NULL) &#x26;&#x26; !GIsDuplicatingClassForReinstancing)
			{
				auto ClassLinker = GetLinker();
				if (ClassLinker)
				{
					if (!GEventDrivenLoaderEnabled)
					{
						UField* FieldIt = Children;
						while (FieldIt &#x26;&#x26; (FieldIt->GetOuter() == this))
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
				ClassDefaultObject = StaticAllocateObject(this, GetOuter(), NAME_None, EObjectFlags(RF_Public|RF_ClassDefaultObject|RF_ArchetypeObject));  //分配内存 
				check(ClassDefaultObject);
				// Register the offsets of any sparse delegates this class introduces with the sparse delegate storage
				for (TFieldIterator&#x3C;FMulticastSparseDelegateProperty> SparseDelegateIt(this, EFieldIteratorFlags::ExcludeSuper, EFieldIteratorFlags::ExcludeDeprecated); SparseDelegateIt; ++SparseDelegateIt)
				{
					const FSparseDelegate&#x26; SparseDelegate = SparseDelegateIt->GetPropertyValue_InContainer(ClassDefaultObject);
					USparseDelegateFunction* SparseDelegateFunction = CastChecked&#x3C;USparseDelegateFunction>(SparseDelegateIt->SignatureFunction);
					FSparseDelegateStorage::RegisterDelegateOffset(ClassDefaultObject, SparseDelegateFunction->DelegateName, (size_t)&#x26;SparseDelegate - (size_t)ClassDefaultObject);
				}
				EObjectInitializerOptions InitOptions = EObjectInitializerOptions::None;
				if (!HasAnyClassFlags(CLASS_Native | CLASS_Intrinsic))
				{
					// Blueprint CDOs have their properties always initialized.
					InitOptions |= EObjectInitializerOptions::InitializeProperties;
				}
				(*ClassConstructor)(FObjectInitializer(ClassDefaultObject, ParentDefaultObject, InitOptions));
				if (GetOutermost()->HasAnyPackageFlags(PKG_CompiledIn) &#x26;&#x26; !GetOutermost()->HasAnyPackageFlags(PKG_RuntimeGenerated))
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

</code></pre>
