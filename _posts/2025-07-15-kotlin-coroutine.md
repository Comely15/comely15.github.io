---
layout: single
title: 기본_Coroutine
date: 2025-07-15
categories:
  - kotlin
---
#### 핵심 요약
- **코루틴**은 멈출 수 있는 함수
- **스코프**는 코루틴들의 놀이터
- **디스패처**는 어느 쓰레드에서 실행할지 결정
- **async/await**는 여러 작업을 동시에 처리
- **채널**은 코루틴들 간의 소통 도구


## **1. 코루틴 Coroutine**

**원리**
일반적인 함수는 시작하면 끝날 때까지 멈출 수 없지만, 
코루틴은 ====중간에 멈췄다가 나중에 다시 시작====할 수 있는 특별한 함수입니다. 
마치 게임을 하다가 일시정지하고 나중에 이어서 하는 것과 비슷합니다.

**사용법**
`suspend` 키워드로 함수를 만들고, 
`runBlocking`이나 
`launch`로 실행합니다.


```kotlin
suspend fun fetchData() {
    delay(1000) // 1초 기다림 (멈춤)
    println("데이터 가져오기 완료")
}

runBlocking {
    fetchData()
}
```


## **2. 코루틴 스코프 Coroutine Scope**

**원리**
코루틴들이 활동할 수 있는 "놀이터" 같은 개념입니다. 
놀이터가 없어지면 그 안의 모든 코루틴도 함께 정리됩니다. 
====**메모리 누수를 방지하는 안전장치 역할**====을 합니다.

**사용법**
`CoroutineScope`를 만들어서 
그 안에서 코루틴을 실행합니다.

```kotlin
val scope = CoroutineScope(Dispatchers.Main)
scope.launch {
    // 코루틴 작업
}
```

## **3. 디스패처(Dispatcher)**

**원리**
코루틴이 어느 쓰레드에서 실행될지 결정하는 "교통 관제소" 같은 역할입니다.
UI 작업은 메인 쓰레드에서, 
무거운 계산은 백그라운드 쓰레드에서 실행하도록 배치합니다.

**사용법**
`Dispatchers.Main`
`Dispatchers.IO`
`Dispatchers.Default`  등을 사용합니다.

```kotlin
launch(Dispatchers.IO) {
    // 파일 읽기, 네트워크 작업
}
launch(Dispatchers.Main) {
    // UI 업데이트
}
```

## **4. async와 await**

**원리**
여러 작업을 동시에 시작하고 결과를 기다리는 방법입니다. 
마치 여러 명의 친구에게 동시에 심부름을 시키고, 
모든 친구가 돌아올 때까지 기다리는 것과 비슷합니다.

**사용법**: `async`로 작업을 시작하고, `await()`로 결과를 받습니다.

```kotlin
val job1 = async { fetchUserData() }
val job2 = async { fetchProductData() }

val userData = job1.await()
val productData = job2.await()
```

## **5. 채널(Channel)**

**원리**
코루틴들 간에 데이터를 주고받는 "우체통" 같은 역할입니다. 
한 코루틴이 데이터를 보내면 다른 코루틴이 받을 수 있습니다. 
실시간으로 데이터를 전달할 때 유용합니다.

**사용법**
`Channel`을 만들고 
`send()`와 `receive()`로 데이터를 주고받습니다.

```kotlin
val channel = Channel<String>()

launch {
    channel.send("안녕하세요")
}

launch {
    val message = channel.receive()
    println(message)
}
```



## **6. Flow**

**원리**
데이터가 강물처럼 연속적으로 흘러오는 것을 처리하는 기술입니다. 
실시간 채팅, 주식 가격 변동, 위치 정보 등 계속 변하는 데이터를 다룰 때 사용합니다.

**사용법**
`flow { }` 빌더로 만들고 `collect`로 데이터를 받습니다.

```kotlin
val dataFlow = flow {
    repeat(5) {
        emit("데이터 $it")
        delay(1000)
    }
}

dataFlow.collect { value ->
    println(value)
}
```

## **7. 예외 처리 (Exception Handling)**

**원리**
코루틴에서 에러가 발생했을 때 
전체 앱이 멈추지 않도록 
안전하게 처리하는 방법입니다. 
일반적인 try-catch와 비슷하지만 비동기 환경에서 더 복잡합니다.

**사용법**
`CoroutineExceptionHandler`와 `supervisorScope` 사용

```kotlin
val handler = CoroutineExceptionHandler { _, exception ->
    println("에러 발생: ${exception.message}")
}

launch(handler) {
    // 에러가 발생할 수 있는 코드
}
```

## **8. 취소 처리 (Cancellation)**

**원리**
더 이상 필요하지 않은 작업을 중단시키는 기능입니다. 
사용자가 화면을 나가거나 앱을 종료할 때 진행 중인 작업을 정리해야 합니다.

**사용법**
`cancel()`로 취소하고, `isActive`로 상태 확인

```kotlin
val job = launch {
    repeat(1000) {
        if (!isActive) return@launch
        delay(100)
        // 작업 수행
    }
}

job.cancel() // 작업 취소
```

## **9. 코루틴 컨텍스트 (Coroutine Context)**

**원리**
코루틴의 "신분증" 같은 개념입니다. 
어느 쓰레드에서 실행되는지, 
어떤 이름을 가지는지, 
에러 처리는 어떻게 할지 등의 정보를 담고 있습니다.

**사용법**
여러 컨텍스트를 `+` 연산자로 조합

```kotlin
launch(Dispatchers.IO + CoroutineName("MyCoroutine")) {
    // IO 쓰레드에서 "MyCoroutine"이라는 이름으로 실행
}
```

## **10. 구조화된 동시성 (Structured Concurrency)**

**원리**
부모-자식 관계를 가진 코루틴들이 서로 연결되어 있어서,
부모가 취소되면 자식들도 모두 자동으로 취소되는 구조입니다. 
메모리 누수를 방지하는 핵심 원리입니다.

**사용법**
`coroutineScope`와 `supervisorScope` 활용

```kotlin
coroutineScope {
    launch { /* 자식 코루틴 1 */ }
    launch { /* 자식 코루틴 2 */ }
    // 모든 자식이 완료될 때까지 기다림
}
```

## **11. 백프레셔 (Backpressure)**

**원리**
데이터를 생산하는 속도가 
소비하는 속도보다 빠를 때 
발생하는 문제를 해결하는 기술입니다. 

버퍼가 가득 차면 어떻게 할지 정하는 전략입니다.

**사용법**
Flow에서 `buffer()`, `conflate()`, `collectLatest()` 사용

```kotlin
flow.buffer(50) // 50개까지 버퍼링
    .collect { /* 처리 */ }
```

## **12. 콜드 스트림 vs 핫 스트림**

**원리**
- **콜드 스트림**: 구독자가 있을 때만 데이터를 생산 (Flow)
- **핫 스트림**: 구독자가 없어도 계속 데이터를 생산 (Channel, StateFlow)

**사용법**
상황에 따라 적절한 타입 선택

```kotlin
// 콜드 스트림
val coldFlow = flow { emit(getData()) }

// 핫 스트림  
val hotChannel = Channel<String>()
```

## **이외 중요한 패턴들**

#### **13. withContext 패턴**
```kotlin
suspend fun loadData(): String = withContext(Dispatchers.IO) {
    // 무거운 작업을 IO 쓰레드에서 실행
    "결과"
}
```

#### **14. 리소스 정리 패턴**
```kotlin
try {
    // 코루틴 작업
} finally {
    // 리소스 정리 (파일 닫기, 연결 해제 등)
}
```

#### **15. 타임아웃 처리**
```kotlin
withTimeout(5000) { // 5초 제한
    fetchData()
}
```

Coroutine을 통해
안드로이드 앱에서 ====복잡한 비동기 작업도 안전하고 효율적으로 처리====할 수 있습니다.