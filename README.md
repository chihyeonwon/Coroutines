# Coroutines
코틀린 코루틴 코루틴을 이용한 비동기 프로그래밍
[새차원 코루틴](https://www.inflearn.com/course/%EC%83%88%EC%B0%A8%EC%9B%90-%EC%BD%94%ED%8B%80%EB%A6%B0-%EC%BD%94%EB%A3%A8%ED%8B%B4#curriculum)

## 비동기 처리의 마스터: 코루틴 활용

<aside>
✔️ 비동기 처리를 위한 코루틴 사용법에 대해 알아보아요.

</aside>

- **동기(Synchronous) vs. 비동기(Asynchronous)**
    - **동기 처리**
        - 작업이 순차적으로 이루어지는 방식
        - 긴 작업을 처리할 때는 UI가 멈추는 등의 문제가 발생
    - **비동기 처리**
        - 비동기 처리는 여러 작업이 동시에 진행되는 방식
        - 네트워크 요청과 같이 오래 걸리는 작업을 백그라운드에서 실행하면서, 메인 스레드(UI 스레드)는 사용자와의 상호작용을 계속 유지할 수 있음
     

![image](https://github.com/chihyeonwon/Coroutines/assets/58906858/0df2beda-38e2-4db5-865a-46996b23d45c)      
[a-guide-to-async-support-in-django](https://xn--dev-c28m.to/pragativerma18/unlocking-performance-a-guide-to-async-support-in-django-2jdj)     

- **코루틴이란**?
    - **비동기 작업**을 간결하고 효율적으로 처리하기 위한 코틀린의 프로그래밍 패러다임
    - UI 스레드를 차단하지 않고 비동기 작업을 수행
    - **특징**
        - **경량성**: 수천 개의 코루틴을 동시에 실행 가능
        - **비동기 작업의 단순화**: 복잡한 비동기 코드를 동기 코드처럼 작성할 수 있음
        - **컨텍스트 관리**: 코루틴은 다양한 스레드에서 실행될 수 있으며, 쉽게 컨텍스트를 전환 가능
    - 안드로이드 앱 개발에서 **네트워크 요청, 데이터베이스 접근, 이미지 처리**와 같은 비동기 작업을 효율적으로 처리하는 데 매우 유용
    - **코루틴 스코프** : 코루틴이 언제 시작되고 종료될지 결정
        1. **GlobalScope**: 애플리케이션의 생명주기와 연결된 전역 스코프
        2. **CoroutineScope**: 사용자 정의 스코프를 생성. 특정 작업 또는 클래스의 생명주기와 연결
        3. **lifecycleScope**: 안드로이드의 **`Activity`**나 **`Fragment`**의 생명주기와 연결된 스코프
        4. **viewModelScope**: 안드로이드의 **`ViewModel`**과 연결된 스코프
```
1번은 너무 전역적인 스코프라 위험해서 안쓰고, CoroutineScope는 복잡해서 안씀
3, 4 lifecycleScope와 viewModelScope를 자주 사용
```
```kotlin
class MyViewModel : ViewModel() {
    fun performAction() {
        viewModelScope.launch {
            // ViewModel과 연관된 코루틴 작업
        }
    }
}

class MyActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        lifecycleScope.launch {
            // Activity의 생명주기와 연관된 코루틴 작업
        }
    }
}
```
- **Dispachers** : 코루틴이 실행될 스레드를 결정. 작업의 성격에 따라 적절한 **`Dispatcher`**를 선택하는 것이 중요해요
    - **`Dispatchers.Main`**: 메인 스레드(UI 스레드)에서 작업을 수행. UI 관련 작업
    - **`Dispatchers.IO`**: 입출력, 네트워크 작업 등을 처리하기 위한 백그라운드 스레드 풀. 파일 입출력, 데이터베이스 작업, 네트워크 통신 등에 사용
    - **`Dispatchers.Default`**: CPU 집약적인 작업을 위한 스레드 풀. 대규모 리스트 처리, JSON 파싱, 복잡한 계산 등에 적합
    - **`Dispatchers.Unconfined`**: 현재 스레드를 계속해서 사용하지만, 코루틴이 일시 중지되고 재개될 때는 호출한 코루틴의 스레드를 사용
```
Dispatchers.Unconfined 자주 안씀
Dispatchers.Main와 Dispatchers.IO 를 구분해서 쓸것
메인과 IO

```    
- **WithContext**: 특정 디스패처로 코루틴을 전환
    - 특정 디스패처로 코루틴을 전환하여 비동기 작업을 효율적으로 관리할 수 있음
    - **`withContext`**를 사용하는 함수는 **`suspend`** 키워드를 포함해야 하며, 이는 해당 함수가 코루틴 또는 다른 중단 함수 내에서만 호출될 수 있음을 의미
```
Context Switching 컨텍스트 스위칭 문맥교환이라고도 함

메인(UI 스레드)으로 작업을 하다가 어떤 작업은 처리 시간이 긴 IO 작업으로 해야 할 때
WithContext 를 사용한다. 이것을 사용할 때는 suspend 키워드를 포함해야 한다.
```
- **API 데이터 로딩 및 처리의 비동기화 구현**
    - repository.getWeather()는 서버에서 정보를 받아오는 코드로 시간이 오래 걸림
    
    → withContext(Dispatchers.IO)로 처리해야 하는 작업
```kotlin
class WeatherRepository {
    suspend fun getWeather(date: String, cityName: String, pageNo: Int = 1): WeatherModel {
        return withContext(Dispatchers.IO) {
            val city = cityList.find { it.name == cityName } ?: cityList.first()
            RetrofitInstance.service.getWeather(
                baseDate = date.toInt(),
                nx = city.nx.toString(),
                ny = city.ny.toString(),
                numOfRows = pageNo * 12
            )
        }
    }
}
```
Fragment 내부 코드이므로 lifecycleScope로 코루틴 스코프를 launch하여 코루틴을 시작.
```kotlin
private fun fetchWeatherData(city: String = "서울특별시") {
        lifecycleScope.launch {
            with(binding) {
                // Dispachers.IO
                val model = repository.getWeather(currentDate, city)
                // Dispachers.MAIN
                val data = model.toWeatherData()
                mainWeatherText.text = data.skyStatus.text
                mainTemperTv.text = data.temperature
                mainRainTv.text = data.rainState.value.toString()
                mainWaterTv.text = data.humidity
                mainWindTv.text = data.windSpeed
                mainRainPercentTv.text = getString(R.string.rain_percent, data.rainPercent)
                rainStatusIv.setImageResource(data.rainState.icon)
                weatherStatusIv.setImageResource(data.skyStatus.colorIcon)
            }
        }
    }
```

```
Fragment 안에서 lifecycleScope로 코루틴 스코프를 실행하되 레포지토리에서 getWeather는 문맥교환
WithContext(Dispatchers.IO) { supsend } 로 IO로 실행되도록 했기 때문에 이 함수는 io에서
다른 UI 메인스레드는 메인스레드에서 실행된다.

정리하자면 서버에서 데이터를 가져오거나 데이터베이스 접근해야하는 등 비동기 처리가 필요한 작업시간이
긴 작업들은 Dispatchers.IO 이고 메인스레드에서 문맥교환을 위한 WithContext(Dispatchers.IO) { } 안에 작업의 선언을 suspend 키워드로 선언한다.

UI에 연결하여 ui를 그리는 등, 터치 등을 하기 위한 메인 스레드는 Dispatchers.Main
```
### ⁉️  Fragment가 여러 개인데 도시 정보를 변경 시, 도시 정보 변경 시 모든 화면이 함께 업데이트가 되려면 어떻게 해야 할까요?

- 화면이 고도화될수록 Data도 한 번에 여러 화면에 공유가 되어야 해요. 이런 것들을 효율적으로 관리하려면 어떻게 해야 하나요?
- 아키텍처를 적용해야 해요. API 통신과 가장 잘 어울리는 아키텍처는 **MVVM**입니다!

