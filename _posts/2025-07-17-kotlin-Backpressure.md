---
layout: single
title: Kotlin_Backpressure
date: 2025-07-17
category: kotlin
---
**주요 구성**
- **Backpressure** - 개념, 발생 상황, 다른 프레임워크와 비교
- **문제 시나리오** - 메모리 누수, 시스템 과부하 상황
- **해결 전략** - Buffer 제한, Drop 전략, Custom 로직
- **실무 패턴** - Rate Limiting, Batch Processing, Circuit Breaker
- **실시간 처리** - 로그 처리, 센서 데이터 처리
- **성능 최적화** - 모니터링, 적응형 Buffer 크기 조정.  

## 🌼 Backpressure란?

#### 🌱 기본 개념
**Backpressure**는
**producer**가 **consumer**보다 빠르게 데이터를 생성할 때 발생하는 문제입니다. 
마치 물이 너무 빠르게 흘러서 배관이 터질 수 있는 것처럼, 

데이터가 너무 빠르게 생성되면 
**memory overflow**나 **system crash**가 발생할 수 있습니다.

#### Backpressure가 발생하는 상황
- **Fast Producer**: 데이터 생성 속도가 매우 빠름
- **Slow Consumer**: 데이터 처리 속도가 느림
- **Network Latency**: 네트워크 지연으로 인한 처리 지연
- **Resource Limitation**: CPU, Memory 등 자원 부족

#### 다른 프레임워크와의 비교
- **RxJava**: Observable의 onBackpressureBuffer(), onBackpressureDrop() 등
- **Reactor**: Flux의 onBackpressureBuffer(), onBackpressureDrop() 등
- **Akka Streams**: Flow control을 통한 자동 backpressure 처리
- **Node.js Streams**: pipe()의 자동 backpressure 처리

## 🍭 Backpressure 문제 시나리오

#### 메모리 누수 문제
**Unbounded buffer**를 사용할 때 발생하는 전형적인 문제입니다.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*

fun demonstrateMemoryLeak() = runBlocking {
    // Unlimited buffer - 메모리 누수 위험
    val channel = Channel<Int>(Channel.UNLIMITED)
    
    // Fast Producer - 초당 1000개 데이터 생성
    launch {
        var counter = 0
        while (true) {
            channel.send(counter++)  // 채널에 데이터 전송
            delay(1)  // 1ms 간격으로 매우 빠른 생성
        }
    }
    
    // Slow Consumer - 초당 10개 데이터 처리
    launch {
        for (value in channel) {
            println("Processing: $value")  // 데이터 처리 시뮬레이션
            delay(100)  // 100ms 간격으로 느린 처리
        }
    }
    
    delay(5000)  // 5초 후 종료
    // 결과: 메모리에 약 5000개의 미처리 데이터가 쌓임
}
```

#### System Overload 문제
**Resource 부족**으로 인한 system 전체 성능 저하입니다.

```kotlin
fun demonstrateSystemOverload() = runBlocking {
    // CPU 집약적인 작업을 시뮬레이션
    val channel = Channel<ByteArray>(1000)
    
    // Heavy Data Producer
    launch {
        repeat(10000) { i ->
            // 대용량 데이터 생성 (1MB씩)
            val data = ByteArray(1024 * 1024) { (i % 256).toByte() }
            channel.send(data)  // 채널에 대용량 데이터 전송
            println("Producer: $i MB 생성")
        }
        channel.close()  // 생산 완료 후 채널 닫기
    }
    
    // Slow Processing Consumer
    launch {
        for (data in channel) {
            // 복잡한 데이터 처리 시뮬레이션
            val checksum = data.sum()  // 체크섬 계산
            println("Consumer: 처리 완료 (checksum: $checksum)")
            delay(1000)  // 1초씩 느린 처리
        }
    }
    
    delay(30000)  // 30초 대기
    // 결과: 메모리 사용량 급증, GC 압박, 성능 저하
}
```

## 🍭 Backpressure 해결

#### 1. Buffer 크기 제한

**Bounded buffer**를 사용하여 메모리 사용량을 제한합니다.

```kotlin
fun solutionBoundedBuffer() = runBlocking {
    // Buffer 크기를 10으로 제한
    val channel = Channel<Int>(10)
    
    // Fast Producer
    launch {
        for (i in 1..100) {
            try {
                channel.send(i)  // buffer가 가득 차면 suspend됨
                println("Producer: $i 전송")
            } catch (e: Exception) {
                println("Producer Error: ${e.message}")
            }
        }
        channel.close()  // producer 완료 후 채널 닫기
    }
    
    // Slow Consumer
    launch {
        for (value in channel) {
            println("Consumer: $value 처리 중...")
            delay(100)  // 느린 처리 시뮬레이션
            println("Consumer: $value 처리 완료")
        }
    }
    
    delay(15000)
    // 결과: producer가 consumer 속도에 맞춰 자동으로 조절됨
}
```

#### 2. Drop 전략
**가장 오래된 데이터**나 **새로운 데이터**를 버리는 전략입니다.

```kotlin
fun solutionDropStrategy() = runBlocking {
    // Conflated Channel - 최신 데이터만 유지
    val channel = Channel<Int>(Channel.CONFLATED)
    
    // Fast Producer
    launch {
        for (i in 1..100) {
            channel.send(i)  // 이전 데이터는 자동으로 버려짐
            println("Producer: $i 전송")
            delay(10)  // 빠른 생성
        }
        channel.close()
    }
    
    // Slow Consumer
    launch {
        for (value in channel) {
            println("Consumer: $value 수신")
            delay(500)  // 매우 느린 처리
        }
    }
    
    delay(3000)
    // 결과: 중간 데이터들이 버려지고 최신 데이터만 처리됨
}
```

#### 3. 수동 Drop 구현
**Custom logic**으로 데이터를 선택적으로 버립니다.

```kotlin
data class PriorityData(val value: Int, val priority: Int)

fun solutionCustomDrop() = runBlocking {
    val channel = Channel<PriorityData>(5)
    
    // Producer with priority
    launch {
        repeat(20) { i ->
            val priority = if (i % 5 == 0) 1 else 2  // 5의 배수는 높은 우선순위
            val data = PriorityData(i, priority)
            
            // 채널이 가득 찬 경우 처리
            if (channel.trySend(data).isSuccess) {
                println("Producer: 우선순위 ${data.priority} 데이터 $i 전송")
            } else {
                println("Producer: 데이터 $i 버림 (채널 가득참)")
            }
            delay(50)
        }
        channel.close()
    }
    
    // Consumer
    launch {
        for (data in channel) {
            println("Consumer: 우선순위 ${data.priority} 데이터 ${data.value} 처리")
            delay(300)  // 느린 처리
        }
    }
    
    delay(8000)
    // 결과: 우선순위 높은 데이터가 우선 처리됨
}
```

## 🍭 Backpressure 패턴

#### 1. Rate Limiting
**데이터 생성 속도**를 제한하여 backpressure를 예방합니다.

```kotlin
import kotlinx.coroutines.sync.Semaphore

class RateLimiter(private val maxRate: Int) {
    private val semaphore = Semaphore(maxRate)  // 동시 처리 제한
    
    suspend fun acquire() {
        semaphore.acquire()  // 허가 획득
    }
    
    fun release() {
        semaphore.release()  // 허가 반환
    }
}

fun rateLimitingExample() = runBlocking {
    val channel = Channel<Int>(50)
    val rateLimiter = RateLimiter(10)  // 동시 10개까지 허용
    
    // Rate Limited Producer
    launch {
        repeat(100) { i ->
            rateLimiter.acquire()  // 처리 허가 대기
            
            launch {
                try {
                    channel.send(i)
                    println("Producer: $i 전송")
                } finally {
                    rateLimiter.release()  // 처리 완료 후 허가 반환
                }
            }
            delay(10)
        }
        channel.close()
    }
    
    // Consumer
    launch {
        for (value in channel) {
            println("Consumer: $value 처리")
            delay(200)
        }
    }
    
    delay(25000)
    // 결과: 생산 속도가 제한되어 안정적인 처리
}
```

#### 2. Batch Processing
**여러 데이터를 묶어서** 처리하여 효율성을 높입니다.

```kotlin
fun batchProcessingExample() = runBlocking {
    val channel = Channel<Int>(Channel.UNLIMITED)
    
    // Fast Producer
    launch {
        repeat(100) { i ->
            channel.send(i)
            delay(10)  // 빠른 생성
        }
        channel.close()
    }
    
    // Batch Consumer
    launch {
        val batch = mutableListOf<Int>()  // 배치 저장소
        
        for (value in channel) {
            batch.add(value)  // 배치에 추가
            
            // 배치 크기가 10이 되면 처리
            if (batch.size >= 10) {
                processBatch(batch.toList())  // 배치 처리
                batch.clear()  // 배치 초기화
            }
        }
        
        // 남은 데이터 처리
        if (batch.isNotEmpty()) {
            processBatch(batch.toList())
        }
    }
    
    delay(5000)
}

// 배치 처리 함수
suspend fun processBatch(batch: List<Int>) {
    println("Batch Processing: ${batch.joinToString(", ")}")
    delay(500)  // 배치 처리 시뮬레이션
    println("Batch Complete: ${batch.size} items processed")
}
```

#### 3. Circuit Breaker Pattern
**시스템 과부하 시** 요청을 차단하여 시스템을 보호합니다.

```kotlin
enum class CircuitState { CLOSED, OPEN, HALF_OPEN }

class CircuitBreaker(
    private val failureThreshold: Int = 5,
    private val timeout: Long = 5000
) {
    private var state = CircuitState.CLOSED  // 초기 상태
    private var failureCount = 0  // 실패 횟수
    private var lastFailureTime = 0L  // 마지막 실패 시간
    
    suspend fun execute(operation: suspend () -> Unit): Boolean {
        when (state) {
            CircuitState.OPEN -> {
                // 타임아웃 후 HALF_OPEN으로 전환
                if (System.currentTimeMillis() - lastFailureTime > timeout) {
                    state = CircuitState.HALF_OPEN
                    println("Circuit Breaker: HALF_OPEN 상태로 전환")
                } else {
                    println("Circuit Breaker: 요청 차단됨")
                    return false
                }
            }
            CircuitState.HALF_OPEN -> {
                try {
                    operation()  // 테스트 요청 실행
                    state = CircuitState.CLOSED  // 성공 시 정상 상태로
                    failureCount = 0
                    println("Circuit Breaker: CLOSED 상태로 복구")
                    return true
                } catch (e: Exception) {
                    state = CircuitState.OPEN  // 실패 시 차단 상태로
                    lastFailureTime = System.currentTimeMillis()
                    println("Circuit Breaker: 다시 OPEN 상태로 전환")
                    return false
                }
            }
            CircuitState.CLOSED -> {
                try {
                    operation()  // 정상 요청 실행
                    failureCount = 0  // 성공 시 실패 카운트 초기화
                    return true
                } catch (e: Exception) {
                    failureCount++
                    if (failureCount >= failureThreshold) {
                        state = CircuitState.OPEN  // 임계값 초과 시 차단
                        lastFailureTime = System.currentTimeMillis()
                        println("Circuit Breaker: OPEN 상태로 전환")
                    }
                    return false
                }
            }
        }
        return false
    }
}

fun circuitBreakerExample() = runBlocking {
    val channel = Channel<Int>(10)
    val circuitBreaker = CircuitBreaker()
    
    // Producer with Circuit Breaker
    launch {
        repeat(20) { i ->
            val success = circuitBreaker.execute {
                if (i > 10) {
                    throw RuntimeException("시스템 과부하")  // 인위적 실패
                }
                channel.send(i)
                println("Producer: $i 전송")
            }
            
            if (!success) {
                println("Producer: $i 전송 실패 (Circuit Breaker)")
            }
            
            delay(200)
        }
        channel.close()
    }
    
    // Consumer
    launch {
        for (value in channel) {
            println("Consumer: $value 처리")
            delay(100)
        }
    }
    
    delay(8000)
    // 결과: 시스템 과부하 시 자동으로 요청 차단
}
```

## 🍭 실시간 데이터 처리 시나리오

#### 1. 실시간 로그 처리
**고속으로 생성되는 로그**를 안정적으로 처리합니다.

```kotlin
data class LogEntry(val timestamp: Long, val level: String, val message: String)

fun realTimeLogProcessing() = runBlocking {
    // 로그 저장용 채널 (buffer 제한)
    val logChannel = Channel<LogEntry>(1000)
    
    // Fast Log Producer (웹서버 로그 시뮬레이션)
    launch {
        repeat(10000) { i ->
            val logEntry = LogEntry(
                timestamp = System.currentTimeMillis(),
                level = if (i % 10 == 0) "ERROR" else "INFO",
                message = "Request $i processed"
            )
            
            // 채널이 가득 찬 경우 처리
            if (logChannel.trySend(logEntry).isSuccess) {
                // 성공적으로 전송됨
            } else {
                // 로그 유실 카운트 (실무에서는 metrics 시스템으로 전송)
                println("Log dropped: $i")
            }
            
            delay(1)  // 1ms 간격으로 빠른 로그 생성
        }
        logChannel.close()
    }
    
    // Log Processor (여러 consumer로 병렬 처리)
    repeat(3) { processorId ->
        launch {
            for (logEntry in logChannel) {
                // 로그 레벨별 처리
                when (logEntry.level) {
                    "ERROR" -> {
                        println("Processor $processorId: 🚨 ERROR 처리 - ${logEntry.message}")
                        // 실무: 알림 시스템으로 전송
                        delay(50)  // ERROR 로그는 더 오래 처리
                    }
                    "INFO" -> {
                        println("Processor $processorId: ℹ️ INFO 처리 - ${logEntry.message}")
                        // 실무: 데이터베이스나 검색 엔진으로 전송
                        delay(10)  // INFO 로그는 빠르게 처리
                    }
                }
            }
        }
    }
    
    delay(15000)
    // 결과: 다중 consumer로 병렬 처리하여 throughput 향상
}
```

#### 2. 실시간 센서 데이터 처리
**IoT 센서 데이터**를 실시간으로 처리합니다.

```kotlin
data class SensorData(
    val sensorId: String,
    val timestamp: Long,
    val temperature: Double,
    val humidity: Double
)

fun sensorDataProcessing() = runBlocking {
    // 센서 데이터 채널 (최신 데이터만 유지)
    val sensorChannel = Channel<SensorData>(Channel.CONFLATED)
    
    // Sensor Data Producer
    launch {
        val sensorIds = listOf("TEMP_01", "TEMP_02", "TEMP_03")
        
        repeat(1000) { i ->
            sensorIds.forEach { sensorId ->
                val data = SensorData(
                    sensorId = sensorId,
                    timestamp = System.currentTimeMillis(),
                    temperature = 20.0 + (Math.random() * 10), // 20-30도 랜덤
                    humidity = 50.0 + (Math.random() * 30)      // 50-80% 랜덤
                )
                
                // 최신 데이터만 유지 (이전 데이터는 자동으로 버려짐)
                sensorChannel.send(data)
                println("Sensor: ${data.sensorId} 데이터 전송")
            }
            delay(100)  // 100ms 간격으로 데이터 전송
        }
        sensorChannel.close()
    }
    
    // Real-time Data Processor
    launch {
        for (data in sensorChannel) {
            // 임계값 확인
            when {
                data.temperature > 28.0 -> {
                    println("🔥 온도 경고: ${data.sensorId} - ${data.temperature}°C")
                    // 실무: 알림 시스템으로 즉시 전송
                }
                data.humidity > 75.0 -> {
                    println("💧 습도 경고: ${data.sensorId} - ${data.humidity}%")
                    // 실무: 환기 시스템 자동 제어
                }
                else -> {
                    println("✅ 정상: ${data.sensorId} - ${data.temperature}°C, ${data.humidity}%")
                }
            }
            
            delay(50)  // 센서 데이터 분석 시간
        }
    }
    
    delay(12000)
    // 결과: 실시간 센서 모니터링으로 즉시 대응 가능
}
```

## 🍭 성능 모니터링과 최적화

#### 1. 채널 성능 모니터링
**실시간으로** 채널 상태를 모니터링합니다.

```kotlin
class ChannelMonitor<T>(private val channel: Channel<T>) {
    private var sentCount = 0L      // 전송된 데이터 수
    private var receivedCount = 0L  // 수신된 데이터 수
    private var droppedCount = 0L   // 버려진 데이터 수
    private val startTime = System.currentTimeMillis()
    
    suspend fun send(data: T): Boolean {
        return if (channel.trySend(data).isSuccess) {
            sentCount++
            true
        } else {
            droppedCount++
            false
        }
    }
    
    suspend fun receive(): T {
        val data = channel.receive()
        receivedCount++
        return data
    }
    
    fun getStats(): String {
        val currentTime = System.currentTimeMillis()
        val elapsedSeconds = (currentTime - startTime) / 1000.0
        
        return """
            📊 채널 통계:
            - 전송: $sentCount (${String.format("%.2f", sentCount / elapsedSeconds)}/초)
            - 수신: $receivedCount (${String.format("%.2f", receivedCount / elapsedSeconds)}/초)
            - 버림: $droppedCount (${String.format("%.2f", droppedCount / elapsedSeconds)}/초)
            - 손실률: ${String.format("%.2f", droppedCount * 100.0 / (sentCount + droppedCount))}%
        """.trimIndent()
    }
}

fun monitoringExample() = runBlocking {
    val channel = Channel<Int>(10)
    val monitor = ChannelMonitor(channel)
    
    // Producer
    launch {
        repeat(1000) { i ->
            if (monitor.send(i)) {
                println("Producer: $i 전송 성공")
            } else {
                println("Producer: $i 전송 실패")
            }
            delay(10)
        }
        channel.close()
    }
    
    // Consumer
    launch {
        try {
            while (true) {
                val data = monitor.receive()
                println("Consumer: $data 처리")
                delay(50)  // 느린 처리
            }
        } catch (e: Exception) {
            println("Consumer 종료: ${e.message}")
        }
    }
    
    // 모니터링 출력
    launch {
        repeat(10) {
            delay(1000)
            println(monitor.getStats())
        }
    }
    
    delay(12000)
    // 결과: 실시간 성능 지표로 시스템 상태 파악
}
```

#### 2. 적응형 Buffer 크기 조정
**시스템 상황에 따라** buffer 크기를 동적으로 조정합니다.

```kotlin
class AdaptiveChannel<T>(initialCapacity: Int = 10) {
    private var currentChannel = Channel<T>(initialCapacity)
    private var currentCapacity = initialCapacity
    private var utilizationHistory = mutableListOf<Double>()
    
    suspend fun send(data: T) {
        if (currentChannel.trySend(data).isFailure) {
            // 채널이 가득 찬 경우 확장 고려
            adaptCapacity()
            // 새 채널에 재시도
            currentChannel.send(data)
        }
    }
    
    suspend fun receive(): T {
        return currentChannel.receive()
    }
    
    private fun adaptCapacity() {
        val currentUtilization = getCurrentUtilization()
        utilizationHistory.add(currentUtilization)
        
        // 최근 5개 측정값의 평균 계산
        if (utilizationHistory.size > 5) {
            utilizationHistory.removeAt(0)
        }
        
        val averageUtilization = utilizationHistory.average()
        
        when {
            averageUtilization > 0.8 && currentCapacity < 1000 -> {
                // 사용률이 80% 이상이면 용량 증가
                val newCapacity = (currentCapacity * 1.5).toInt()
                println("📈 Buffer 확장: $currentCapacity -> $newCapacity")
                recreateChannel(newCapacity)
            }
            averageUtilization < 0.3 && currentCapacity > 10 -> {
                // 사용률이 30% 이하이면 용량 감소
                val newCapacity = (currentCapacity * 0.7).toInt()
                println("📉 Buffer 축소: $currentCapacity -> $newCapacity")
                recreateChannel(newCapacity)
            }
        }
    }
    
    private fun getCurrentUtilization(): Double {
        // 실제 구현에서는 채널의 현재 사용률을 측정
        // 여기서는 시뮬레이션으로 랜덤 값 반환
        return Math.random()
    }
    
    private fun recreateChannel(newCapacity: Int) {
        val oldChannel = currentChannel
        currentChannel = Channel(newCapacity)
        currentCapacity = newCapacity
        
        // 기존 데이터 이주 (실제 구현에서는 더 복잡)
        // 여기서는 단순화
    }
}
```

##  📘 Best Practices 정리

#### 1. 적절한 Buffer 전략 선택
- **Bounded Buffer**: 메모리 사용량 제한
- **Conflated**: 실시간 데이터에 적합
- **Unlimited**: 신중하게 사용 (메모리 누수 위험)
#### 2. 모니터링과 알림
- **성능 지표** 실시간 수집
- **임계값 초과** 시 알림
- **Historical 데이터** 분석
#### 3. 점진적 장애 대응
- **Circuit Breaker** 패턴
- **Graceful Degradation**
- **Retry 전략**
#### 4. 리소스 관리
- **적절한 Thread Pool** 크기
- **메모리 사용량** 모니터링
- **GC 압박** 최소화

Backpressure 처리는 **안정적인 시스템**을 구축하는 핵심 요소입니다. 
적절한 **전략**과 **모니터링**을 통해 **고가용성** 시스템을 구축할 수 있습니다.