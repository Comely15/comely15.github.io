---
layout: single
title: Kotlin_Jetpack-_Compose1
date: 2025-07-17
category: kotlin
---
## Jetpack Compose란?

#### 📋 기본 개념

**Jetpack Compose**는 Android의 **declarative UI toolkit**입니다. 
기존의 **XML 기반 UI**와 달리 
**Kotlin 코드**로 UI를 구성하는 **modern**한 방식입니다. 
**React**나 **Flutter**와 유사한 **declarative programming** 패러다임을 사용합니다.

### 목차

> 🎯 State Management
> 🔄 Recomposition
> 🎨 Layout과 UI Components
> 🎛️ Material Design Components
> 🌐 Navigation과 State Management
> 🗂️ ViewModel과 State Management
> 🎨 Animation과 Transition
> 🌊 Side Effects와 Lifecycle

## 🏗️ 기본 구조와 개념

## 📦 Composable Function
**Composable**은 UI component를 만드는 함수입니다. `@Composable` annotation을 사용하여 정의합니다.

```kotlin
import androidx.compose.foundation.layout.*
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.unit.dp

/**
 * 기본 Composable 함수
 * React의 Function Component와 동일한 개념
 * @Composable annotation은 React의 함수형 컴포넌트와 같은 역할
 */
@Composable
fun Greeting(name: String) {
    // Text는 React의 <Text> 또는 HTML의 <p>와 같은 역할
    Text(
        text = "안녕하세요, $name님!",
        modifier = Modifier.padding(16.dp), // CSS padding과 동일
        style = MaterialTheme.typography.headlineMedium // CSS font-size와 유사
    )
}

/**
 * 복합 Composable 함수
 * React의 복합 컴포넌트와 동일한 구조
 */
@Composable
fun WelcomeScreen() {
    // Column은 CSS flexbox의 flex-direction: column과 동일
    Column(
        modifier = Modifier
            .fillMaxSize() // CSS의 width: 100%, height: 100%와 동일
            .padding(16.dp), // CSS padding: 16px와 동일
        horizontalAlignment = Alignment.CenterHorizontally, // CSS align-items: center
        verticalArrangement = Arrangement.Center // CSS justify-content: center
    ) {
        Greeting("홍길동")
        Greeting("김철수")
        Greeting("이영희")
    }
}
```

## 🎯 State Management
**State**는 UI가 시간에 따라 변하는 데이터입니다. **React의 useState**와 동일한 개념입니다.

```kotlin
import androidx.compose.foundation.clickable
import androidx.compose.material3.Button
import androidx.compose.runtime.*

/**
 * State 관리 예제
 * React의 useState Hook과 동일한 개념
 */
@Composable
fun Counter() {
    // remember는 React의 useState와 동일
    // mutableStateOf는 React의 state setter와 유사
    var count by remember { mutableStateOf(0) }
    
    Column(
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.spacedBy(16.dp)
    ) {
        // count 값이 변경되면 자동으로 UI 리컴포지션
        // React의 state 변경 시 리렌더링과 동일
        Text(
            text = "Count: $count",
            style = MaterialTheme.typography.headlineLarge
        )
        
        // Button의 onClick은 React의 onClick과 동일
        Button(
            onClick = { count++ } // React의 setState와 동일
        ) {
            Text("증가")
        }
        
        Button(
            onClick = { count-- }
        ) {
            Text("감소")
        }
        
        // 조건부 렌더링 (React의 conditional rendering과 동일)
        if (count > 10) {
            Text(
                text = "🎉 10을 넘었습니다!",
                color = MaterialTheme.colorScheme.primary
            )
        }
    }
}
```

## 🔄 Recomposition
**Recomposition**은 state가 변경될 때 UI를 다시 그리는 과정입니다. 
**React의 리렌더링**과 동일한 개념입니다.

```kotlin
/**
 * Recomposition 최적화 예제
 * React의 useMemo, useCallback과 유사한 최적화 기법
 */
@Composable
fun OptimizedComponent() {
    var text by remember { mutableStateOf("") }
    var count by remember { mutableStateOf(0) }
    
    Column {
        // 이 부분은 text가 변경될 때만 recomposition
        TextField(
            value = text,
            onValueChange = { text = it },
            label = { Text("입력하세요") }
        )
        
        // 이 부분은 count가 변경될 때만 recomposition
        Text("Count: $count")
        
        // remember를 사용하여 불필요한 재계산 방지
        // React의 useMemo와 동일한 개념
        val expensiveResult = remember(count) {
            // count가 변경될 때만 계산
            (1..count).map { it * it }.sum()
        }
        
        Text("제곱합: $expensiveResult")
        
        Button(onClick = { count++ }) {
            Text("증가")
        }
    }
}
```

## 🎨 Layout과 UI Components

#### 📐 Layout Composables
**Layout**은 child component들을 배치하는 방식을 정의합니다. 
**CSS Flexbox**나 **CSS Grid**와 유사한 개념입니다.

```kotlin
import androidx.compose.foundation.background
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.material3.*
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.text.style.TextAlign
import androidx.compose.ui.unit.dp

/**
 * 다양한 Layout 예제
 * CSS Flexbox와 Grid의 Compose 버전
 */
@Composable
fun LayoutExamples() {
    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp),
        verticalArrangement = Arrangement.spacedBy(16.dp)
    ) {
        // Row Layout - CSS flexbox의 flex-direction: row와 동일
        Row(
            modifier = Modifier
                .fillMaxWidth()
                .background(
                    Color.LightGray,
                    RoundedCornerShape(8.dp)
                )
                .padding(16.dp),
            horizontalArrangement = Arrangement.SpaceBetween, // CSS justify-content: space-between
            verticalAlignment = Alignment.CenterVertically // CSS align-items: center
        ) {
            Text("Left", color = Color.Blue)
            Text("Center", color = Color.Green)
            Text("Right", color = Color.Red)
        }
        
        // Box Layout - CSS position: relative와 유사
        Box(
            modifier = Modifier
                .size(200.dp)
                .background(Color.Yellow, RoundedCornerShape(8.dp))
        ) {
            // CSS position: absolute와 유사한 절대 위치
            Text(
                "Top Start",
                modifier = Modifier.align(Alignment.TopStart)
            )
            Text(
                "Center",
                modifier = Modifier.align(Alignment.Center)
            )
            Text(
                "Bottom End",
                modifier = Modifier.align(Alignment.BottomEnd)
            )
        }
        
        // LazyColumn - React의 Virtual List와 동일
        LazyColumn(
            modifier = Modifier
                .fillMaxWidth()
                .height(200.dp)
                .background(Color.LightBlue, RoundedCornerShape(8.dp)),
            verticalArrangement = Arrangement.spacedBy(8.dp),
            contentPadding = PaddingValues(16.dp)
        ) {
            // items는 React의 map 함수와 유사
            items(20) { index ->
                Card(
                    modifier = Modifier.fillMaxWidth(),
                    elevation = CardDefaults.cardElevation(4.dp)
                ) {
                    Text(
                        text = "Item $index",
                        modifier = Modifier.padding(16.dp),
                        textAlign = TextAlign.Center
                    )
                }
            }
        }
    }
}
```

## 🎛️ Material Design Components
**Material Design**은 Google의 design system입니다.
**Bootstrap**이나 **Ant Design**과 유사한 UI 라이브러리입니다.

```kotlin
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.selection.selectable
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.*
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp

/**
 * Material Design Components 예제
 * Bootstrap이나 Ant Design의 Component와 유사
 */
@Composable
fun MaterialComponents() {
    var textFieldValue by remember { mutableStateOf("") }
    var switchChecked by remember { mutableStateOf(false) }
    var sliderValue by remember { mutableStateOf(0.5f) }
    var selectedOption by remember { mutableStateOf("Option 1") }
    var showDialog by remember { mutableStateOf(false) }
    
    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp),
        verticalArrangement = Arrangement.spacedBy(16.dp)
    ) {
        // TextField - HTML input과 동일
        TextField(
            value = textFieldValue,
            onValueChange = { textFieldValue = it },
            label = { Text("사용자 이름") },
            placeholder = { Text("이름을 입력하세요") },
            leadingIcon = { Icon(Icons.Default.Person, contentDescription = null) },
            modifier = Modifier.fillMaxWidth()
        )
        
        // Switch - HTML checkbox와 유사하지만 toggle 형태
        Row(
            verticalAlignment = Alignment.CenterVertically,
            horizontalArrangement = Arrangement.spacedBy(8.dp)
        ) {
            Switch(
                checked = switchChecked,
                onCheckedChange = { switchChecked = it }
            )
            Text("알림 활성화")
        }
        
        // Slider - HTML range input과 동일
        Column {
            Text("값: ${(sliderValue * 100).toInt()}%")
            Slider(
                value = sliderValue,
                onValueChange = { sliderValue = it },
                modifier = Modifier.fillMaxWidth()
            )
        }
        
        // RadioButton Group - HTML radio button과 동일
        Column {
            Text("옵션을 선택하세요:")
            listOf("Option 1", "Option 2", "Option 3").forEach { option ->
                Row(
                    modifier = Modifier
                        .fillMaxWidth()
                        .selectable(
                            selected = selectedOption == option,
                            onClick = { selectedOption = option }
                        )
                        .padding(vertical = 4.dp),
                    verticalAlignment = Alignment.CenterVertically
                ) {
                    RadioButton(
                        selected = selectedOption == option,
                        onClick = { selectedOption = option }
                    )
                    Text(
                        text = option,
                        modifier = Modifier.padding(start = 8.dp)
                    )
                }
            }
        }
        
        // Button - HTML button과 동일
        Row(
            horizontalArrangement = Arrangement.spacedBy(8.dp)
        ) {
            Button(
                onClick = { showDialog = true }
            ) {
                Icon(Icons.Default.Info, contentDescription = null)
                Spacer(modifier = Modifier.width(8.dp))
                Text("다이얼로그 열기")
            }
            
            OutlinedButton(
                onClick = { /* 액션 */ }
            ) {
                Text("외곽선 버튼")
            }
            
            TextButton(
                onClick = { /* 액션 */ }
            ) {
                Text("텍스트 버튼")
            }
        }
        
        // Dialog - Web의 modal과 동일
        if (showDialog) {
            AlertDialog(
                onDismissRequest = { showDialog = false },
                title = { Text("확인") },
                text = { Text("정말로 실행하시겠습니까?") },
                confirmButton = {
                    TextButton(
                        onClick = { showDialog = false }
                    ) {
                        Text("확인")
                    }
                },
                dismissButton = {
                    TextButton(
                        onClick = { showDialog = false }
                    ) {
                        Text("취소")
                    }
                }
            )
        }
    }
}
```

## 🌐 Navigation과 State Management

#### 🧭 Navigation
**Navigation**은 화면 간 이동을 관리합니다. 

```kotlin
import androidx.compose.foundation.layout.*
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.*
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp
import androidx.navigation.NavController
import androidx.navigation.compose.NavHost
import androidx.navigation.compose.composable
import androidx.navigation.compose.rememberNavController

/**
 * Navigation 예제
 * React Router의 Router, Route와 동일한 개념
 */
@Composable
fun NavigationExample() {
    // NavController는 React Router의 useNavigate와 유사
    val navController = rememberNavController()
    
    // NavHost는 React Router의 Routes와 동일
    NavHost(
        navController = navController,
        startDestination = "home" // React Router의 default route
    ) {
        // composable은 React Router의 Route와 동일
        composable("home") {
            HomeScreen(navController)
        }
        composable("profile") {
            ProfileScreen(navController)
        }
        composable("settings") {
            SettingsScreen(navController)
        }
        // 매개변수가 있는 라우트 - React Router의 :id와 동일
        composable("detail/{itemId}") { backStackEntry ->
            val itemId = backStackEntry.arguments?.getString("itemId")
            DetailScreen(navController, itemId)
        }
    }
}

/**
 * 홈 화면
 * React의 Home Component와 동일한 구조
 */
@Composable
fun HomeScreen(navController: NavController) {
    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.spacedBy(16.dp)
    ) {
        Text(
            text = "🏠 홈 화면",
            style = MaterialTheme.typography.headlineLarge
        )
        
        // 네비게이션 버튼들
        Button(
            onClick = { 
                // React Router의 navigate('/profile')와 동일
                navController.navigate("profile")
            },
            modifier = Modifier.fillMaxWidth()
        ) {
            Icon(Icons.Default.Person, contentDescription = null)
            Spacer(modifier = Modifier.width(8.dp))
            Text("프로필")
        }
        
        Button(
            onClick = { 
                navController.navigate("settings")
            },
            modifier = Modifier.fillMaxWidth()
        ) {
            Icon(Icons.Default.Settings, contentDescription = null)
            Spacer(modifier = Modifier.width(8.dp))
            Text("설정")
        }
        
        Button(
            onClick = { 
                // 매개변수와 함께 네비게이션
                navController.navigate("detail/123")
            },
            modifier = Modifier.fillMaxWidth()
        ) {
            Icon(Icons.Default.Info, contentDescription = null)
            Spacer(modifier = Modifier.width(8.dp))
            Text("상세 정보")
        }
    }
}

/**
 * 프로필 화면
 */
@Composable
fun ProfileScreen(navController: NavController) {
    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.spacedBy(16.dp)
    ) {
        Text(
            text = "👤 프로필 화면",
            style = MaterialTheme.typography.headlineLarge
        )
        
        Card(
            modifier = Modifier.fillMaxWidth(),
            elevation = CardDefaults.cardElevation(4.dp)
        ) {
            Column(
                modifier = Modifier.padding(16.dp)
            ) {
                Text("이름: 홍길동")
                Text("이메일: hong@example.com")
                Text("가입일: 2024-01-01")
            }
        }
        
        Button(
            onClick = { 
                // React Router의 navigate(-1) 또는 history.back()과 동일
                navController.popBackStack()
            }
        ) {
            Icon(Icons.Default.ArrowBack, contentDescription = null)
            Spacer(modifier = Modifier.width(8.dp))
            Text("뒤로가기")
        }
    }
}

/**
 * 설정 화면
 */
@Composable
fun SettingsScreen(navController: NavController) {
    var darkMode by remember { mutableStateOf(false) }
    var notifications by remember { mutableStateOf(true) }
    
    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp),
        verticalArrangement = Arrangement.spacedBy(16.dp)
    ) {
        Text(
            text = "⚙️ 설정 화면",
            style = MaterialTheme.typography.headlineLarge
        )
        
        Card(
            modifier = Modifier.fillMaxWidth(),
            elevation = CardDefaults.cardElevation(4.dp)
        ) {
            Column(
                modifier = Modifier.padding(16.dp),
                verticalArrangement = Arrangement.spacedBy(12.dp)
            ) {
                // 다크 모드 설정
                Row(
                    modifier = Modifier.fillMaxWidth(),
                    horizontalArrangement = Arrangement.SpaceBetween,
                    verticalAlignment = Alignment.CenterVertically
                ) {
                    Text("다크 모드")
                    Switch(
                        checked = darkMode,
                        onCheckedChange = { darkMode = it }
                    )
                }
                
                Divider()
                
                // 알림 설정
                Row(
                    modifier = Modifier.fillMaxWidth(),
                    horizontalArrangement = Arrangement.SpaceBetween,
                    verticalAlignment = Alignment.CenterVertically
                ) {
                    Text("알림")
                    Switch(
                        checked = notifications,
                        onCheckedChange = { notifications = it }
                    )
                }
            }
        }
        
        Button(
            onClick = { navController.popBackStack() }
        ) {
            Icon(Icons.Default.ArrowBack, contentDescription = null)
            Spacer(modifier = Modifier.width(8.dp))
            Text("뒤로가기")
        }
    }
}

/**
 * 상세 화면
 */
@Composable
fun DetailScreen(navController: NavController, itemId: String?) {
    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.spacedBy(16.dp)
    ) {
        Text(
            text = "📋 상세 화면",
            style = MaterialTheme.typography.headlineLarge
        )
        
        Card(
            modifier = Modifier.fillMaxWidth(),
            elevation = CardDefaults.cardElevation(4.dp)
        ) {
            Column(
                modifier = Modifier.padding(16.dp)
            ) {
                Text("Item ID: $itemId")
                Text("상세 정보가 여기에 표시됩니다.")
                Text("이것은 매개변수를 받는 화면의 예제입니다.")
            }
        }
        
        Button(
            onClick = { navController.popBackStack() }
        ) {
            Icon(Icons.Default.ArrowBack, contentDescription = null)
            Spacer(modifier = Modifier.width(8.dp))
            Text("뒤로가기")
        }
    }
}
```

## 🗂️ ViewModel과 State Management
**ViewModel**은 UI와 비즈니스 로직을 분리하는 아키텍처 패턴입니다. **Redux**나 **MobX**와 유사한 개념입니다.

```kotlin
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.*
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp
import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewmodel.compose.viewModel

/**
 * 할일 데이터 클래스
 */
data class Todo(
    val id: Int,
    val title: String,
    val completed: Boolean = false
)

/**
 * TodoViewModel
 * Redux의 Store나 MobX의 Store와 유사한 개념
 */
class TodoViewModel : ViewModel() {
    // _todos는 private state (Redux의 state와 유사)
    private val _todos = mutableStateListOf<Todo>()
    val todos: List<Todo> = _todos
    
    // Redux의 action creator와 유사한 함수들
    fun addTodo(title: String) {
        val newTodo = Todo(
            id = _todos.size + 1,
            title = title
        )
        _todos.add(newTodo)
    }
    
    fun toggleTodo(id: Int) {
        val index = _todos.indexOfFirst { it.id == id }
        if (index != -1) {
            _todos[index] = _todos[index].copy(completed = !_todos[index].completed)
        }
    }
    
    fun deleteTodo(id: Int) {
        _todos.removeAll { it.id == id }
    }
    
    // 계산된 속성 (Redux의 selector와 유사)
    val completedCount: Int
        get() = _todos.count { it.completed }
    
    val pendingCount: Int
        get() = _todos.count { !it.completed }
}

/**
 * Todo 앱 UI
 * React의 App Component와 유사
 */
@Composable
fun TodoApp() {
    // viewModel()은 React의 useContext나 Redux의 useSelector와 유사
    val viewModel: TodoViewModel = viewModel()
    var newTodoText by remember { mutableStateOf("") }
    
    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp)
    ) {
        // 헤더
        Text(
            text = "📝 할일 관리",
            style = MaterialTheme.typography.headlineLarge,
            modifier = Modifier.padding(bottom = 16.dp)
        )
        
        // 통계 정보
        Card(
            modifier = Modifier.fillMaxWidth(),
            elevation = CardDefaults.cardElevation(4.dp)
        ) {
            Row(
                modifier = Modifier
                    .fillMaxWidth()
                    .padding(16.dp),
                horizontalArrangement = Arrangement.SpaceBetween
            ) {
                Text("완료: ${viewModel.completedCount}")
                Text("대기: ${viewModel.pendingCount}")
                Text("전체: ${viewModel.todos.size}")
            }
        }
        
        Spacer(modifier = Modifier.height(16.dp))
        
        // 새 할일 추가
        Row(
            modifier = Modifier.fillMaxWidth(),
            verticalAlignment = Alignment.CenterVertically
        ) {
            TextField(
                value = newTodoText,
                onValueChange = { newTodoText = it },
                placeholder = { Text("새 할일을 입력하세요") },
                modifier = Modifier.weight(1f)
            )
            
            Spacer(modifier = Modifier.width(8.dp))
            
            Button(
                onClick = {
                    if (newTodoText.isNotBlank()) {
                        viewModel.addTodo(newTodoText)
                        newTodoText = ""
                    }
                }
            ) {
                Icon(Icons.Default.Add, contentDescription = null)
            }
        }
        
        Spacer(modifier = Modifier.height(16.dp))
        
        // 할일 목록
        LazyColumn(
            verticalArrangement = Arrangement.spacedBy(8.dp)
        ) {
            items(viewModel.todos) { todo ->
                TodoItem(
                    todo = todo,
                    onToggle = { viewModel.toggleTodo(todo.id) },
                    onDelete = { viewModel.deleteTodo(todo.id) }
                )
            }
        }
    }
}

/**
 * 개별 할일 항목
 * React의 TodoItem Component와 유사
 */
@Composable
fun TodoItem(
    todo: Todo,
    onToggle: () -> Unit,
    onDelete: () -> Unit
) {
    Card(
        modifier = Modifier.fillMaxWidth(),
        elevation = CardDefaults.cardElevation(2.dp)
    ) {
        Row(
            modifier = Modifier
                .fillMaxWidth()
                .padding(16.dp),
            verticalAlignment = Alignment.CenterVertically
        ) {
            // 체크박스
            Checkbox(
                checked = todo.completed,
                onCheckedChange = { onToggle() }
            )
            
            Spacer(modifier = Modifier.width(8.dp))
            
            // 할일 제목
            Text(
                text = todo.title,
                modifier = Modifier.weight(1f),
                style = if (todo.completed) {
                    MaterialTheme.typography.bodyLarge.copy(
                        color = MaterialTheme.colorScheme.onSurface.copy(alpha = 0.6f)
                    )
                } else {
                    MaterialTheme.typography.bodyLarge
                }
            )
            
            // 삭제 버튼
            IconButton(
                onClick = onDelete
            ) {
                Icon(
                    Icons.Default.Delete,
                    contentDescription = "삭제",
                    tint = MaterialTheme.colorScheme.error
                )
            }
        }
    }
}
```

## 🎨 Animation과 Transition

#### ✨ Animation

**Animation**은 UI 요소의 변화를 부드럽게 만듭니다. **CSS Animation**이나 **Framer Motion**과 유사한 개념입니다.

```kotlin
import androidx.compose.animation.*
import androidx.compose.animation.core.*
import androidx.compose.foundation.background
import androidx.compose.foundation.clickable
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.shape.CircleShape
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.Favorite
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.draw.clip
import androidx.compose.ui.draw.rotate
import androidx.compose.ui.draw.scale
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.unit.dp

/**
 * Animation 예제
 * CSS Animation이나 Framer Motion과 유사한 개념
 */
@Composable
fun AnimationExamples() {
    var isExpanded by remember { mutableStateOf(false) }
    var isRotated by remember { mutableStateOf(false) }
    var isVisible by remember { mutableStateOf(true) }
    var selectedColor by remember { mutableStateOf(Color.Blue) }
    
    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp),
        verticalArrangement = Arrangement.spacedBy(24.dp)
    ) {
        Text(
            text = "✨ Animation 예제",
            style = MaterialTheme.typography.headlineLarge
        )
        
        // 1. 크기 애니메이션 (CSS transform: scale과 유사)
        Card(
            modifier = Modifier
                .fillMaxWidth()
                .clickable { isExpanded = !isExpanded },
            elevation = CardDefaults.cardElevation(4.dp)
        ) {
            // animateContentSize는 CSS transition과 유사
            Column(
                modifier = Modifier
                    .animateContentSize( // 자동으로 크기 변화 애니메이션
                        animationSpec = tween(
                            durationMillis = 300,
                            easing = FastOutSlowInEasing
                        )
                    )
                    .padding(16.dp)
            ) {
                Text(
                    text = "📏 크기 애니메이션",
                    style = MaterialTheme.typography.headlineMedium
                )
                
                // 조건부 콘텐츠 - React의 conditional rendering과 유사
                if (isExpanded) {
                    Spacer(modifier = Modifier.height(8.dp))
                    Text(
                        text = "이 텍스트는 확장될 때만 보입니다. " +
                                "카드가 부드럽게 크기가 변합니다. " +
                                "CSS의 transition 속성과 동일한 효과입니다.",
                        color = MaterialTheme.colorScheme.onSurfaceVariant
                    )
                }
            }
        }
        
        // 2. 회전 애니메이션 (CSS transform: rotate와 유사)
        Card(
            modifier = Modifier
                .fillMaxWidth()
                .clickable { isRotated = !isRotated },
            elevation = CardDefaults.cardElevation(4.dp)
        ) {
            Row(
                modifier = Modifier.padding(16.dp),
                verticalAlignment = Alignment.CenterVertically
            ) {
                // animateFloatAsState는 CSS transition과 유사
                val rotation by animateFloatAsState(
                    targetValue = if (isRotated) 360f else 0f,
                    animationSpec = tween(
                        durationMillis = 1000,
                        easing = LinearOutSlowInEasing
                    ),
                    label = "rotation"
                )
                
                Icon(
                    Icons.Default.Favorite,
                    contentDescription = null,
                    modifier = Modifier
                        .size(48.dp)
                        .rotate(rotation), // CSS transform: rotate()와 동일
                    tint = Color.Red
                )
                
                Spacer(modifier = Modifier.width(16.dp))
                
                Text(
                    text = "🔄 회전 애니메이션\n클릭하여 회전시키기",
                    style = MaterialTheme.typography.bodyLarge
                )
            }
        }
        
        // 3. 페이드 인/아웃 애니메이션 (CSS opacity와 유사)
        Card(
            modifier = Modifier.fillMaxWidth(),
            elevation = CardDefaults.cardElevation(4.dp)
        ) {
            Column(
                modifier = Modifier.padding(16.dp)
            ) {
                Text(
                    text = "👻 페이드 애니메이션",
                    style = MaterialTheme.typography.headlineMedium
                )
                
                Spacer(modifier = Modifier.height(8.dp))
                
                // AnimatedVisibility는 CSS transition과 유사
                AnimatedVisibility(
                    visible = isVisible,
                    enter = fadeIn(
                        animationSpec = tween(500)
                    ) + slideInVertically( // CSS translateY와 유사
                        animationSpec = tween(500)
                    ),
                    exit = fadeOut(
                        animationSpec = tween(500)
                    ) + slideOutVertically(
                        animationSpec = tween(500)
                    )
                ) {
                    Text(
                        text = "이 텍스트는 페이드 인/아웃됩니다",
                        modifier = Modifier
                            .background(
                                Color.LightGray,
                                RoundedCornerShape(8.dp)
                            )
                            .padding(16.dp)
                    )
                }
                
                Spacer(modifier = Modifier.height(8.dp))
                
                Button(
                    onClick = { isVisible = !isVisible }
                ) {
                    Text(if (isVisible) "숨기기" else "보이기")
                }
            }
        }
        
        // 4. 색상 애니메이션 (CSS color transition과 유사)
        Card(
            modifier = Modifier.fillMaxWidth(),
            elevation = CardDefaults.cardElevation(4.dp)
        ) {
            Column(
                modifier = Modifier.padding(16.dp)
            ) {
                Text(
                    text = "🎨 색상 애니메이션",
                    style = MaterialTheme.typography.headlineMedium
                )
                
                Spacer(modifier = Modifier.height(16.dp))
                
                // animateColorAsState는 CSS color transition과 유사
                val animatedColor by animateColorAsState(
                    targetValue = selectedColor,
                    animationSpec = tween(
                        durationMillis = 800,
                        easing = FastOutSlowInEasing
                    ),
                    label = "color"
                )
                
                Box(
                    modifier = Modifier
                        .size(100.dp)
                        .background(animatedColor, CircleShape)
                        .clip(CircleShape)
                        .align(Alignment.CenterHorizontally)
                )
                
                Spacer(modifier = Modifier.height(16.dp))
                
                // 색상 선택 버튼들
                Row(
                    horizontalArrangement = Arrangement.SpaceEvenly,
                    modifier = Modifier.fillMaxWidth()
                ) {
                    listOf(
                        Color.Red to "빨강",
                        Color.Green to "초록",
                        Color.Blue to "파랑",
                        Color.Yellow to "노랑"
                    ).forEach { (color, name) ->
                        Button(
                            onClick = { selectedColor = color },
                            colors = ButtonDefaults.buttonColors(
                                containerColor = color
                            )
                        ) {
                            Text(name)
                        }
                    }
                }
            }
        }
        
        // 5. 스케일 애니메이션 (CSS transform: scale과 유사)
        Card(
            modifier = Modifier.fillMaxWidth(),
            elevation = CardDefaults.cardElevation(4.dp)
        ) {
            Column(
                modifier = Modifier.padding(16.dp),
                horizontalAlignment = Alignment.CenterHorizontally
            ) {
                Text(
                    text = "📐 스케일 애니메이션",
                    style = MaterialTheme.typography.headlineMedium
                )
                
                Spacer(modifier = Modifier.height(16.dp))
                
                var isScaled by remember { mutableStateOf(false) }
                
                // animateFloatAsState로 스케일 값 애니메이션
                val scale by animateFloatAsState(
                    targetValue = if (isScaled) 1.5f else 1f,
                    animationSpec = spring(
                        dampingRatio = Spring.DampingRatioMediumBouncy,
                        stiffness = Spring.StiffnessLow
                    ),
                    label = "scale"
                )
                
                Text(
                    text = "클릭하여 확대/축소",
                    modifier = Modifier
                        .scale(scale) // CSS transform: scale()와 동일
                        .clickable { isScaled = !isScaled }
                        .background(
                            Color.LightBlue,
                            RoundedCornerShape(8.dp)
                        )
                        .padding(16.dp),
                    style = MaterialTheme.typography.bodyLarge
                )
            }
        }
    }
}
```

## 🌊 Side Effects와 Lifecycle

#### ⚡ Side Effects
Composable 함수에서 **외부 상태를 변경하거나 비동기 작업**을 수행하는 것입니다. 

```kotlin
import androidx.compose.foundation.layout.*
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.platform.LocalLifecycleOwner
import androidx.compose.ui.unit.dp
import androidx.lifecycle.Lifecycle
import androidx.lifecycle.LifecycleEventObserver
import kotlinx.coroutines.delay

/**
 * Side Effects 예제
 * React의 useEffect Hook과 동일한 개념
 */
@Composable
fun SideEffectsExamples() {
    var count by remember { mutableStateOf(0) }
    var time by remember { mutableStateOf(0L) }
    var isTimerRunning by remember { mutableStateOf(false) }
    
    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp),
        verticalArrangement = Arrangement.spacedBy(16.dp)
    ) {
        Text(
            text = "⚡ Side Effects 예제",
            style = MaterialTheme.typography.headlineLarge
        )
        
        // 1. LaunchedEffect - React의 useEffect와 유사
        // 컴포넌트가 마운트될 때 한 번만 실행
        LaunchedEffect(Unit) {
            println("🚀 컴포넌트가 마운트되었습니다") // React의 useEffect(() => {}, [])
        }
        
        // 2. LaunchedEffect with dependency
        // count가 변경될 때마다 실행
        LaunchedEffect(count) {
            println("📊 Count가 변경되었습니다: $count") // React의 useEffect(() => {}, [count])
            
            // 5초 후 자동으로 카운트 증가
            delay(5000)
            if (count < 10) {
                count++
            }
        }
        
        // 3. Timer using LaunchedEffect
        // React의 setInterval과 유사
        LaunchedEffect(isTimerRunning) {
            if (isTimerRunning) {
                while (isTimerRunning) {
                    delay(1000)
                    time++
                }
            }
        }
        
        // 4. DisposableEffect - React의 useEffect cleanup과 유사
        val lifecycleOwner = LocalLifecycleOwner.current
        DisposableEffect(lifecycleOwner) {
            val observer = LifecycleEventObserver { _, event ->
                when (event) {
                    Lifecycle.Event.ON_START -> {
                        println("🌟 화면이 시작되었습니다")
                    }
                    Lifecycle.Event.ON_STOP -> {
                        println("⏹️ 화면이 멈췄습니다")
                    }
                    else -> {}
                }
            }
            
            lifecycleOwner.lifecycle.addObserver(observer)
            
            // cleanup 함수 - React의 useEffect cleanup과 동일
            onDispose {
                lifecycleOwner.lifecycle.removeObserver(observer)
                println("🧹 cleanup 실행됨")
            }
        }
        
        // UI 요소들
        Card(
            modifier = Modifier.fillMaxWidth(),
            elevation = CardDefaults.cardElevation(4.dp)
        ) {
            Column(
                modifier = Modifier.padding(16.dp)
            ) {
                Text(
                    text = "자동 카운터: $count",
                    style = MaterialTheme.typography.headlineMedium
                )
                Text(
                    text = "5초마다 자동으로 증가합니다",
                    color = MaterialTheme.colorScheme.onSurfaceVariant
                )
                
                Spacer(modifier = Modifier.height(8.dp))
                
                Button(
                    onClick = { count = 0 }
                ) {
                    Text("리셋")
                }
            }
        }
        
        Card(
            modifier = Modifier.fillMaxWidth(),
            elevation = CardDefaults.cardElevation(4.dp)
        ) {
            Column(
                modifier = Modifier.padding(16.dp)
            ) {
                Text(
                    text = "타이머: ${time}초",
                    style = MaterialTheme.typography.headlineMedium
                )
                
                Spacer(modifier = Modifier.height(8.dp))
                
                Row(
                    horizontalArrangement = Arrangement.spacedBy(8.dp)
                ) {
                    Button(
                        onClick = { isTimerRunning = !isTimerRunning }
                    ) {
                        Text(if (isTimerRunning) "일시정지" else "시작")
                    }
                    
                    Button(
                        onClick = { 
                            time = 0
                            isTimerRunning = false
                        }
                    ) {
                        Text("리셋")
                    }
                }
            }
        }
    }
}
```

#### 🔄 다른 프레임워크와의 비교
- **React**: JSX와 유사한 Kotlin DSL 사용
- **Flutter**: Widget tree와 비슷한 Composable tree 구조
- **SwiftUI**: Apple의 declarative UI와 동일한 개념
- **XML Layout**: Imperative한 방식에서 Declarative한 방식으로 전환

#### ✨ 주요 특징
- **Less Code**: XML과 Java/Kotlin 코드 분리 불필요
- **Intuitive**: 직관적인 API 설계
- **Accelerated Development**: 빠른 개발 속도
- **Powerful**: 강력한 customization 기능