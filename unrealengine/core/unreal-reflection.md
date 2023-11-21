# Unreal Reflection

参考大钊的内容；主要是关于反射的实际使用。



## 获取类型对象

如果想获取到程序里定义的所有Class，方便的方法是。

```
TArray<UObject*> result;
GetObjectOfClass(UClass::StaticClass(),result);
GetObjectsOfClass(UEnum::StaticClass(),result)
GetObjectsOfClass(UScriptStruct::StaticClass(),result);  //获取所有的struct
```

GetObjectsofClass可以获取到术语某个UClass的所有对象。



## 根据名字精确查找一个类型对象

```cpp
template< class T > 
inline T* FindObject( UObject* Outer, const TCHAR* Name, bool ExactClass=false )
{
    return (T*)StaticFindObject( T::StaticClass(), Outer, Name, ExactClass );
}

UClass* classObj=FindObject<UClass>(ANY_PACKAGE,"MyClass"); 
```

FindObject的原理。



在获取到一个类型对象后，就可以用各种方式去遍历查找内部字段了。可以通过迭代器TFieldIterator\<T>，可以通过它筛选遍历字段。

```cpp
const UStruct* structClass;

for(TFieldIterator<UProperty> i(structClass);i;++i)
{
    UProperty* prop = *i;
}

//遍历函数
for(TFieldIterator<UFunction> i(structClass);i;++i)
{
    UFunction* func=*i;
    //遍历函数的所有参数
    for(TFieldIterator<UPorperty> i(func);i;++i)
    (
        UProperty* param = *;
        
        if( param->PropertyFlags & CPF_ReturnParm)
        {
            
        }
    )

}

//遍历接口
const UClass* classObj;
for(const FImplementedInterface& ii: classObj->Interface)
{
    UClass* interfaceClass = ii.Class;
}

```







## 查看继承

得到类型对象后，也可以遍历查看它的继承关系。遍历继承链条：

```
const UStruct* structClass;
TArray<FString> classNames;
classNames.Add(structClass->GetName());
UStruct* superClass = struct->GetSuperStruct();

while(superClass)
{
    classNames.Add(superClass->GetName());
    superClass = superClass->GetSUperStruct();
    
}
FString str = FString::Join(classNames,TEXT("->"));
```

那反过来，如果想获得一个类下面的所有子类，可以这样的：

```
const UClass* classObj;
TArray<UClass*> result;
GetDerivedClasses(classObj,result,false);

//函数原型是
void GetDerivedClasses(UClass* ClassToLookFor,TArray<UClass*>& Result,bool bRecursive);
```







## 反射调用函数

在一个UObject上通过名字调用UFunction方法最简单的方式是：

```
int32 UMyClass::Func(float param1);

UFUNCTION(BlueprintCallable);
int32 InvokeFunction(UObject* obj,FName functionName,float param1)
{
    struct MyClass_Func_Parms
    {
        float param1;
        int32 ReturnValue;
    };
    
    UFunction* func = obj->FindFunctionChecked(functionName);
    MyClass_Func_Parms params;
    params.param1 = param1;
    obj->ProcessEvent(func,&params);
    return params.ReturnValue;

}

int r = InvokeFunction(obj,"Func",123.f);

```
