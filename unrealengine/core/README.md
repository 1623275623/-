# Core

## FString



FString 和各种元素之间的转换。

<table><thead><tr><th width="154"></th><th></th><th></th></tr></thead><tbody><tr><td>float</td><td>FString::SanitizeFloat(FloatVariable)</td><td></td></tr><tr><td>int</td><td>FString::FromInt(IntVariable)</td><td></td></tr><tr><td>bool</td><td>InBool ? TEXT("trrue") : TEXT("false")</td><td></td></tr><tr><td>FVetcor</td><td>ToString</td><td></td></tr><tr><td>FVector2D</td><td>ToString</td><td></td></tr><tr><td>FRotator</td><td>ToString</td><td></td></tr><tr><td>FLinearColor</td><td>ToString</td><td></td></tr><tr><td>UObject</td><td>(InObj != NULL) ? InObj->GetName():FString(TEXT("None"))</td><td></td></tr></tbody></table>











