---
layout: single
title: Kotlin_Scope
date: 2025-07-16
category: kotlin
---
# Kotlin Scope 가이드

## Scope란 무엇인가?

**스코프(Scope)**는 변수나 함수가 사용될 수 있는 **범위**를 의미합니다. 마치 학교에서 교실, 복도, 운동장처럼 각각의 공간이 있듯이, 코드에서도 변수가 살아있는 공간이 정해져 있습니다.

#### 기본 개념

```kotlin
fun main() {
    val name = "홍길동"  // main 함수 스코프
    
    if (true) {
        val age = 17    // if 블록 스코프
        println("$name은 $age살입니다")  // name 접근 가능
    }
    
    // println(age)  // 에러! age는 if 블록 밖에서 접근 불가
}
```

## 1. 전역 스코프 (Global Scope)

**전역 스코프**는 파일 전체에서 접근할 수 있는 범위입니다. 학교 전체에서 사용할 수 있는 공용 시설과 같습니다.

#### 전역 변수와 함수

```kotlin
// 전역 변수
val SCHOOL_NAME = "한국고등학교"
var studentCount = 500

// 전역 함수
fun getSchoolInfo(): String {
    return "$SCHOOL_NAME - 학생 수: $studentCount명"
}

class Student {
    fun introduce() {
        // 클래스 안에서도 전역 변수 사용 가능
        println("저는 $SCHOOL_NAME 학생입니다")
    }
}

fun main() {
    println(getSchoolInfo())  // 전역 함수 호출
    val student = Student()
    student.introduce()
}
```

## 2. 함수 스코프 (Function Scope)

**함수 스코프**는 함수 내부에서만 접근할 수 있는 범위입니다. 각 교실 안에서만 사용할 수 있는 물건과 같습니다.

#### 함수 내부 변수

```kotlin
fun calculateGrade(score: Int): String {
    val maxScore = 100        // 함수 스코프 변수
    val percentage = score.toDouble() / maxScore * 100
    
    return when {
        percentage >= 90 -> "A"
        percentage >= 80 -> "B"
        percentage >= 70 -> "C"
        else -> "F"
    }
}

fun main() {
    val grade = calculateGrade(85)
    println("학점: $grade")
    
    // println(maxScore)  // 에러! 함수 밖에서 접근 불가
}
```

#### 매개변수의 Scope

```kotlin
fun greetStudent(name: String, grade: Int) {
    // name과 grade는 이 함수 안에서만 사용 가능
    val message = "안녕하세요, $grade학년 $name님!"
    println(message)
    
    fun innerFunction() {
        // 내부 함수에서도 매개변수 접근 가능
        println("내부에서: $name")
    }
    
    innerFunction()
}
```

## 3. 블록 스코프 (Block Scope)

**블록 스코프**는 중괄호 `{}` 안에서만 접근할 수 있는 범위입니다. 각 방 안에서만 사용할 수 있는 물건과 같습니다.

#### if 블록 스코프

```kotlin
fun checkAge(age: Int) {
    if (age >= 18) {
        val status = "성인"       // if 블록 스코프
        val message = "축하합니다! $status이 되었습니다"
        println(message)
    } else {
        val status = "미성년자"   // else 블록 스코프
        val message = "아직 $status입니다"
        println(message)
    }
    
    // println(status)  // 에러! 블록 밖에서 접근 불가
}
```

#### for 블록 스코프

```kotlin
fun printNumbers() {
    for (i in 1..5) {
        val doubled = i * 2  // for 블록 스코프
        println("$i의 2배는 $doubled입니다")
    }
    
    // println(i)       // 에러! 루프 변수 접근 불가
    // println(doubled) // 에러! 블록 변수 접근 불가
}
```

#### when 블록 스코프

```kotlin
fun getSeasonMessage(month: Int): String {
    return when (month) {
        in 3..5 -> {
            val season = "봄"          // when 블록 스코프
            val activity = "꽃구경"
            "$season에는 $activity을 즐겨보세요"
        }
        in 6..8 -> {
            val season = "여름"        // 다른 블록 스코프
            val activity = "수영"
            "$season에는 $activity을 즐겨보세요"
        }
        else -> "계절 정보 없음"
    }
}
```

## 4. Class Scope

**클래스 스코프**는 클래스 내부에서 접근할 수 있는 범위입니다. 한 반 안에서만 공유하는 물건과 같습니다.

#### 클래스 프로퍼티와 메서드

```kotlin
class Student(val name: String) {
    private var grade: Int = 1        // 클래스 스코프 (private)
    var isActive: Boolean = true      // 클래스 스코프 (public)
    
    fun promote() {
        grade++  // 클래스 내부에서 private 변수 접근 가능
        println("$name이(가) ${grade}학년으로 진급했습니다")
    }
    
    fun getInfo(): String {
        return "이름: $name, 학년: $grade, 활동상태: $isActive"
    }
    
    inner class Homework {
        fun submit() {
            // inner class에서 외부 클래스 변수 접근 가능
            println("$name의 숙제를 제출했습니다")
        }
    }
}
```

#### 접근 제한자와 Scope

```kotlin
class School {
    private val students = mutableListOf<String>()    // 클래스 내부만
    protected val teachers = mutableListOf<String>()  // 상속 클래스까지
    internal val facilities = mutableListOf<String>() // 같은 모듈
    public val name = "한국고등학교"                   // 모든 곳
    
    private fun addStudent(name: String) {
        students.add(name)
    }
    
    fun enrollStudent(name: String) {
        addStudent(name)  // 같은 클래스 내부에서 private 함수 호출 가능
        println("$name 학생이 등록되었습니다")
    }
}
```

## 5. 람다 스코프 (Lambda Scope)

**람다 스코프**는 람다 표현식 내부의 범위입니다. 람다는 **외부 변수를 캡처**할 수 있습니다.

#### 람다에서 외부 변수 접근

```kotlin
fun processScores() {
    val scores = listOf(85, 92, 78, 96, 88)
    var totalAbove90 = 0  // 람다 외부 변수
    
    scores.forEach { score ->
        if (score >= 90) {
            totalAbove90++  // 람다에서 외부 변수 수정 가능
        }
        println("점수: $score")
    }
    
    println("90점 이상 개수: $totalAbove90")
}
```

#### 람다 매개변수 스코프

```kotlin
fun filterStudents() {
    val students = listOf("홍길동", "김철수", "이영희", "박지민")
    
    val filteredStudents = students.filter { name ->
        // name은 람다 매개변수 (블록 스코프)
        name.length >= 3
    }
    
    val mappedStudents = students.map { student ->
        // student는 이 람다에서만 사용 가능
        "학생: $student"
    }
}
```

## 6. 스코프 함수 (Scope Functions)

**스코프 함수**는 객체의 컨텍스트 내에서 코드를 실행할 수 있게 해주는 함수입니다.

#### let 함수

```kotlin
fun processStudent() {
    val student: Student? = getStudent()
    
    student?.let { s ->
        // s는 null이 아닌 student 객체
        println("학생 이름: ${s.name}")
        s.promote()
    }
}
```

#### apply 함수

```kotlin
fun createStudent(): Student {
    return Student("홍길동").apply {
        // this는 Student 객체
        isActive = true
        promote()
    }
}
```

#### with 함수

```kotlin
fun printStudentInfo(student: Student) {
    with(student) {
        // this는 student 객체
        println("이름: $name")
        println("정보: ${getInfo()}")
    }
}
```

## 7. 스코프 충돌과 해결

**스코프 충돌**은 같은 이름의 변수가 여러 스코프에 있을 때 발생합니다.

#### 변수 쉐도잉 (Variable Shadowing)

```kotlin
val name = "전역 홍길동"  // 전역 변수

fun demonstrateScoping() {
    val name = "함수 홍길동"  // 함수 스코프 변수
    
    println("함수에서: $name")  // "함수 홍길동" 출력
    
    if (true) {
        val name = "블록 홍길동"  // 블록 스코프 변수
        println("블록에서: $name")  // "블록 홍길동" 출력
    }
    
    println("함수에서 다시: $name")  // "함수 홍길동" 출력
}
```

#### 클래스에서의 스코프 충돌

```kotlin
class Student(val name: String) {
    fun introduce(name: String) {  // 매개변수 name
        println("안녕하세요, 저는 ${this.name}입니다")  // 클래스 프로퍼티
        println("매개변수로 받은 이름: $name")           // 매개변수
    }
}
```

## 8. 실전 예제

#### 학생 성적 관리 시스템

```kotlin
class GradeManager {
    private val students = mutableMapOf<String, MutableList<Int>>()
    
    fun addStudent(name: String) {
        students[name] = mutableListOf()
        println("$name 학생이 추가되었습니다")
    }
    
    fun addScore(name: String, score: Int) {
        students[name]?.let { scores ->
            scores.add(score)
            
            // 스코프 함수를 사용한 평균 계산
            val average = scores.average()
            val grade = when {
                average >= 90 -> "A"
                average >= 80 -> "B"
                average >= 70 -> "C"
                else -> "F"
            }
            
            println("$name의 새 점수: $score, 평균: $average, 학점: $grade")
        }
    }
    
    fun getTopStudent(): String? {
        var topStudent: String? = null
        var topAverage = 0.0
        
        students.forEach { (name, scores) ->
            if (scores.isNotEmpty()) {
                val average = scores.average()
                if (average > topAverage) {
                    topAverage = average
                    topStudent = name
                }
            }
        }
        
        return topStudent
    }
}

fun main() {
    val manager = GradeManager()
    
    manager.addStudent("홍길동")
    manager.addStudent("김철수")
    
    manager.addScore("홍길동", 95)
    manager.addScore("홍길동", 87)
    manager.addScore("김철수", 92)
    
    val topStudent = manager.getTopStudent()
    println("최고 성적 학생: $topStudent")
}
```

## 정리

#### 스코프 규칙

1. **내부 스코프**에서 **외부 스코프** 변수에 접근 가능
2. **외부 스코프**에서 **내부 스코프** 변수에 접근 불가
3. **같은 이름**의 변수가 있으면 **가까운 스코프**가 우선
4. **람다**는 외부 변수를 **캡처**할 수 있음

#### 스코프 활용 팁

- **val**을 우선 사용하여 **변수 범위 최소화**
- **적절한 접근 제한자**로 **클래스 스코프 관리**
- **스코프 함수**로 **간결한 코드 작성**
- **변수 이름 충돌** 방지를 위한 **명확한 네이밍**

스코프를 잘 이해하면 **버그를 줄이고** **코드를 더 안전하게** 작성할 수 있습니다.