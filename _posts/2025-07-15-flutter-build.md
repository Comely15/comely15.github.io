---
layout: single
title: Flutter_Build
date: 2025-07-15
categories:
  - flutter
---
**Flavor**와 **Fastlane**은 앱을 **빠르게**, **실수 없이**, **환경에 맞게** 배포하기 위해 꼭 알아두면 좋은 기술입니다.

---

좋아요! Flutter 앱 개발에서 **Flavor**와 **Fastlane**은 앱을 **빠르게**, **실수 없이**, **환경에 맞게** 배포하기 위해 꼭 알아두면 좋은 기술이에요.  
고등학생도 이해할 수 있도록 쉽게 풀어드릴게요! 👇

---

### 🍭 1. **Flavor (플레이버)** — “앱 환경을 구분하는 기술”

👉 플레이버는 하나의 앱을 **여러 버전으로 나눠서** 만들 수 있게 도와주는 기능입니다.  

|구분|하는 일|
|---|---|
|`Dev` (개발용)|개발자가 테스트할 때 쓰는 앱|
|`Staging` (준비용)|테스트 중인 버전, QA 팀이 확인용|
|`Prod` (운영용)|진짜 사용자들이 쓰는 앱|

→ 같은 앱이라도 **이름, 아이콘, 서버 주소**를 다르게 설정할 수 있어요!

---

### 💡 왜 필요한가?

> 실수 방지 + 테스트 편리

예: `prod`용 앱은 진짜 유저 데이터를 쓰고,  
`dev`는 테스트용 서버를 쓰게 할 수 있어요.

---

### 📂 어떻게 씁니까?

```bash
lib/
  main_dev.dart
  main_prod.dart
  flavors/
    dev_config.dart
    prod_config.dart
```

- `main_dev.dart`: 개발용 앱 실행 진입점
- `dev_config.dart`: 개발용 설정들 (예: 테스트 서버 주소)

---

### ⚙️ Android 설정 (`build.gradle`)

```groovy
flavorDimensions "app"
productFlavors {
  dev {
    dimension "app"
    applicationIdSuffix ".dev"  // 앱 ID에 .dev 붙임
    resValue "string", "app_name", "MyApp Dev"
  }
  prod {
    dimension "app"
    resValue "string", "app_name", "MyApp"
  }
}
```

### ⚙️ iOS 설정 (`Xcode`)

- Xcode에서 **Scheme(스킴)** 복제
- `.xcconfig` 파일로 dev/prod 설정 구분

---

### 🚀 2. **Fastlane (패스트레인)** — “배포 자동화 도구”

👉 앱을 iOS, Android에 **자동으로 업로드**하고  
버전 번호도 **자동으로 올려주는 도구**예요!

> 📦 복잡한 앱 배포 작업을 “한 줄 명령어”로 끝낼 수 있어요.

---

### 💡 어떤 일을 합니까?

|예시|설명|
|---|---|
|앱 빌드|코드 → 앱 파일로 만듦 (`.apk`, `.ipa`)|
|스토어 업로드|Play Store, App Store에 업로드|
|테스트 업로드|Firebase App Distribution / TestFlight|
|버전 관리|자동으로 버전 넘버 증가|

---

### 🛠 예시

#### ✅ Android (`android/fastlane/Fastfile`)

```ruby
platform :android do
  lane :deploy_dev do
    gradle(task: "assembleDevRelease")   # dev용 빌드
    upload_to_play_store(track: 'internal')  # 내부 테스트 업로드
  end
end
```

#### ✅ iOS (`ios/fastlane/Fastfile`)

```ruby
platform :ios do
  lane :beta do
    build_app(scheme: "MyAppDev")  # dev 스킴으로 빌드
    upload_to_testflight           # TestFlight로 업로드
  end
end
```

---

### 🚀 실행 예시

```bash
fastlane deploy_dev    # 안드로이드 dev용 빌드 + 업로드
fastlane beta          # iOS dev용 빌드 + TestFlight 업로드
```

---

## 🧩 3. Flavor + Fastlane 함께 쓰면?

|명령어|하는 일|
|---|---|
|`fastlane deploy_dev`|dev 환경으로 앱 빌드 + 테스트 배포|
|`fastlane deploy_prod`|운영용 앱을 앱스토어에 자동 업로드|

---

## 📘 꼭 함께 알아두면 좋은 것들

|기술|설명|
|---|---|
|`.env` + `flutter_dotenv`|환경변수로 API 주소 관리|
|Firebase App Distribution|테스트 사용자에게 빠르게 배포|
|GitHub Actions|Fastlane과 함께 자동 빌드하는 CI/CD 도구|

---

#### 🧠 비유로 정리!

- **Flavor** = “같은 음식(앱)을 매운맛/순한맛(환경)으로 나눠서 조리하는 것”
- **Fastlane** = “배달로봇처럼 앱을 자동으로 스토어나 테스트 사용자에게 전달하는 도구”

---

#### 🧪 연습하고 싶다면

- 간단한 Flutter 앱에서 dev/prod flavor 만들고
- Android에서 `fastlane` 설정해서 `deploy_dev` 해보기

---
