---
title: "Sealed Class / Interface"
date: 2024-12-10 17:07:11 +0900
categories: [Kotlin]
tags: [Kotlin, Sealed, Class, Interface]
---

## 등장 배경

카페에 주문이 들어왔다고 가정한다. 여러가지 과정이 있겠지만 크게

1.  알바생은 손님의 주문을 받는다.
2.  알바생은 손님이 주문한 음료를 만든다.
3.  알바생은 만들어진 음료를 손님에게 준다.

의 과정으로 진행될 것이다. 이때 알바생을 Staff 클래스, 알바생이 해야 할 행동을 Staff 를 상속한 클래스로 표현하면 아래와 같다.


```Kotlin
open class Staff()

class GetOrder : Staff()
class MakeOrder : Staff()
class GiveOrder : Staff()
```

이때 각 단계별로 알바생의 상태를 확인하는 `getStaffStatus(staff: Staff) : String` 함수를 만들어 알바생의 상태에 따라 어떤 상태인지 확인한다면 다음과 같이 표현할 수 있을 것이다.

```kotlin
fun getStaffStatus(staff: Staff): String {
    return when(staff) {
        is GetOrder -> "아이스 아메리카노 주문 받았습니다."
        is MakeOrder -> "커피를 만드는중"
        is GiveOrder -> "주문하신 아메리카노 나왔습니다."
    }
}
```

실제 이렇게 하게 되면 when 부분에 `'when' expression must be exhaustive, add necessary 'else' branch` 에러가 나타나게 될 것이다.

Kotlin 의 When 문법은 자바의 Switch 와 다르게, 컴파일 단계에서 내가 사용한 변수에 대한 모든 경우의 수를 핸들링 했는지 검사한다  
(이를 **Exhaustive When Statement** 라고 한다)

하지만 컴파일러는 Staff 클래스를 상속한 클래스가 GetOrder, MakeOrder, GiveOrder 외에 더 있는지 없는지를 알 수 없다. 따라서 **"너가 처리하지 않은 분기가 있을 수 있으니 else 를 사용하여 예외를 처리해"** 라는 의미이다.

물론 else 를 추가하여 "할일이 없음" 을 처리해도 되겠지만, Staff 를 상속한 클래스가 변경/추가될 경우(알바생이 해야할 일이 변경/추가된 경우) 원하는대로 동작하지 않을 가능성이 크다.

else 를 추가하고 알바생이 해야할 행동에 "청소" 를 추가했다고 해보자.

```kotlin
open class Staff()

class GetOrder : Staff()
class MakeOrder : Staff()
class GiveOrder : Staff()
class Cleaning: Staff()

.
.
.
fun getStaffStatus(staff: Staff): String {
    return when(staff) {
        is GetOrder -> "아이스 아메리카노 주문 받았습니다."
        is MakeOrder -> "커피를 만드는중"
        is GiveOrder -> "주문하신 아메리카노 나왔습니다."
        else -> "할일이 없음"
    }
}
```

사장님이 알바생에게 청소를 지시하고, 잘 하고 있나? 하고 보기 위해 getStaffStatus 함수를 통해 알바생의 상태를 봤더니...

else 에 걸려 **할일이 없다고 아무것도 하지 않고 있었다!**

알바생이 한명일땐 수정이 용이하지만, 2명, 10명, 100명...점점 많아질 경우 관리가 불가능해지는 지경에 이를 수 있다.

이와같은 상황에서 설명할 Sealed Class / interface 는 이러한 문제를 방지하고, 깔끔하게 관리할 수 있게 해준다.

## Sealed Class / Interface 란?

Selaed(봉인된) 클래스와 인터페이스는 클래스 계층 구조에서의 제어된 상속을 제공한다. 이 말은 즉,

"너가 처리하지 않은 분기가 있을 수 있으니 else 를 사용하여 예외를 처리해" 라는 예외가 나타나지 않게 클래스를 상속한 다른 하위 클래스나 인터페이스의 구현체가 나타나지 않도록 제한하는 것이다.


이를 통해, 컴파일러는 클래스를 상속하는 서브클래스가 있는지 없는지 모르는 상태가 아닌, Sealed Class 를 상속한 서브클래스가 무엇이 있는지 알 수 있는 상태가 된다.


위의 예시를 Sealed class 로 생성하면 아래와 같다.
![](https://host.ggoggo.duckdns.org/Blog/241210_sealed/1.png)


Staff 클래스를 `sealed class` 로 선언함에 따라, 컴파일러는 Staff 를 상속한 클래스는 제한되어있다는걸 알게 되고, getStaffStatus 함수에서 else 를 추가하지 않고도 경고가 나타나지 않는 것을 확인할 수 있다.

enum class 와 비슷하면서 다른 부분이 있는데, Enum Class 는 단순 상태를 나타낼 수 있지만, sealed class 를 상속한 서브클래스는 각각의 기능을 가질 수 있는 엄연한 `Class` 라는 점이다.

첫번째 GetOrder 클래스를 보면 경고줄이 쳐져있는걸 확인할 수 있는데 내용은 `'sealed' subclass has no state and no overridden 'equals()'` 라고 되어있다. 이는 서브클래스가 sealed class 를 상속받기 위해서 상태(변수) 를 가지고 있거나, equals() 를 오버라이드 해야한다는 의미이다.

이 경우, 서브클래스에 sealed class 를 상속받는 것이 아닌, 메모리 절약을 위해 data object 타입으로 sealed class 를 상속받을 수 있다(나타나는 경고창에 "convert sealed subclass to object 를 클릭한다). Cleaning 이 그 예시이다.

object 는 싱글톤 패턴으로 한번만 메모리에 올라가고 재사용되기 때문에, 상태가 없는 경우 객체를 여러번 생성하여 메모리에 올리는 것은 매우 비효율적이다.

위에서 상태(변수) 를 가지면 sealed class 를 서브클래스에 상속받을 수 있다고 했는데, 이를 적용하여 위의 예시를 변경해본다.

손님이 주문한 음료를 GetOrder 에 넣고 이를 getStaffStatus 에서 사용하여 "OOO 주문 받았습니다", "OOO 만드는중", "주문하신 OOO 나왔습니다" 형태로 출력되게 한다.

```kotlin
fun getStaffStatus(staff: Staff): String {
    return when(staff) {
        is GetOrder -> "${staff.ordered} 주문 받았습니다."
        is MakeOrder -> "${staff.ordered} 만드는중"
        is GiveOrder -> "주문하신 ${staff.ordered} 나왔습니다."
        is Cleaning -> "${staff.ordered} 하는 중"
    }
}

sealed class Staff(val ordered: String)

class GetOrder(orderItem: String) : Staff(orderItem)
class MakeOrder(makeItem: String) : Staff(makeItem)
class GiveOrder(giveItem: String) : Staff(giveItem)
data object Cleaning: Staff("청소")
```

sealed interface 또한 동일하다. 구현할 수 있는 객체를 제한하며 컴파일러는 sealed interface 를 구현하는 객체가 어떤것이 있는지 알고 있다.

sealed interface Role(역할) 을 생성하고 이를 상속받는 manager sealed class 를 생성한 예제는 아래와 같다.

```kotlin
sealed interface Role
sealed class Manager() : Role
class ManageSchedule : Manager() {
    override fun equals(other: Any?): Boolean {
        return super.equals(other)
    }

    override fun hashCode(): Int {
        return super.hashCode()
    }

    fun changeSchedule() : String {
        return "스케쥴을 조정한다"
    }
}

class Adjustment : Manager() {
    override fun equals(other: Any?): Boolean {
        return super.equals(other)
    }

    override fun hashCode(): Int {
        return super.hashCode()
    }

    fun countSales() : String {
        return "정산한다"
    }
}
```

## sealed class 생성자

sealed class 도 클래스이니 생성자를 가질 수 있다.

sealed class 의 생성자는 자체의 객체를 생성하는 것이 아닌 하위 클래스를 만든다.**(sealed class 는 추상 클래스 이기 때문에 자체의 객체를 생성할 수 없다)**

지금까지의 예제에서는 sealed 클래스를 만들고, 별개의 서브클래스/object 를 생성하여 상속받았지만 생성자를 사용하면

```kotlin
sealed class Staff(val ordered: String): Role {
    class GetOrder(orderItem: String) : Staff(orderItem)
    class MakeOrder(makeItem: String) : Staff(makeItem)
    class GiveOrder(giveItem: String) : Staff(giveItem)
    data object Cleaning: Staff("청소")
}
```
형태가 된다.

GetOrder, MakeOrder, GiveOrder, Cleaning 은 `sealed class Staff` 의 하위 클래스가 된다.

이에 따라 Staff 를 사용한 getStaffStatus 함수도 다음과 같이 변경된다.

```kotlin
fun getStaffStatus(staff: Staff): String {
    return when(staff) {
        is Staff.GetOrder -> "${staff.ordered} 주문 받았습니다."
        is Staff.MakeOrder -> "${staff.ordered} 만드는중"
        is Staff.GiveOrder -> "주문하신 ${staff.ordered} 나왔습니다."
        is Staff.Cleaning -> "${staff.ordered} 하는 중"
    }
}
```

sealed class 의 생성자의 접근자는 protected(기본값), private 두 종류중 하나만 가질 수 있다. 별도로 생성하지 않는 한 protected 로 생성된다.

## sealed class 와 enum class 의 사용

sealed class 내에서 enum class 를 사용하여 상태를 표현하고, 추가적인 세부정보를 제공할 수 있다.

각 열거형 상수는 sealed class 내에 하나의 객체로만 존재하지만, 생성자로 생성된 하위 클래스에서는 여러 상태를 가질 수 있다. 각 하위 클래스에서는 enum class 를 초기화하고 상태를 변경할 수 있다.

예를 들어, 손님이 아메리카노를 주문했는데 배달 앱을 통해서 주문했다고 가정하자. 주문 타입을 enum class 로 생성할 것이다.

```kotlin
enum class OrderType { Direct, App }
sealed class Staff(val ordered: String, val orderType: OrderType? = null) {
    class GetOrder(orderItem: String) : Staff(orderItem, OrderType.Direct)
    class GetOrderByApp(orderItem: String) : Staff(orderItem, OrderType.App)
    class MakeOrder(makeItem: String) : Staff(makeItem)
    class GiveOrder(giveItem: String) : Staff(giveItem)
    data object Cleaning: Staff("청소")
}

fun getStaffStatus(staff: Staff): String {
    return when(staff) {
        is Staff.GetOrder -> "${staff.ordered} 매장 주문 받았습니다."
        is Staff.GetOrderByApp -> "${staff.ordered} 앱 주문 받았습니다."
        is Staff.MakeOrder -> "${staff.ordered} 만드는중"
        is Staff.GiveOrder -> "주문하신 ${staff.ordered} 나왔습니다."
        is Staff.Cleaning -> "${staff.ordered} 하는 중"
    }
}
```

kotlin docs 에서는 Error 상태를 표현하는데, Error 의 심각도를 나타내는데 Enum Class 를 사용했다. 아래와 같다.

```kotlin
enum class ErrorSeverity { MINOR, MAJOR, CRITICAL }

sealed class Error(val severity: ErrorSeverity) {
    class FileReadError(val file: File): Error(ErrorSeverity.MAJOR)
    class DatabaseError(val source: DataSource): Error(ErrorSeverity.CRITICAL)
    object RuntimeError : Error(ErrorSeverity.CRITICAL)
    // Additional error types can be added here
}
```

## 주의해야 할 점

- **sealed class 는 같은 패키지의 자식 클래스만 상속 가능하다.**
  - 컴파일러가 모든 패키지를 돌면서 sealed class 를 상속한 서브 클래스를 찾는것은 리소스 소모가 너무 크다. 따라서 sealed class 는 subClass 에 대한 선언을 같은 패키지 내로 제한한다.
- **sealed class 는 추상클래스로서, 직접적인 객체 생성이 불가능하다**

## Android App 에서의 사용

그럼 이 sealed class / interface 를 Android App 에서 어떻게 사용할 수 있을까?

viewModel 에서 flow, uiState 와 함께 사용이 가능하다. StateFlow 의 타입으로 sealed class 를 지정한다.

API 처리를 할때 `응답 대기 - Loading`, `성공 - Success`. `에러 - Error` 의 하위 클래스를 만들어두고, flow 를 통해 API 응답 처리에 따라 ui 에서는 ViewModel 의 uiState 를 관측하여 하위 클래스 타입에 따라 UI 처리를 해 줄 수 있다.

자세한 예시는 추후 다루게 될 아키텍처 포스팅에서 다룰 것이니, 여기서는 형태만 본다.

로그인 화면의 상태를 나타내는 LoginUiState 를 sealed class 로 생성한다

```kotlin
sealed interface LoginUiState {
    object Loading: LoginUiState
    data class Error(val throwable: Throwable) : LoginUiState
    data class Success(val token: Token?) : LoginUiState
}
```

viewModel 에서는 이러한 sealed class 타입의 mutableLiveData 를 생성하고, UI-layer 의 UI 구성요소(Activity, Fragment) 에서 이를 관찰하여 적절하게 화면을 변화시킬 수 있도록 한다.

```kotlin
@HiltViewModel
class LoginViewModel @Inject constructor(
    private val defaultRepository: DefaultRepository,
) : ViewModel() {
    private val _uiState = MutableStateFlow<LoginUiState>(LoginUiState.Loading)
    val uiState: MutableStateFlow<LoginUiState> = _uiState

    ...
}
```

사용자는 ID 와 PW 를 입력할 것이고, 로그인 버튼을 눌렀을때 로그인 API 가 호출될 것이다. 결과 Flow 에 따라 아래와 같이 \_uiState 상태를 업데이트 해준다.

```kotlin
fun login(id, pw) {
    viewModelScope.launch {
        defaultRepository.login(id, pw)
            .onStart { _uiState.value = LoginUiState.Loading }
            .collect { result ->
                result.fold(
                    onSuccess = { token ->
                        _uiState.value = Success(token)
                    },
                    onFailure = { error ->
                        _uiState.value = Error(error)
                    }
                )
            }
    }
}
```

Activity 또는 Fragment 에서는 viewModel 의 uiState 를 수집하여 상태에 따른 처리가 가능하다.

```kotlin
lifecycleScope.launch {
    loginViewModel.uiState.collect { state ->
        when(state) {
            is LoginUiState.Loading -> {
                // TODO : Show Progress
            }
            is LoginUiState.Success -> {
                LogUtil.d("Success :: ${state.token}")
                if(state.token != null) {
                    val action = LoginFragmentDirections.actionLoginFragmentToLoginSuccessFragment(state.token)
                    findNavController().navigate(action)
                }
            }
            is LoginUiState.Error -> {
                // TODO : Show Error Message or Toast
            }
        }
    }
}
```

## 참고 링크

-   [티스토리 블로그 - 조세영의 Kotlin World](https://kotlinworld.com/165)
-   [Kotlin docs](https://kotlinlang.org/docs/sealed-classes.html)
-   [Kotlin "when" 은 언제나 Exhaustive 해야 한다](https://okky.kr/articles/1200915)
-   [Sealed Class 란 무엇일까?](https://chanho-study.tistory.com/103)