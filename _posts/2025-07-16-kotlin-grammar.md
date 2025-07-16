---
layout: single
title: Kotlin_Grammar
date: 2025-07-16
category: kotlin
---
## 코틀린 문법

#### 1. 기본 문법과 변수

#### 변수 선언

**val**과 **var**로 변수를 선언합니다.

```kotlin
val name = "홍길동"        // 읽기 전용 (변경 불가)
var age = 17              // 변경 가능
var score: Int = 95       // 타입 명시
```

#### 기본 데이터 타입

- **Int**: 정수 (32비트)
- **Long**: 큰 정수 (64비트)
- **Double**: 실수 (64비트)
- **Float**: 실수 (32비트)
- **String**: 문자열
- **Boolean**: 참/거짓
- **Char**: 단일 문자

```kotlin
val number: Int = 42
val price: Double = 19.99
val isStudent: Boolean = true
val grade: Char = 'A'
```

#### 문자열 다루기

```kotlin
val firstName = "길동"
val lastName = "홍"
val fullName = "$lastName$firstName"           // 문자열 템플릿
val message = "안녕하세요, ${fullName}님!"      // 표현식 사용
```

#### 널 안전성

```kotlin
var name: String = "홍길동"        // null 불가
var nickname: String? = null       // null 가능
var length = nickname?.length      // 안전한 호출
```

## 2. 조건문과 반복문

#### if 문

```kotlin
val score = 85
val grade = if (score >= 90) {
    "A"
} else if (score >= 80) {
    "B"
} else {
    "C"
}
```

#### when 문 (자바의 switch)

```kotlin
val day = 3
val dayName = when (day) {
    1 -> "월요일"
    2 -> "화요일"
    3 -> "수요일"
    in 4..5 -> "목금요일"
    else -> "주말"
}
```

#### for 반복문

```kotlin
// 범위 반복
for (i in 1..5) {
    println(i)
}

// 컬렉션 반복
val fruits = listOf("사과", "바나나", "오렌지")
for (fruit in fruits) {
    println(fruit)
}

// 인덱스와 함께
for ((index, fruit) in fruits.withIndex()) {
    println("$index: $fruit")
}
```

#### while 반복문

```kotlin
var count = 0
while (count < 5) {
    println("카운트: $count")
    count++
}
```

## 3. 함수

#### 함수 정의

```kotlin
fun greet(name: String): String {
    return "안녕하세요, ${name}님!"
}

// 단일 표현식 함수
fun add(a: Int, b: Int) = a + b

// 기본 매개변수
fun introduce(name: String, age: Int = 18) {
    println("이름: $name, 나이: $age")
}
```

#### 함수 호출

```kotlin
val message = greet("홍길동")
val sum = add(10, 20)
introduce("김철수")           // age는 기본값 18
introduce("이영희", 19)       // age를 명시적으로 전달
```

## 4. 클래스와 객체

#### 클래스 정의

```kotlin
class Student(val name: String, var grade: Int) {
    // 속성
    var isActive: Boolean = true
    
    // 메서드
    fun study() {
        println("${name}이(가) 공부하고 있습니다.")
    }
    
    fun getInfo(): String {
        return "이름: $name, 학년: $grade"
    }
}
```

#### 객체 생성과 사용

```kotlin
val student = Student("홍길동", 2)
student.study()
println(student.getInfo())
student.grade = 3  // var로 선언된 속성은 변경 가능
```

#### 상속

```kotlin
open class Person(val name: String) {
    open fun introduce() {
        println("안녕하세요, 저는 ${name}입니다.")
    }
}

class Teacher(name: String, val subject: String) : Person(name) {
    override fun introduce() {
        println("안녕하세요, 저는 ${subject} 선생님 ${name}입니다.")
    }
}
```

## 5. 컬렉션

#### 리스트

```kotlin
// 읽기 전용 리스트
val fruits = listOf("사과", "바나나", "오렌지")

// 변경 가능한 리스트
val mutableFruits = mutableListOf("사과", "바나나")
mutableFruits.add("오렌지")
mutableFruits.remove("바나나")
```

#### 맵

```kotlin
// 읽기 전용 맵
val scores = mapOf("홍길동" to 95, "김철수" to 87)

// 변경 가능한 맵
val mutableScores = mutableMapOf<String, Int>()
mutableScores["홍길동"] = 95
mutableScores["김철수"] = 87
```

#### 집합

```kotlin
val numbers = setOf(1, 2, 3, 2, 1)  // 중복 제거: {1, 2, 3}
val mutableNumbers = mutableSetOf<Int>()
mutableNumbers.add(1)
mutableNumbers.add(2)
```

## 6. 람다와 고차 함수

#### 람다 표현식

```kotlin
val numbers = listOf(1, 2, 3, 4, 5)

// 람다로 필터링
val evenNumbers = numbers.filter { it % 2 == 0 }

// 람다로 변환
val doubled = numbers.map { it * 2 }

// 람다로 정렬
val sorted = numbers.sortedBy { -it }  // 내림차순
```

#### 고차 함수

```kotlin
fun calculate(x: Int, y: Int, operation: (Int, Int) -> Int): Int {
    return operation(x, y)
}

val result1 = calculate(10, 5) { a, b -> a + b }  // 15
val result2 = calculate(10, 5) { a, b -> a * b }  // 50
```

## 7. 예외 처리

#### try-catch

```kotlin
fun divide(a: Int, b: Int): Int {
    return try {
        a / b
    } catch (e: ArithmeticException) {
        println("0으로 나눌 수 없습니다.")
        0
    } finally {
        println("계산이 완료되었습니다.")
    }
}
```

## 8. 확장 함수

#### 기존 클래스에 함수 추가

```kotlin
fun String.isValidEmail(): Boolean {
    return this.contains("@") && this.contains(".")
}

// 사용
val email = "student@school.com"
if (email.isValidEmail()) {
    println("유효한 이메일입니다.")
}
```

## 9. 데이터 클래스

#### 데이터를 담는 클래스

```kotlin
data class Student(
    val name: String,
    val age: Int,
    val grade: String
)

// 자동으로 equals, hashCode, toString 생성
val student1 = Student("홍길동", 17, "A")
val student2 = Student("홍길동", 17, "A")
println(student1 == student2)  // true
```

## 10. 실용적인 예제

#### 학생 성적 관리 시스템

```kotlin
data class Student(val name: String, val scores: MutableList<Int>)

class GradeManager {
    private val students = mutableListOf<Student>()
    
    fun addStudent(name: String) {
        students.add(Student(name, mutableListOf()))
    }
    
    fun addScore(name: String, score: Int) {
        val student = students.find { it.name == name }
        student?.scores?.add(score)
    }
    
    fun getAverage(name: String): Double? {
        val student = students.find { it.name == name }
        return student?.scores?.average()
    }
    
    fun getTopStudent(): Student? {
        return students.maxByOrNull { it.scores.average() }
    }
}

// 사용 예시
fun main() {
    val manager = GradeManager()
    
    manager.addStudent("홍길동")
    manager.addStudent("김철수")
    
    manager.addScore("홍길동", 95)
    manager.addScore("홍길동", 87)
    manager.addScore("김철수", 92)
    
    println("홍길동 평균: ${manager.getAverage("홍길동")}")
    println("최고 성적 학생: ${manager.getTopStudent()?.name}")
}
```

코틀린의 핵심 문법을 이해하였습니다.

코틀린은 자바와 100% 호환되면서도 
더 간결하고 안전한 코드를 작성할 수 있게 해주는 현대적인 언어입니다.