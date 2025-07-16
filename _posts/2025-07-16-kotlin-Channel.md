---
layout: single
title: Kotlin_Channel
date: 2025-07-16
category: kotlin
---
**주요 구성**
- **Channel 기본 개념** - 정의, 특징, 종류
- **실무 패턴** - Producer-Consumer, Fan-Out, Fan-In
- **Buffer 활용** - Buffered, Conflated Channel
- **실무 사례** - 파일 처리, 웹 크롤링, 실시간 채팅
- **고급 기능** - Select Expression, Pipeline
- **성능 최적화** - Buffer 크기 조정, 에러 처리

## ➡️  Channel이란?

#### - 기본 개념
**Channel**은 코루틴 간에 데이터를 **안전하게 전달**하는 통로입니다. 마치 두 사람이 편지를 주고받는 우편함과 같습니다. 한 코루틴은 **sender**가 되어 데이터를 보내고, 다른 코루틴은 **receiver**가 되어 데이터를 받습니다.

#### - Channel의 특징
- **Thread-Safe**: 여러 코루틴이 동시에 접근해도 안전합니다
- **Suspend Function**: 데이터를 보내고 받을 때 코루틴이 **suspend**됩니다
- **Buffer**: 데이터를 **임시 저장**할 수 있는 공간이 있습니다
- **Flow Control**: 데이터 흐름을 **제어**할 수 있습니다

## ➡️  Channel의 종류

#### - Unlimited Channel
**buffer 크기 제한이 없는** channel입니다. sender가 데이터를 보낼 때 **block되지 않습니다**.
#### - Buffered Channel
**정해진 크기의 buffer**를 가진 channel입니다. buffer가 **가득 차면** sender가 **suspend**됩니다.
#### - Rendezvous Channel
**buffer가 0인** channel입니다. sender와 receiver가 **동시에** 만나야 데이터 전달이 가능합니다.
#### - Conflated Channel
**가장 최신 데이터만** 유지하는 channel입니다. 새로운 데이터가 오면 **이전 데이터를 버립니다**.

## ➡️  실무에서 사용되는 Channel 패턴

#### 1. Producer-Consumer Pattern

하나의 **producer**가 데이터를 생성하고, 여러 **consumer**가 데이터를 처리하는 패턴입니다.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*

fun main() = runBlocking {
    // Unlimited channel 생성
    val channel = Channel<Int>(Channel.UNLIMITED)
    
    // Producer 코루틴
    launch {
        for (i in 1..5) {
            println("Producer: $i 전송")
            channel.send(i)
            delay(100)
        }
        channel.close()
    }
    
    // Consumer 코루틴
    launch {
        for (value in channel) {
            println("Consumer: $value 수신")
            delay(200)
        }
    }
    
    delay(2000)
}
```

#### 2. Fan-Out Pattern

하나의 **producer**가 여러 **consumer**에게 데이터를 분배하는 패턴입니다.

```kotlin
fun main() = runBlocking {
    val channel = Channel<Int>()
    
    // Producer
    launch {
        for (i in 1..10) {
            channel.send(i)
            println("Producer: $i 전송")
        }
        channel.close()
    }
    
    // Consumer 1
    launch {
        for (value in channel) {
            println("Consumer 1: $value 처리")
            delay(100)
        }
    }
    
    // Consumer 2
    launch {
        for (value in channel) {
            println("Consumer 2: $value 처리")
            delay(150)
        }
    }
    
    delay(3000)
}
```

#### 3. Fan-In Pattern
여러 **producer**가 하나의 **consumer**에게 데이터를 전송하는 패턴입니다.

```kotlin
fun main() = runBlocking {
    val channel = Channel<String>()
    
    // Producer 1
    launch {
        for (i in 1..3) {
            channel.send("Producer1: $i")
            delay(200)
        }
    }
    
    // Producer 2
    launch {
        for (i in 1..3) {
            channel.send("Producer2: $i")
            delay(300)
        }
    }
    
    // Consumer
    launch {
        repeat(6) {
            val value = channel.receive()
            println("Consumer: $value 수신")
        }
        channel.close()
    }
    
    delay(2000)
}
```

## ➡️  Channel Buffer 활용

#### 1. Buffered Channel
**정해진 크기의 buffer**를 가진 channel로, **성능 최적화**에 중요합니다.

```kotlin
fun main() = runBlocking {
    // Buffer 크기 3인 channel
    val channel = Channel<Int>(3)
    
    // Fast Producer
    launch {
        for (i in 1..10) {
            println("Producer: $i 전송 시작")
            channel.send(i)
            println("Producer: $i 전송 완료")
        }
        channel.close()
    }
    
    // Slow Consumer
    launch {
        for (value in channel) {
            println("Consumer: $value 수신")
            delay(500) // 느린 처리
        }
    }
    
    delay(8000)
}
```

#### 2. Conflated Channel
**최신 데이터만** 유지하는 channel로, **real-time 업데이트**에 유용합니다.

```kotlin
fun main() = runBlocking {
    val channel = Channel<Int>(Channel.CONFLATED)
    
    // Fast Producer
    launch {
        for (i in 1..10) {
            channel.send(i)
            println("Producer: $i 전송")
            delay(50)
        }
        channel.close()
    }
    
    // Slow Consumer
    launch {
        for (value in channel) {
            println("Consumer: $value 수신")
            delay(200)
        }
    }
    
    delay(3000)
}
```

## ➡️  활용 사례

#### 1. 파일 처리 시스템
**대용량 파일**을 **chunk** 단위로 나누어 처리하는 시스템입니다.

```kotlin
data class FileChunk(val id: Int, val data: String)

fun main() = runBlocking {
    val fileChannel = Channel<FileChunk>()
    val resultChannel = Channel<String>()
    
    // File Reader
    launch {
        for (i in 1..10) {
            val chunk = FileChunk(i, "Data chunk $i")
            fileChannel.send(chunk)
            println("File Reader: Chunk $i 읽음")
            delay(100)
        }
        fileChannel.close()
    }
    
    // File Processor
    launch {
        for (chunk in fileChannel) {
            val processed = "Processed: ${chunk.data}"
            resultChannel.send(processed)
            println("File Processor: ${chunk.id} 처리 완료")
            delay(200)
        }
        resultChannel.close()
    }
    
    // Result Handler
    launch {
        for (result in resultChannel) {
            println("Result Handler: $result")
        }
    }
    
    delay(5000)
}
```

#### 2. 웹 크롤링 시스템
**여러 URL**을 **병렬로** 크롤링하는 시스템입니다.

```kotlin
data class UrlRequest(val url: String)
data class CrawlResult(val url: String, val content: String)

fun main() = runBlocking {
    val requestChannel = Channel<UrlRequest>()
    val resultChannel = Channel<CrawlResult>()
    
    // URL Producer
    launch {
        val urls = listOf(
            "https://example1.com",
            "https://example2.com",
            "https://example3.com"
        )
        
        for (url in urls) {
            requestChannel.send(UrlRequest(url))
            println("URL Producer: $url 추가")
        }
        requestChannel.close()
    }
    
    // Crawler Workers (병렬 처리)
    repeat(2) { workerId ->
        launch {
            for (request in requestChannel) {
                // 실제로는 HTTP 요청
                val result = CrawlResult(
                    request.url,
                    "Content from ${request.url}"
                )
                resultChannel.send(result)
                println("Crawler $workerId: ${request.url} 크롤링 완료")
                delay(500)
            }
        }
    }
    
    // Result Processor
    launch {
        var receivedCount = 0
        for (result in resultChannel) {
            println("Result: ${result.url} -> ${result.content}")
            receivedCount++
            if (receivedCount == 3) break
        }
        resultChannel.close()
    }
    
    delay(3000)
}
```

#### 3. 실시간 채팅 시스템
**실시간 메시지**를 **여러 클라이언트**에게 전달하는 시스템입니다.

```kotlin
data class Message(val sender: String, val content: String, val timestamp: Long)

class ChatServer {
    private val messageChannel = Channel<Message>(Channel.UNLIMITED)
    private val clients = mutableListOf<Channel<Message>>()
    
    suspend fun addClient(): Channel<Message> {
        val clientChannel = Channel<Message>(Channel.UNLIMITED)
        clients.add(clientChannel)
        return clientChannel
    }
    
    suspend fun sendMessage(message: Message) {
        messageChannel.send(message)
    }
    
    suspend fun start() {
        for (message in messageChannel) {
            println("Server: ${message.sender}님이 메시지 전송")
            
            // 모든 클라이언트에게 broadcast
            clients.forEach { client ->
                client.send(message)
            }
        }
    }
}

fun main() = runBlocking {
    val server = ChatServer()
    
    // Server 시작
    launch {
        server.start()
    }
    
    // Client 1
    launch {
        val clientChannel = server.addClient()
        
        // 메시지 수신
        launch {
            for (message in clientChannel) {
                println("Client 1 수신: [${message.sender}] ${message.content}")
            }
        }
    }
    
    // Client 2
    launch {
        val clientChannel = server.addClient()
        
        // 메시지 수신
        launch {
            for (message in clientChannel) {
                println("Client 2 수신: [${message.sender}] ${message.content}")
            }
        }
    }
    
    delay(100)
    
    // 메시지 전송
    server.sendMessage(Message("User1", "안녕하세요!", System.currentTimeMillis()))
    server.sendMessage(Message("User2", "반갑습니다!", System.currentTimeMillis()))
    
    delay(1000)
}
```

## ➡️  Channel 고급 

#### 1. Select Expression
**여러 channel**을 **동시에** 처리할 수 있는 기능입니다.

```kotlin
fun main() = runBlocking {
    val channel1 = Channel<String>()
    val channel2 = Channel<String>()
    
    // Producer 1
    launch {
        delay(100)
        channel1.send("Channel 1 데이터")
    }
    
    // Producer 2
    launch {
        delay(200)
        channel2.send("Channel 2 데이터")
    }
    
    // Select를 사용한 처리
    select<Unit> {
        channel1.onReceive { value ->
            println("Channel 1에서 수신: $value")
        }
        channel2.onReceive { value ->
            println("Channel 2에서 수신: $value")
        }
    }
    
    delay(500)
}
```

#### 2. Channel Pipeline
**여러 처리 단계**를 **pipeline**으로 연결하는 패턴입니다.

```kotlin
fun main() = runBlocking {
    val numbers = produce {
        for (i in 1..10) {
            send(i)
        }
    }
    
    val squares = produce {
        for (number in numbers) {
            send(number * number)
        }
    }
    
    val filtered = produce {
        for (square in squares) {
            if (square % 2 == 0) {
                send(square)
            }
        }
    }
    
    // 최종 결과 처리
    for (result in filtered) {
        println("Pipeline 결과: $result")
    }
}
```

## ➡️  Channel 성능 최적화

#### 1. Buffer Size 조정
**적절한 buffer 크기**를 설정하여 **성능을 최적화**합니다.

```kotlin
fun main() = runBlocking {
    // CPU 코어 수만큼 buffer 설정
    val optimalBuffer = Runtime.getRuntime().availableProcessors()
    val channel = Channel<Int>(optimalBuffer)
    
    // 성능 측정
    val startTime = System.currentTimeMillis()
    
    // Fast Producer
    launch {
        for (i in 1..1000) {
            channel.send(i)
        }
        channel.close()
    }
    
    // Consumer
    launch {
        for (value in channel) {
            // 간단한 처리 시뮬레이션
            value * 2
        }
        
        val endTime = System.currentTimeMillis()
        println("처리 시간: ${endTime - startTime}ms")
    }
    
    delay(2000)
}
```

#### 2. 에러 처리
**Channel**에서 발생하는 **exception**을 적절히 처리합니다.

```kotlin
fun main() = runBlocking {
    val channel = Channel<Int>()
    
    // Producer with error handling
    launch {
        try {
            for (i in 1..5) {
                if (i == 3) {
                    throw RuntimeException("Producer 에러 발생")
                }
                channel.send(i)
            }
        } catch (e: Exception) {
            println("Producer 에러: ${e.message}")
            channel.close(e)
        }
    }
    
    // Consumer with error handling
    launch {
        try {
            for (value in channel) {
                println("Consumer: $value")
            }
        } catch (e: Exception) {
            println("Consumer 에러: ${e.message}")
        }
    }
    
    delay(1000)
}
```

## Best Practices

#### 1. Channel 생명주기 관리
Channel을 적절히 닫아 **메모리 누수를 방지**합니다.

#### 2. 적절한 Buffer 크기
**producer**와 **consumer**의 **처리 속도**를 고려하여 buffer 크기를 설정합니다.

#### 3. 에러 처리
Channel에서 발생하는 **exception**을 적절히 처리합니다.

#### 4. 성능 모니터링
Channel의 **throughput**과 **latency**를 모니터링하여 최적화합니다.

Channel은 코루틴 기반 **비동기 프로그래밍**에서 
**데이터 스트림**을 안전하고 효율적으로 처리하는 핵심 도구입니다. 
적절한 **buffer 전략**과 **패턴**을 사용하면 **고성능** 애플리케이션을 구축할 수 있습니다.