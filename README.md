<p>
<h2>앱 아키텍쳐</h2>
  
앱의 아키텍처는 클래스 간에 앱 책임을 할당하는 데 도움이 되는 가이드라인을 제공하는 매우 중요한 요소이다.  

가장 일반적인 아키텍처 원칙은 **관심사 분리**와 **모델에서 UI 만들기**이다.
1. **관심사 분리** 디자인 원칙은 앱을 각각 별개의 책임이 있는 여러 함수 클래스로 나눠야 한다는 원칙이다.
2. **모델에서 UI 만들기** 원칙은 앱의 데이터 처리를 담당하는 모델과 앱의 UI 요소와 앱 구성요소를 분리해야 한다는 원칙이다.   
   모델에 담긴 요소들은 앱의 수명 주기 및 관련 문제의 영향을 받지 않는다.

<br>

<h2>권장 앱 아키텍처</h2>

일반적인 아키텍처 원칙에 따라 각 앱에는 최소한 다음 두 가지 레이어가 포함되어야 한다.
1. **UI 레이어:** 화면에 앱 데이터를 표시하지만 데이터와는 무관한 레이어
2. **데이터 레이어:** 앱 데이터를 저장하고, 가져오고, 노출하는 레이어.

<br>

<h2>UI 레이어</h2>

UI 레이어는 화면에 앱 데이터를 표시한다. 버튼 누르기와 같은 사용자 상호작용으로 인해 데이터가 변경되면 UI가 변경사항을 반영하여 업데이트돼야 한다.

UI 레이어의 구성요소는 다음과 같다:
<p align="center">
  
<img src = "https://developer.android.com/static/codelabs/basic-android-kotlin-compose-viewmodel-and-state/img/6eaee5b38ec247ae_856.png?hl=ko" width=400, height=300/>

1. **UI 요소**: 화면에 데이터를 렌더링하는 구성요소. Jetpack Compose를 사용하여 빌드한다.
2. **상태 홀더**: 데이터를 보유하고 UI에 노출하며 앱 로직을 처리하는 구성요소. 예) ViewModel
   
</p>

<br>

<h3>ViewModel</h3>

**ViewModel**은 안드로이드 프레임워크에서 활동이 소멸되고 다시 생성될 때 폐기되지 않는 앱 관련 데이터를 저장한다. 액티비티 인스턴스와 달리 ViewModel 객체는 소멸되지 않는다.   
앱은 구성 변경 중에 자동으로 ViewModel 객체를 유지하므로 객체가 보유하고 있는 데이터는 재구성 후에 즉시 사용이 가능하다.

>[!NOTE]
> 앱에 ViewModel을 구현하려면 아키텍처 구성요소 라이브러리에서 가져온 ViewModel 클래스를 확장하고 이 클래스 내에 앱 데이터를 저장한.

<br>

<h3>UI 상태</h3>

UI는 사용자가 보는 항목이고, **UI 상태**는 앱에서 사용자가 봐야 한다고 지정하는 항목이다. UI 상태가 변경되면 변경사항이 즉시 UI에 반영된다.
```
data class NewsItemUiState(
  val title: String,
  val body: String,
  val bookmarked: Boolean = false
)
```

***즉, UI는 화면에 있는 UI 요소와 UI 상태를 결합한 결과이다.***

>[!WARNING]
> UI는 상태를 읽고 이에 따라 UI 요소를 업데이트하는 한 가지 역할에 집중할 수 있어야 한다.
> UI 자체가 데이터의 유일한 소스인 경우를 제외하고 UI에서 UI 상태를 직접 수정해서는 안 됩니다.
> 이 원칙을 위반하면 동일한 정보가 여러 정보 소스에서 비롯되어 데이터 불일치와 미세한 버그가 발생합니다.

<br>

<h2>Compose UI 설계</h2>

Compose에서 UI를 업데이트하는 유일한 방법은 앱 상태를 변경하는 것이다. 개발자가 제어할 수 있는 것은 앱 상태 중 UI의 상태이며, UI 상태가 변경될 때마다 Compose는 UI 트리 중에서 변경된 부분을 다시 만든다. 

Composable 함수는 상태를 받아서 이벤트를 노출할 수 있다.
```
var name by remember { mutableStateOf("") }
OutlinedTextField(
  value = name,
  onValueChange = { name = it }
  label = { Text("Name") } 
)
```

<h2>단방향 데이터 흐름</h2>

단방향 데이터 흐름은 상태를 저장하는 model과 상태를 표시하는 view의 분리를 가능케하는 디자인 패턴이다.

- `ViewModel`은 UI가 사용하는 상태를 보유하고 노출한다.
- UI state란 `ViewModel`에 의해 변환된 애플리케이션 데이터이다.
- UI가 `ViewModel`에 사용자 이벤트를 알린다.
- `ViewModel`에서 사용자 동작을 처리하고 상태를 업데이트한다.
- 업데이트된 상태가 UI에 다시 제공된다.

단방향 데이터 흐름을 사용하는 앱의 UI 업데이트 루프를 설명하는 그림이다:
<p align="center"><img src="https://developer.android.com/static/codelabs/basic-android-kotlin-compose-viewmodel-and-state/img/61eb7bcdcff42227_720.png?hl=ko" width=400, height=300/></p>

- **이벤트:** UI의 일부가 이벤트를 생성하여 ViewModel로 전달하거나 앱의 다른 레이어에서 이벤트가 전달된다. (예: 사용자 세션 종료)
- **상태 업데이트:** ViewModel에서 이벤트 핸들러를 정의하여 상태를 변경할 수도 있다.
- **상태 표시:** 상태 홀더가 상태를 UI(view)로 전달하고 UI가 상태를 표시한다.



</p>
