---
layout: single
title: Kotlin_Jetpack_Compose2
date: 2025-07-17
category: kotlin
---
  

## 🎯 Best Practices 요약

#### 💡 성능 최적화
- **remember** 사용으로 불필요한 재계산 방지
- **LazyColumn/LazyRow** 사용으로 대용량 리스트 최적화
- **key** 파라미터 사용으로 recomposition 최적화

#### 🔧 코드 품질
- **Composable 함수 분리**로 재사용성 향상
- **State hoisting**으로 상태 관리 최적화
- **testTag** 사용으로 테스트 가능한 UI 구성

#### 🌟 사용자 경험
- **Loading states**로 사용자 피드백 제공
- **Error handling**으로 오류 상황 대응
- **Accessibility**를 고려한 UI 설계


#### 목차

🌐 **네트워크 통신과 데이터 처리**

🎨 **테마와 스타일링**

🚀 **성능 Recomposition 최적화**

🧪 **테스트**

💡 **코드** **구조화**


## 🌐 네트워크 통신과 데이터 처리

#### 🔗 HTTP 통신
**HTTP 통신**은 서버와 데이터를 주고받는 방식입니다. **Axios**나 **Fetch API**와 유사한 개념입니다.

```kotlin
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.Refresh
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp
import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import androidx.lifecycle.viewmodel.compose.viewModel
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.launch
import java.net.HttpURLConnection
import java.net.URL

/**
 * 사용자 데이터 클래스
 */
data class User(
    val id: Int,
    val name: String,
    val email: String,
    val phone: String
)

/**
 * UI 상태 관리
 * Redux의 State와 유사한 개념
 */
sealed class UiState {
    object Loading : UiState()
    data class Success(val users: List<User>) : UiState()
    data class Error(val message: String) : UiState()
}

/**
 * 네트워크 통신 ViewModel
 * React의 Custom Hook이나 Redux Thunk와 유사
 */
class NetworkViewModel : ViewModel() {
    // StateFlow는 React의 state와 유사
    private val _uiState = MutableStateFlow<UiState>(UiState.Loading)
    val uiState: StateFlow<UiState> = _uiState.asStateFlow()
    
    init {
        // 초기 데이터 로드
        loadUsers()
    }
    
    /**
     * 사용자 데이터 로드
     * Axios나 Fetch API와 유사한 기능
     */
    fun loadUsers() {
        viewModelScope.launch {
            _uiState.value = UiState.Loading
            
            try {
                // 실제 프로젝트에서는 Retrofit이나 Ktor 사용
                val users = fetchUsersFromApi()
                _uiState.value = UiState.Success(users)
            } catch (e: Exception) {
                _uiState.value = UiState.Error("데이터 로드 실패: ${e.message}")
            }
        }
    }
    
    /**
     * 모의 API 호출
     * 실제로는 Retrofit이나 Ktor 사용
     */
    private suspend fun fetchUsersFromApi(): List<User> {
        // 네트워크 지연 시뮬레이션
        kotlinx.coroutines.delay(2000)
        
        // 모의 데이터 반환
        return listOf(
            User(1, "홍길동", "hong@example.com", "010-1234-5678"),
            User(2, "김철수", "kim@example.com", "010-2345-6789"),
            User(3, "이영희", "lee@example.com", "010-3456-7890"),
            User(4, "박지민", "park@example.com", "010-4567-8901"),
            User(5, "정수빈", "jung@example.com", "010-5678-9012")
        )
    }
}

/**
 * 네트워크 통신 화면
 * React의 App Component와 유사
 */
@Composable
fun NetworkScreen() {
    val viewModel: NetworkViewModel = viewModel()
    val uiState by viewModel.uiState.collectAsState()
    
    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp)
    ) {
        // 헤더
        Row(
            modifier = Modifier.fillMaxWidth(),
            horizontalArrangement = Arrangement.SpaceBetween,
            verticalAlignment = Alignment.CenterVertically
        ) {
            Text(
                text = "👥 사용자 목록",
                style = MaterialTheme.typography.headlineLarge
            )
            
            IconButton(
                onClick = { viewModel.loadUsers() }
            ) {
                Icon(
                    Icons.Default.Refresh,
                    contentDescription = "새로고침"
                )
            }
        }
        
        Spacer(modifier = Modifier.height(16.dp))
        
        // UI 상태에 따른 화면 표시
        // React의 conditional rendering과 유사
        when (uiState) {
            is UiState.Loading -> {
                // 로딩 상태
                Box(
                    modifier = Modifier.fillMaxSize(),
                    contentAlignment = Alignment.Center
                ) {
                    Column(
                        horizontalAlignment = Alignment.CenterHorizontally
                    ) {
                        CircularProgressIndicator()
                        Spacer(modifier = Modifier.height(16.dp))
                        Text("데이터를 불러오는 중...")
                    }
                }
            }
            
            is UiState.Success -> {
                // 성공 상태
                LazyColumn(
                    verticalArrangement = Arrangement.spacedBy(8.dp)
                ) {
                    items(uiState.users) { user ->
                        UserCard(user = user)
                    }
                }
            }
            
            is UiState.Error -> {
                // 에러 상태
                Card(
                    modifier = Modifier.fillMaxWidth(),
                    colors = CardDefaults.cardColors(
                        containerColor = MaterialTheme.colorScheme.errorContainer
                    )
                ) {
                    Column(
                        modifier = Modifier.padding(16.dp)
                    ) {
                        Text(
                            text = "❌ 오류 발생",
                            style = MaterialTheme.typography.headlineMedium,
                            color = MaterialTheme.colorScheme.onErrorContainer
                        )
                        
                        Spacer(modifier = Modifier.height(8.dp))
                        
                        Text(
                            text = uiState.message,
                            color = MaterialTheme.colorScheme.onErrorContainer
                        )
                        
                        Spacer(modifier = Modifier.height(16.dp))
                        
                        Button(
                            onClick = { viewModel.loadUsers() }
                        ) {
                            Text("다시 시도")
                        }
                    }
                }
            }
        }
    }
}

/**
 * 사용자 카드 컴포넌트
 * React의 UserCard Component와 유사
 */
@Composable
fun UserCard(user: User) {
    Card(
        modifier = Modifier.fillMaxWidth(),
        elevation = CardDefaults.cardElevation(4.dp)
    ) {
        Column(
            modifier = Modifier.padding(16.dp)
        ) {
            Text(
                text = user.name,
                style = MaterialTheme.typography.headlineSmall
            )
            
            Spacer(modifier = Modifier.height(4.dp))
            
            Text(
                text = "📧 ${user.email}",
                color = MaterialTheme.colorScheme.onSurfaceVariant
            )
            
            Text(
                text = "📱 ${user.phone}",
                color = MaterialTheme.colorScheme.onSurfaceVariant
            )
        }
    }
}
```

## 🎨 테마와 스타일링

#### 🎭 Material Design 테마
**Material Design**은 Google의 design language입니다. **Bootstrap**이나 **Ant Design**과 유사한 design system입니다.

```kotlin
import androidx.compose.foundation.isSystemInDarkTheme
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.text.font.FontFamily
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.unit.dp
import androidx.compose.ui.unit.sp

/**
 * 커스텀 색상 팔레트
 * CSS Variables나 SCSS Variables와 유사
 */
private val LightColorScheme = lightColorScheme(
    primary = Color(0xFF1976D2),
    onPrimary = Color.White,
    primaryContainer = Color(0xFFBBDEFB),
    onPrimaryContainer = Color(0xFF0D47A1),
    secondary = Color(0xFF03DAC6),
    onSecondary = Color.Black,
    tertiary = Color(0xFF018786),
    background = Color(0xFFFFFBFE),
    surface = Color(0xFFFFFBFE),
    onBackground = Color(0xFF1C1B1F),
    onSurface = Color(0xFF1C1B1F),
    error = Color(0xFFBA1A1A),
    onError = Color.White
)

private val DarkColorScheme = darkColorScheme(
    primary = Color(0xFF90CAF9),
    onPrimary = Color(0xFF0D47A1),
    primaryContainer = Color(0xFF1565C0),
    onPrimaryContainer = Color(0xFFE3F2FD),
    secondary = Color(0xFF03DAC6),
    onSecondary = Color.Black,
    tertiary = Color(0xFF66FFF9),
    background = Color(0xFF1C1B1F),
    surface = Color(0xFF1C1B1F),
    onBackground = Color(0xFFE6E1E5),
    onSurface = Color(0xFFE6E1E5),
    error = Color(0xFFFFB4AB),
    onError = Color(0xFF690005)
)

/**
 * 커스텀 타이포그래피
 * CSS Font Family와 유사
 */
private val CustomTypography = Typography(
    displayLarge = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Normal,
        fontSize = 57.sp,
        lineHeight = 64.sp,
        letterSpacing = (-0.25).sp
    ),
    headlineLarge = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Bold,
        fontSize = 32.sp,
        lineHeight = 40.sp,
        letterSpacing = 0.sp
    ),
    titleLarge = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Medium,
        fontSize = 22.sp,
        lineHeight = 28.sp,
        letterSpacing = 0.sp
    ),
    bodyLarge = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Normal,
        fontSize = 16.sp,
        lineHeight = 24.sp,
        letterSpacing = 0.5.sp
    ),
    labelLarge = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Medium,
        fontSize = 14.sp,
        lineHeight = 20.sp,
        letterSpacing = 0.1.sp
    )
)

/**
 * 커스텀 Shapes
 * CSS border-radius와 유사
 */
private val CustomShapes = Shapes(
    small = RoundedCornerShape(4.dp),
    medium = RoundedCornerShape(8.dp),
    large = RoundedCornerShape(16.dp)
)

/**
 * 앱 테마 설정
 * CSS Global Styles나 styled-components의 ThemeProvider와 유사
 */
@Composable
fun CustomTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    content: @Composable () -> Unit
) {
    val colorScheme = if (darkTheme) {
        DarkColorScheme
    } else {
        LightColorScheme
    }
    
    MaterialTheme(
        colorScheme = colorScheme,
        typography = CustomTypography,
        shapes = CustomShapes,
        content = content
    )
}

/**
 * 테마 데모 화면
 * 다양한 Material Design 컴포넌트 스타일링 예제
 */
@Composable
fun ThemeDemo() {
    var isDarkMode by remember { mutableStateOf(false) }
    
    CustomTheme(darkTheme = isDarkMode) {
        Column(
            modifier = Modifier
                .fillMaxSize()
                .padding(16.dp),
            verticalArrangement = Arrangement.spacedBy(16.dp)
        ) {
            // 테마 토글 헤더
            Row(
                modifier = Modifier.fillMaxWidth(),
                horizontalArrangement = Arrangement.SpaceBetween,
                verticalAlignment = Alignment.CenterVertically
            ) {
                Text(
                    text = "🎨 테마 예제",
                    style = MaterialTheme.typography.headlineLarge
                )
                
                Row(
                    verticalAlignment = Alignment.CenterVertically,
                    horizontalArrangement = Arrangement.spacedBy(8.dp)
                ) {
                    Text("🌞")
                    Switch(
                        checked = isDarkMode,
                        onCheckedChange = { isDarkMode = it }
                    )
                    Text("🌙")
                }
            }
            
            // 색상 팔레트 표시
            Card(
                modifier = Modifier.fillMaxWidth(),
                elevation = CardDefaults.cardElevation(4.dp)
            ) {
                Column(
                    modifier = Modifier.padding(16.dp)
                ) {
                    Text(
                        text = "색상 팔레트",
                        style = MaterialTheme.typography.titleLarge
                    )
                    
                    Spacer(modifier = Modifier.height(12.dp))
                    
                    // Primary 색상
                    ColorSwatch(
                        color = MaterialTheme.colorScheme.primary,
                        name = "Primary",
                        textColor = MaterialTheme.colorScheme.onPrimary
                    )
                    
                    Spacer(modifier = Modifier.height(8.dp))
                    
                    // Secondary 색상
                    ColorSwatch(
                        color = MaterialTheme.colorScheme.secondary,
                        name = "Secondary",
                        textColor = MaterialTheme.colorScheme.onSecondary
                    )
                    
                    Spacer(modifier = Modifier.height(8.dp))
                    
                    // Surface 색상
                    ColorSwatch(
                        color = MaterialTheme.colorScheme.surface,
                        name = "Surface",
                        textColor = MaterialTheme.colorScheme.onSurface
                    )
                }
            }
            
            // 타이포그래피 예제
            Card(
                modifier = Modifier.fillMaxWidth(),
                elevation = CardDefaults.cardElevation(4.dp)
            ) {
                Column(
                    modifier = Modifier.padding(16.dp)
                ) {
                    Text(
                        text = "타이포그래피",
                        style = MaterialTheme.typography.titleLarge
                    )
                    
                    Spacer(modifier = Modifier.height(12.dp))
                    
                    Text(
                        text = "Display Large",
                        style = MaterialTheme.typography.displayLarge
                    )
                    
                    Text(
                        text = "Headline Large",
                        style = MaterialTheme.typography.headlineLarge
                    )
                    
                    Text(
                        text = "Title Large",
                        style = MaterialTheme.typography.titleLarge
                    )
                    
                    Text(
                        text = "Body Large - 본문 텍스트입니다.",
                        style = MaterialTheme.typography.bodyLarge
                    )
                    
                    Text(
                        text = "Label Large",
                        style = MaterialTheme.typography.labelLarge
                    )
                }
            }
            
            // 컴포넌트 스타일링 예제
            Card(
                modifier = Modifier.fillMaxWidth(),
                elevation = CardDefaults.cardElevation(4.dp)
            ) {
                Column(
                    modifier = Modifier.padding(16.dp),
                    verticalArrangement = Arrangement.spacedBy(12.dp)
                ) {
                    Text(
                        text = "컴포넌트 스타일링",
                        style = MaterialTheme.typography.titleLarge
                    )
                    
                    // 다양한 버튼 스타일
                    Row(
                        horizontalArrangement = Arrangement.spacedBy(8.dp)
                    ) {
                        Button(
                            onClick = { /* 액션 */ }
                        ) {
                            Text("Primary")
                        }
                        
                        OutlinedButton(
                            onClick = { /* 액션 */ }
                        ) {
                            Text("Outlined")
                        }
                        
                        TextButton(
                            onClick = { /* 액션 */ }
                        ) {
                            Text("Text")
                        }
                    }
                    
                    // 커스텀 스타일 버튼
                    Button(
                        onClick = { /* 액션 */ },
                        colors = ButtonDefaults.buttonColors(
                            containerColor = MaterialTheme.colorScheme.tertiary,
                            contentColor = MaterialTheme.colorScheme.onTertiary
                        ),
                        shape = MaterialTheme.shapes.large
                    ) {
                        Text("커스텀 버튼")
                    }
                    
                    // 텍스트 필드
                    var textValue by remember { mutableStateOf("") }
                    TextField(
                        value = textValue,
                        onValueChange = { textValue = it },
                        label = { Text("테마 적용된 텍스트 필드") },
                        modifier = Modifier.fillMaxWidth()
                    )
                }
            }
        }
    }
}

/**
 * 색상 견본 컴포넌트
 * CSS Color Palette와 유사
 */
@Composable
fun ColorSwatch(
    color: Color,
    name: String,
    textColor: Color
) {
    Row(
        modifier = Modifier
            .fillMaxWidth()
            .height(48.dp)
            .padding(horizontal = 4.dp),
        verticalAlignment = Alignment.CenterVertically
    ) {
        Box(
            modifier = Modifier
                .size(40.dp)
                .background(color, MaterialTheme.shapes.small)
        )
        
        Spacer(modifier = Modifier.width(12.dp))
        
        Text(
            text = name,
            style = MaterialTheme.typography.bodyLarge,
            color = textColor
        )
    }
}
```

## 🚀 성능 최적화

#### ⚡ Recomposition 최적화
**Recomposition**은 필요한 부분만 다시 그려야 성능이 좋습니다. **React의 리렌더링 최적화**와 동일한 개념입니다.

```kotlin
import androidx.compose.foundation.background
import androidx.compose.foundation.clickable
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.unit.dp

/**
 * 성능 최적화 예제
 * React의 useMemo, useCallback, React.memo와 유사한 최적화 기법
 */
@Composable
fun PerformanceExample() {
    var counter by remember { mutableStateOf(0) }
    var expensiveValue by remember { mutableStateOf(0) }
    
    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp),
        verticalArrangement = Arrangement.spacedBy(16.dp)
    ) {
        Text(
            text = "⚡ 성능 최적화 예제",
            style = MaterialTheme.typography.headlineLarge
        )
        
        // 1. remember를 사용한 값 캐싱
        // React의 useMemo와 동일한 개념
        val expensiveCalculation = remember(expensiveValue) {
            // expensiveValue가 변경될 때만 계산
            println("🔄 비용이 많이 드는 계산 실행")
            (1..expensiveValue).map { it * it }.sum()
        }
        
        Card(
            modifier = Modifier.fillMaxWidth(),
            elevation = CardDefaults.cardElevation(4.dp)
        ) {
            Column(
                modifier = Modifier.padding(16.dp)
            ) {
                Text(
                    text = "💰 비용이 많이 드는 계산",
                    style = MaterialTheme.typography.titleLarge
                )
                
                Text("입력값: $expensiveValue")
                Text("계산 결과: $expensiveCalculation")
                
                Spacer(modifier = Modifier.height(8.dp))
                
                Button(
                    onClick = { expensiveValue++ }
                ) {
                    Text("값 증가")
                }
            }
        }
        
        // 2. 분리된 컴포넌트로 recomposition 최적화
        // React의 컴포넌트 분리와 동일한 개념
        CounterSection(
            counter = counter,
            onIncrement = { counter++ },
            onDecrement = { counter-- }
        )
        
        // 3. LazyColumn으로 대용량 리스트 최적화
        // React의 Virtual List와 동일한 개념
        LargeListSection()
    }
}

/**
 * 분리된 카운터 컴포넌트
 * React의 memo()와 유사한 최적화 효과
 */
@Composable
fun CounterSection(
    counter: Int,
    onIncrement: () -> Unit,
    onDecrement: () -> Unit
) {
    // 이 컴포넌트는 counter 관련 값이 변경될 때만 recomposition
    Card(
        modifier = Modifier.fillMaxWidth(),
        elevation = CardDefaults.cardElevation(4.dp)
    ) {
        Column(
            modifier = Modifier.padding(16.dp),
            horizontalAlignment = Alignment.CenterHorizontally
        ) {
            Text(
                text = "📊 카운터 섹션",
                style = MaterialTheme.typography.titleLarge
            )
            
            Text(
                text = "Count: $counter",
                style = MaterialTheme.typography.headlineMedium
            )
            
            Spacer(modifier = Modifier.height(16.dp))
            
            Row(
                horizontalArrangement = Arrangement.spacedBy(8.dp)
            ) {
                Button(onClick = onDecrement) {
                    Text("-")
                }
                
                Button(onClick = onIncrement) {
                    Text("+")
                }
            }
        }
    }
}

/**
 * 대용량 리스트 섹션
 * React의 Virtual List와 동일한 개념
 */
@Composable
fun LargeListSection() {
    var items by remember { mutableStateOf(generateLargeList()) }
    
    Card(
        modifier = Modifier.fillMaxWidth(),
        elevation = CardDefaults.cardElevation(4.dp)
    ) {
        Column(
            modifier = Modifier.padding(16.dp)
        ) {
            Text(
                text = "📋 대용량 리스트 (${items.size}개)",
                style = MaterialTheme.typography.titleLarge
            )
            
            Spacer(modifier = Modifier.height(8.dp))
            
            Button(
                onClick = { items = generateLargeList() }
            ) {
                Text("목록 새로고침")
            }
            
            Spacer(modifier = Modifier.height(8.dp))
            
            // LazyColumn은 보이는 항목만 렌더링
            // React의 react-window나 react-virtualized와 동일
            LazyColumn(
                modifier = Modifier.height(200.dp),
                verticalArrangement = Arrangement.spacedBy(4.dp)
            ) {
                items(items) { item ->
                    // 각 항목은 독립적으로 recomposition
                    ListItem(item = item)
                }
            }
        }
    }
}

/**
 * 리스트 항목 컴포넌트
 * React의 memo()와 유사한 최적화 효과
 */
@Composable
fun ListItem(item: String) {
    var isSelected by remember { mutableStateOf(false) }
    
    Row(
        modifier = Modifier
            .fillMaxWidth()
            .clickable { isSelected = !isSelected }
            .background(
                if (isSelected) Color.LightGray else Color.Transparent,
                MaterialTheme.shapes.small
            )
            .padding(8.dp),
        verticalAlignment = Alignment.CenterVertically
    ) {
        Text(
            text = item,
            modifier = Modifier.weight(1f)
        )
        
        if (isSelected) {
            Text(
                text = "✓",
                color = MaterialTheme.colorScheme.primary
            )
        }
    }
}

/**
 * 대용량 리스트 생성 함수
 */
fun generateLargeList(): List<String> {
    return (1..1000).map { "Item $it" }
}
```

## 🧪 테스트

#### 🔍 UI 테스트
**UI 테스트**는 사용자 인터페이스가 올바르게 작동하는지 확인합니다. 
**React Testing Library**나 **Cypress**와 유사한 개념입니다.

```kotlin
import androidx.compose.foundation.layout.*
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.platform.testTag
import androidx.compose.ui.semantics.contentDescription
import androidx.compose.ui.semantics.semantics
import androidx.compose.ui.unit.dp

/**
 * 테스트 가능한 컴포넌트 예제
 * React Testing Library의 테스트 패턴과 유사
 */
@Composable
fun TestableComponent() {
    var count by remember { mutableStateOf(0) }
    var text by remember { mutableStateOf("") }
    
    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp)
            .semantics { 
                contentDescription = "메인 화면" // 접근성을 위한 설명
            },
        verticalArrangement = Arrangement.spacedBy(16.dp),
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Text(
            text = "🧪 테스트 예제",
            style = MaterialTheme.typography.headlineLarge
        )
        
        // 카운터 섹션
        Card(
            modifier = Modifier
                .fillMaxWidth()
                .testTag("counter-card"), // 테스트 ID
            elevation = CardDefaults.cardElevation(4.dp)
        ) {
            Column(
                modifier = Modifier.padding(16.dp),
                horizontalAlignment = Alignment.CenterHorizontally
            ) {
                Text(
                    text = "Count: $count",
                    modifier = Modifier.testTag("counter-text"), // 테스트 ID
                    style = MaterialTheme.typography.headlineMedium
                )
                
                Spacer(modifier = Modifier.height(16.dp))
                
                Row(
                    horizontalArrangement = Arrangement.spacedBy(8.dp)
                ) {
                    Button(
                        onClick = { count-- },
                        modifier = Modifier.testTag("decrement-button") // 테스트 ID
                    ) {
                        Text("-")
                    }
                    
                    Button(
                        onClick = { count++ },
                        modifier = Modifier.testTag("increment-button") // 테스트 ID
                    ) {
                        Text("+")
                    }
                }
                
                Spacer(modifier = Modifier.height(16.dp))
                
                Button(
                    onClick = { count = 0 },
                    modifier = Modifier.testTag("reset-button") // 테스트 ID
                ) {
                    Text("Reset")
                }
            }
        }
        
        // 텍스트 입력 섹션
        Card(
            modifier = Modifier
                .fillMaxWidth()
                .testTag("text-input-card"), // 테스트 ID
            elevation = CardDefaults.cardElevation(4.dp)
        ) {
            Column(
                modifier = Modifier.padding(16.dp)
            ) {
                TextField(
                    value = text,
                    onValueChange = { text = it },
                    label = { Text("텍스트 입력") },
                    modifier = Modifier
                        .fillMaxWidth()
                        .testTag("text-input"), // 테스트 ID
                    placeholder = { Text("여기에 입력하세요") }
                )
                
                Spacer(modifier = Modifier.height(16.dp))
                
                if (text.isNotBlank()) {
                    Text(
                        text = "입력된 텍스트: $text",
                        modifier = Modifier.testTag("display-text"), // 테스트 ID
                        style = MaterialTheme.typography.bodyLarge
                    )
                }
                
                Spacer(modifier = Modifier.height(16.dp))
                
                Button(
                    onClick = { text = "" },
                    modifier = Modifier.testTag("clear-text-button"), // 테스트 ID
                    enabled = text.isNotBlank()
                ) {
                    Text("Clear")
                }
            }
        }
        
        // 조건부 렌더링 테스트
        if (count > 5) {
            Card(
                modifier = Modifier
                    .fillMaxWidth()
                    .testTag("achievement-card"), // 테스트 ID
                colors = CardDefaults.cardColors(
                    containerColor = MaterialTheme.colorScheme.primaryContainer
                )
            ) {
                Text(
                    text = "🎉 5를 넘었습니다!",
                    modifier = Modifier
                        .padding(16.dp)
                        .testTag("achievement-text"), // 테스트 ID
                    style = MaterialTheme.typography.titleLarge
                )
            }
        }
    }
}

/**
 * 테스트 예제 (실제 테스트 파일에 작성)
 * React Testing Library의 테스트 패턴과 유사
 */

@Test
fun testCounterIncrement() {
    // React Testing Library의 render()와 유사
    composeTestRule.setContent {
        TestableComponent()
    }
    
    // 초기 상태 확인 - getByTestId()와 유사
    composeTestRule
        .onNodeWithTag("counter-text")
        .assertTextEquals("Count: 0")
    
    // 버튼 클릭 - fireEvent.click()와 유사
    composeTestRule
        .onNodeWithTag("increment-button")
        .performClick()
    
    // 상태 변경 확인 - expect()와 유사
    composeTestRule
        .onNodeWithTag("counter-text")
        .assertTextEquals("Count: 1")
}

@Test
fun testTextInput() {
    composeTestRule.setContent {
        TestableComponent()
    }
    
    // 텍스트 입력 - fireEvent.change()와 유사
    composeTestRule
        .onNodeWithTag("text-input")
        .performTextInput("Hello, World!")
    
    // 입력 결과 확인
    composeTestRule
        .onNodeWithTag("display-text")
        .assertTextEquals("입력된 텍스트: Hello, World!")
    
    // 클리어 버튼 클릭
    composeTestRule
        .onNodeWithTag("clear-text-button")
        .performClick()
    
    // 텍스트가 지워졌는지 확인
    composeTestRule
        .onNodeWithTag("display-text")
        .assertDoesNotExist()
}

@Test
fun testConditionalRendering() {
    composeTestRule.setContent {
        TestableComponent()
    }
    
    // 초기에는 achievement-card가 없음
    composeTestRule
        .onNodeWithTag("achievement-card")
        .assertDoesNotExist()
    
    // 6번 클릭하여 5를 넘김
    repeat(6) {
        composeTestRule
            .onNodeWithTag("increment-button")
            .performClick()
    }
    
    // achievement-card가 나타났는지 확인
    composeTestRule
        .onNodeWithTag("achievement-card")
        .assertExists()
    
    composeTestRule
        .onNodeWithTag("achievement-text")
        .assertTextEquals("🎉 5를 넘었습니다!")
}

```

## 🔧 실무 Best Practices

#### 💡 코드 구조화
**코드 구조화**는 유지보수성과 확장성을 높이는 핵심입니다.

```kotlin
/**
 * 실무 프로젝트 구조 예제
 * 
 * 📁 app/
 * ├── 📁 src/main/java/com/example/app/
 * │   ├── 📁 ui/
 * │   │   ├── 📁 components/     # 재사용 가능한 컴포넌트
 * │   │   ├── 📁 screens/        # 화면별 컴포넌트
 * │   │   ├── 📁 theme/          # 테마 관련 파일
 * │   │   └── 📁 navigation/     # 네비게이션 관련
 * │   ├── 📁 data/
 * │   │   ├── 📁 repository/     # 데이터 저장소
 * │   │   ├── 📁 network/        # 네트워크 관련
 * │   │   └── 📁 local/          # 로컬 데이터베이스
 * │   ├── 📁 domain/
 * │   │   ├── 📁 model/          # 데이터 모델
 * │   │   ├── 📁 usecase/        # 비즈니스 로직
 * │   │   └── 📁 repository/     # 저장소 인터페이스
 * │   └── 📁 presentation/
 * │       ├── 📁 viewmodel/      # ViewModel
 * │       └── 📁 state/          # UI State
 */

/**
 * 재사용 가능한 컴포넌트 예제
 * React의 공통 컴포넌트와 동일한 개념
 */
@Composable
fun CommonButton(
    text: String,
    onClick: () -> Unit,
    modifier: Modifier = Modifier,
    enabled: Boolean = true,
    variant: ButtonVariant = ButtonVariant.Primary
) {
    val colors = when (variant) {
        ButtonVariant.Primary -> ButtonDefaults.buttonColors()
        ButtonVariant.Secondary -> ButtonDefaults.outlinedButtonColors()
        ButtonVariant.Danger -> ButtonDefaults.buttonColors(
            containerColor = MaterialTheme.colorScheme.error
        )
    }
    
    Button(
        onClick = onClick,
        modifier = modifier,
        enabled = enabled,
        colors = colors
    ) {
        Text(text)
    }
}

enum class ButtonVariant {
    Primary, Secondary, Danger
}

/**
 * 공통 로딩 컴포넌트
 * React의 Loading Component와 동일
 */
@Composable
fun LoadingSpinner(
    modifier: Modifier = Modifier,
    message: String = "로딩 중..."
) {
    Box(
        modifier = modifier.fillMaxSize(),
        contentAlignment = Alignment.Center
    ) {
        Column(
            horizontalAlignment = Alignment.CenterHorizontally
        ) {
            CircularProgressIndicator(
                color = MaterialTheme.colorScheme.primary
            )
            
            Spacer(modifier = Modifier.height(16.dp))
            
            Text(
                text = message,
                style = MaterialTheme.typography.bodyLarge,
                color = MaterialTheme.colorScheme.onSurface
            )
        }
    }
}

/**
 * 에러 표시 컴포넌트
 * React의 Error Component와 동일
 */
@Composable
fun ErrorDisplay(
    message: String,
    onRetry: () -> Unit,
    modifier: Modifier = Modifier
) {
    Card(
        modifier = modifier.fillMaxWidth(),
        colors = CardDefaults.cardColors(
            containerColor = MaterialTheme.colorScheme.errorContainer
        )
    ) {
        Column(
            modifier = Modifier.padding(16.dp),
            horizontalAlignment = Alignment.CenterHorizontally
        ) {
            Text(
                text = "❌ 오류 발생",
                style = MaterialTheme.typography.headlineSmall,
                color = MaterialTheme.colorScheme.onErrorContainer
            )
            
            Spacer(modifier = Modifier.height(8.dp))
            
            Text(
                text = message,
                style = MaterialTheme.typography.bodyMedium,
                color = MaterialTheme.colorScheme.onErrorContainer
            )
            
            Spacer(modifier = Modifier.height(16.dp))
            
            CommonButton(
                text = "다시 시도",
                onClick = onRetry,
                variant = ButtonVariant.Primary
            )
        }
    }
}
```

## 📱 프로젝트 예제

#### 🛒 쇼핑몰 앱 예제

```kotlin
/**
 * 완전한 쇼핑몰 앱 예제
 * 실무에서 사용되는 모든 패턴을 포함
 */

// 데이터 모델 - TypeScript의 interface와 유사
data class Product(
    val id: Int,
    val name: String,
    val price: Double,
    val imageUrl: String,
    val description: String,
    val category: String,
    val rating: Float = 0f,
    val reviewCount: Int = 0
)

data class CartItem(
    val product: Product,
    val quantity: Int
)

// UI 상태 관리 - Redux의 State와 유사
sealed class ShoppingUiState {
    object Loading : ShoppingUiState()
    data class Success(
        val products: List<Product>,
        val cartItems: List<CartItem> = emptyList(),
        val selectedCategory: String = "All"
    ) : ShoppingUiState()
    data class Error(val message: String) : ShoppingUiState()
}

// ViewModel - React의 Custom Hook이나 Redux Store와 유사
class ShoppingViewModel : ViewModel() {
    // StateFlow는 React의 state와 유사
    private val _uiState = MutableStateFlow<ShoppingUiState>(ShoppingUiState.Loading)
    val uiState: StateFlow<ShoppingUiState> = _uiState.asStateFlow()
    
    init {
        loadProducts() // 컴포넌트 마운트 시 자동 실행
    }
    
    // 상품 로드 - async/await 패턴과 유사
    private fun loadProducts() {
        viewModelScope.launch {
            _uiState.value = ShoppingUiState.Loading
            
            try {
                delay(1000) // 네트워크 지연 시뮬레이션
                
                val products = generateSampleProducts()
                _uiState.value = ShoppingUiState.Success(products)
            } catch (e: Exception) {
                _uiState.value = ShoppingUiState.Error("상품을 불러오는데 실패했습니다")
            }
        }
    }
    
    // 장바구니 추가 - Redux의 action과 유사
    fun addToCart(product: Product) {
        val currentState = _uiState.value
        if (currentState is ShoppingUiState.Success) {
            val existingItem = currentState.cartItems.find { it.product.id == product.id }
            val updatedCart = if (existingItem != null) {
                // 기존 아이템 수량 증가
                currentState.cartItems.map {
                    if (it.product.id == product.id) {
                        it.copy(quantity = it.quantity + 1)
                    } else it
                }
            } else {
                // 새 아이템 추가
                currentState.cartItems + CartItem(product, 1)
            }
            
            _uiState.value = currentState.copy(cartItems = updatedCart)
        }
    }
    
    // 카테고리 필터 - Redux의 action과 유사
    fun filterByCategory(category: String) {
        val currentState = _uiState.value
        if (currentState is ShoppingUiState.Success) {
            _uiState.value = currentState.copy(selectedCategory = category)
        }
    }
    
    // 모의 데이터 생성 - 실제로는 API 호출
    private fun generateSampleProducts(): List<Product> {
        return listOf(
            Product(1, "스마트폰", 899.99, "", "최신 스마트폰", "Electronics", 4.5f, 123),
            Product(2, "노트북", 1299.99, "", "고성능 노트북", "Electronics", 4.3f, 89),
            Product(3, "운동화", 129.99, "", "편안한 운동화", "Fashion", 4.7f, 234),
            Product(4, "책", 19.99, "", "베스트셀러 도서", "Books", 4.2f, 45),
            Product(5, "커피", 12.99, "", "원두 커피", "Food", 4.6f, 167)
        )
    }
}

// 메인 앱 컴포넌트
@Composable
fun ShoppingApp() {
    val viewModel: ShoppingViewModel = viewModel() // React의 useContext와 유사
    val uiState by viewModel.uiState.collectAsState() // React의 useSelector와 유사
    
    Box(
        modifier = Modifier.fillMaxSize()
    ) {
        // 상태에 따른 조건부 렌더링 - React의 conditional rendering과 동일
        when (uiState) {
            is ShoppingUiState.Loading -> {
                LoadingSpinner(message = "상품을 불러오는 중...")
            }
            
            is ShoppingUiState.Success -> {
                ShoppingContent(
                    products = uiState.products,
                    cartItems = uiState.cartItems,
                    selectedCategory = uiState.selectedCategory,
                    onAddToCart = viewModel::addToCart, // React의 callback props와 유사
                    onCategoryChange = viewModel::filterByCategory
                )
            }
            
            is ShoppingUiState.Error -> {
                ErrorDisplay(
                    message = uiState.message,
                    onRetry = { viewModel.loadProducts() }
                )
            }
        }
    }
}

// 메인 콘텐츠 컴포넌트
@Composable
fun ShoppingContent(
    products: List<Product>,
    cartItems: List<CartItem>,
    selectedCategory: String,
    onAddToCart: (Product) -> Unit,
    onCategoryChange: (String) -> Unit
) {
    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp)
    ) {
        // 헤더 컴포넌트
        ShoppingHeader(
            cartItemCount = cartItems.sumOf { it.quantity }
        )
        
        Spacer(modifier = Modifier.height(16.dp))
        
        // 카테고리 필터 컴포넌트
        CategoryFilter(
            selectedCategory = selectedCategory,
            onCategoryChange = onCategoryChange
        )
        
        Spacer(modifier = Modifier.height(16.dp))
        
        // 상품 목록 컴포넌트
        ProductList(
            products = products.filter { 
                selectedCategory == "All" || it.category == selectedCategory 
            },
            onAddToCart = onAddToCart
        )
    }
}

// 헤더 컴포넌트
@Composable
fun ShoppingHeader(cartItemCount: Int) {
    Row(
        modifier = Modifier.fillMaxWidth(),
        horizontalArrangement = Arrangement.SpaceBetween,
        verticalAlignment = Alignment.CenterVertically
    ) {
        Text(
            text = "🛍️ 쇼핑몰",
            style = MaterialTheme.typography.headlineLarge
        )
        
        // 장바구니 배지 - Material UI의 Badge와 유사
        Badge(
            modifier = Modifier.testTag("cart-badge")
        ) {
            Text("$cartItemCount")
        }
    }
}

// 카테고리 필터 컴포넌트 
@Composable
fun CategoryFilter(
    selectedCategory: String,
    onCategoryChange: (String) -> Unit
) {
    val categories = listOf("All", "Electronics", "Fashion", "Books", "Food")
    
    // 가로 스크롤 리스트 - React의 horizontal scroll과 유사
    LazyRow(
        horizontalArrangement = Arrangement.spacedBy(8.dp)
    ) {
        items(categories) { category ->
            FilterChip(
                selected = selectedCategory == category,
                onClick = { onCategoryChange(category) },
                label = { Text(category) }
            )
        }
    }
}

// 상품 목록 컴포넌트
@Composable
fun ProductList(
    products: List<Product>,
    onAddToCart: (Product) -> Unit
) {
    // 세로 스크롤 리스트 - React의 VirtualizedList와 유사
    LazyColumn(
        verticalArrangement = Arrangement.spacedBy(8.dp)
    ) {
        items(products) { product ->
            ProductCard(
                product = product,
                onAddToCart = { onAddToCart(product) }
            )
        }
    }
}

// 상품 카드 컴포넌트
@Composable
fun ProductCard(
    product: Product,
    onAddToCart: () -> Unit
) {
    Card(
        modifier = Modifier
            .fillMaxWidth()
            .testTag("product-card-${product.id}"), // 테스트용 ID
        elevation = CardDefaults.cardElevation(4.dp)
    ) {
        Column(
            modifier = Modifier.padding(16.dp)
        ) {
            // 상품명
            Text(
                text = product.name,
                style = MaterialTheme.typography.headlineSmall
            )
            
            // 카테고리
            Text(
                text = product.category,
                style = MaterialTheme.typography.bodySmall,
                color = MaterialTheme.colorScheme.onSurfaceVariant
            )
            
            Spacer(modifier = Modifier.height(8.dp))
            
            // 가격과 평점
            Row(
                modifier = Modifier.fillMaxWidth(),
                horizontalArrangement = Arrangement.SpaceBetween,
                verticalAlignment = Alignment.CenterVertically
            ) {
                Text(
                    text = "${product.price}",
                    style = MaterialTheme.typography.titleLarge,
                    color = MaterialTheme.colorScheme.primary
                )
                
                Row(
                    verticalAlignment = Alignment.CenterVertically
                ) {
                    Text(
                        text = "⭐ ${product.rating}",
                        style = MaterialTheme.typography.bodySmall
                    )
                    
                    Spacer(modifier = Modifier.width(4.dp))
                    
                    Text(
                        text = "(${product.reviewCount})",
                        style = MaterialTheme.typography.bodySmall,
                        color = MaterialTheme.colorScheme.onSurfaceVariant
                    )
                }
            }
            
            Spacer(modifier = Modifier.height(8.dp))
            
            // 상품 설명
            Text(
                text = product.description,
                style = MaterialTheme.typography.bodyMedium,
                color = MaterialTheme.colorScheme.onSurfaceVariant
            )
            
            Spacer(modifier = Modifier.height(16.dp))
            
            // 장바구니 추가 버튼
            Button(
                onClick = onAddToCart,
                modifier = Modifier
                    .fillMaxWidth()
                    .testTag("add-to-cart-${product.id}")
            ) {
                Text("장바구니에 추가")
            }
        }
    }
}
```

#### 🎨 UI 패턴 예제

```kotlin
/**
 * 자주 사용되는 고급 UI 패턴들
 */

// 1. Pull to Refresh 패턴
@Composable
fun PullToRefreshExample() {
    var isRefreshing by remember { mutableStateOf(false) }
    var items by remember { mutableStateOf(generateItems()) }
    
    // React의 useCallback과 유사한 함수 메모이제이션
    val onRefresh = remember<() -> Unit> {
        {
            isRefreshing = true
            // 실제로는 네트워크 호출
            CoroutineScope(Dispatchers.Main).launch {
                delay(2000)
                items = generateItems()
                isRefreshing = false
            }
        }
    }
    
    PullToRefreshBox(
        isRefreshing = isRefreshing,
        onRefresh = onRefresh
    ) {
        LazyColumn {
            items(items) { item ->
                ItemCard(item = item)
            }
        }
    }
}

// 2. 무한 스크롤 패턴
@Composable
fun InfiniteScrollExample() {
    var items by remember { mutableStateOf(generateItems()) }
    var isLoading by remember { mutableStateOf(false) }
    var hasMore by remember { mutableStateOf(true) }
    
    // 더 많은 데이터 로드 함수
    val loadMore = remember {
        {
            if (!isLoading && hasMore) {
                isLoading = true
                CoroutineScope(Dispatchers.Main).launch {
                    delay(1000)
                    val newItems = generateItems()
                    items = items + newItems
                    isLoading = false
                    hasMore = items.size < 100 // 최대 100개까지
                }
            }
        }
    }
    
    LazyColumn {
        items(items) { item ->
            ItemCard(item = item)
        }
        
        // 로딩 인디케이터
        if (isLoading) {
            item {
                Box(
                    modifier = Modifier
                        .fillMaxWidth()
                        .padding(16.dp),
                    contentAlignment = Alignment.Center
                ) {
                    CircularProgressIndicator()
                }
            }
        }
        
        // 스크롤 끝 감지
        if (hasMore && !isLoading) {
            item {
                LaunchedEffect(Unit) {
                    loadMore()
                }
            }
        }
    }
}

// 3. 검색 기능 패턴
@Composable
fun SearchExample() {
    var searchQuery by remember { mutableStateOf("") }
    var searchResults by remember { mutableStateOf<List<String>>(emptyList()) }
    var isSearching by remember { mutableStateOf(false) }
    
    // 디바운스 검색
    LaunchedEffect(searchQuery) {
        if (searchQuery.isNotBlank()) {
            isSearching = true
            delay(300) // 300ms 디바운스
            
            // 실제로는 API 검색
            val results = performSearch(searchQuery)
            searchResults = results
            isSearching = false
        } else {
            searchResults = emptyList()
            isSearching = false
        }
    }
    
    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp)
    ) {
        // 검색 입력 필드
        TextField(
            value = searchQuery,
            onValueChange = { searchQuery = it },
            placeholder = { Text("검색어를 입력하세요") },
            modifier = Modifier.fillMaxWidth(),
            leadingIcon = {
                Icon(Icons.Default.Search, contentDescription = null)
            },
            trailingIcon = {
                if (searchQuery.isNotBlank()) {
                    IconButton(
                        onClick = { searchQuery = "" }
                    ) {
                        Icon(Icons.Default.Clear, contentDescription = null)
                    }
                }
            }
        )
        
        Spacer(modifier = Modifier.height(16.dp))
        
        // 검색 결과
        when {
            isSearching -> {
                Box(
                    modifier = Modifier.fillMaxSize(),
                    contentAlignment = Alignment.Center
                ) {
                    CircularProgressIndicator()
                }
            }
            
            searchResults.isEmpty() && searchQuery.isNotBlank() -> {
                Box(
                    modifier = Modifier.fillMaxSize(),
                    contentAlignment = Alignment.Center
                ) {
                    Text("검색 결과가 없습니다")
                }
            }
            
            searchResults.isNotEmpty() -> {
                LazyColumn {
                    items(searchResults) { result ->
                        SearchResultItem(
                            text = result,
                            query = searchQuery
                        )
                    }
                }
            }
        }
    }
}

// 검색 결과 항목 - 하이라이트 기능 포함
@Composable
fun SearchResultItem(
    text: String,
    query: String
) {
    Card(
        modifier = Modifier
            .fillMaxWidth()
            .padding(vertical = 4.dp)
            .clickable { /* 클릭 처리 */ },
        elevation = CardDefaults.cardElevation(2.dp)
    ) {
        Text(
            text = buildAnnotatedString {
                val startIndex = text.indexOf(query, ignoreCase = true)
                if (startIndex != -1) {
                    append(text.substring(0, startIndex))
                    withStyle(
                        style = SpanStyle(
                            background = Color.Yellow,
                            fontWeight = FontWeight.Bold
                        )
                    ) {
                        append(text.substring(startIndex, startIndex + query.length))
                    }
                    append(text.substring(startIndex + query.length))
                } else {
                    append(text)
                }
            },
            modifier = Modifier.padding(16.dp)
        )
    }
}

// 4. 탭 네비게이션 패턴
@Composable
fun TabNavigationExample() {
    var selectedTab by remember { mutableStateOf(0) }
    val tabs = listOf("홈", "검색", "알림", "프로필")
    
    Column(
        modifier = Modifier.fillMaxSize()
    ) {
        // 탭 바
        TabRow(
            selectedTabIndex = selectedTab,
            modifier = Modifier.fillMaxWidth()
        ) {
            tabs.forEachIndexed { index, title ->
                Tab(
                    selected = selectedTab == index,
                    onClick = { selectedTab = index },
                    text = { Text(title) }
                )
            }
        }
        
        // 탭 콘텐츠
        when (selectedTab) {
            0 -> HomeTabContent()
            1 -> SearchTabContent()
            2 -> NotificationTabContent()
            3 -> ProfileTabContent()
        }
    }
}

// 각 탭의 콘텐츠
@Composable
fun HomeTabContent() {
    Box(
        modifier = Modifier.fillMaxSize(),
        contentAlignment = Alignment.Center
    ) {
        Text(
            text = "🏠 홈 탭",
            style = MaterialTheme.typography.headlineLarge
        )
    }
}

@Composable
fun SearchTabContent() {
    Box(
        modifier = Modifier.fillMaxSize(),
        contentAlignment = Alignment.Center
    ) {
        Text(
            text = "🔍 검색 탭",
            style = MaterialTheme.typography.headlineLarge
        )
    }
}

@Composable
fun NotificationTabContent() {
    Box(
        modifier = Modifier.fillMaxSize(),
        contentAlignment = Alignment.Center
    ) {
        Text(
            text = "🔔 알림 탭",
            style = MaterialTheme.typography.headlineLarge
        )
    }
}

@Composable
fun ProfileTabContent() {
    Box(
        modifier = Modifier.fillMaxSize(),
        contentAlignment = Alignment.Center
    ) {
        Text(
            text = "👤 프로필 탭",
            style = MaterialTheme.typography.headlineLarge
        )
    }
}

// 유틸리티 함수들
fun generateItems(): List<String> {
    return (1..20).map { "Item $it" }
}

fun performSearch(query: String): List<String> {
    // 실제로는 API 호출
    return listOf(
        "검색 결과 1: $query",
        "검색 결과 2: $query 관련",
        "검색 결과 3: $query 정보"
    )
}
```


Jetpack Compose는 **modern한 Android UI 개발**을 위한 강력한 도구입니다. 
**declarative programming**을 통해 **직관적이고 효율적인** UI 개발이 가능합니다. 🚀