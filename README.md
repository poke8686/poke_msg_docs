# MSG - 알림 기반 자동 전달 앱

안드로이드 기기의 알림(Notification)을 실시간으로 수신하여 저장하고, 사용자가 설정한 조건에 맞는 알림이 감지되면 자동으로 SMS 발송 및/또는 URL 호출을 수행하는 앱입니다.

## 주요 기능

### 알림 수신 및 저장
- `NotificationListenerService`를 활용하여 모든 앱의 알림을 실시간 수신
- 알림 제목, 내용, 앱 정보를 Room DB에 저장
- 확장 알림 데이터 완전 캡처 (BigTextStyle, InboxStyle, MessagingStyle, SubText 등)
- 알림에 포함된 이미지(BigPicture, 대형 아이콘, Icon 객체)를 WebP 형식으로 캡처 및 저장
- 30일 이후 데이터 자동 삭제 (이미지 파일 포함)
- 상태바 알림 재스캔 기능 (현재 표시 중인 알림을 다시 캡처)

### 조건 기반 자동 전달
- 앱별 알림 감시 규칙을 생성하여 자동 전달
- **제목/내용 분리 키워드**: 제목과 내용에 각각 독립적인 키워드 설정 가능
- **다중 키워드** 지원: 줄바꿈으로 여러 키워드 입력
- **AND/OR 결합**: 모든 키워드 일치(AND) 또는 하나 이상 일치(OR) 모드 선택
- **정규식** 지원: 고급 패턴 매칭을 위한 정규표현식 사용 가능
- **메시지 템플릿**: `{app}`, `{title}`, `{content}`, `{time}` 변수로 본문 커스터마이징

### 다중 전송 방식
- **SMS 전송**: 조건 매칭 시 자동 문자 전송 (한글 장문 자동 분할)
- **URL 호출**: HTTP GET/POST 방식 선택, 템플릿 변수 지원
  - GET: URL에 변수를 포함한 전체 주소 입력 (예: `https://api.example.com?title={title}&time={time}`)
  - POST: URL과 Body에 각각 변수 사용 가능
- SMS와 URL을 독립적으로 ON/OFF (둘 다 활성화 시 병렬 실행)
- 최소 하나 이상의 전송 방식 선택 필수

### 규칙 백업/복원
- JSON 형식으로 전달 규칙을 파일로 내보내기
- 백업 파일에서 규칙 가져오기 (기존 규칙에 추가)

### 알림 제외
- 알림 기록에서 항목을 길게 눌러 상세 정보 확인 후 바로 "알림제외" 가능
- 알림제외 탭에서 제외된 앱 관리 (해제하면 다시 기록)

### 백그라운드 안정성
- Foreground Service로 앱 생존 보장
- 재부팅 후 자동 시작 (BootReceiver)
- WorkManager 기반 15분 주기 서비스 헬스체크
- 제조사별 자동시작/배터리 최적화 설정 안내 (삼성, 샤오미, 화웨이 등)

## 설치 및 빌드

### 요구 사항
- Android Studio (Arctic Fox 이상)
- JDK 17 이상
- Android SDK API 35

### 빌드
```bash
git clone https://github.com/poke8686/poke_msg.git
cd poke_msg

# 릴리즈 APK
./gradlew assembleRelease

# 릴리즈 AAB (Play Store 배포용)
./gradlew bundleRelease

# 디버그 APK
./gradlew assembleDebug
```

- 릴리즈 APK: `app/build/outputs/apk/release/`
- 릴리즈 AAB: `app/build/outputs/bundle/release/`

## 필요 권한

| 권한 | 용도 |
|------|------|
| 알림 접근 | 다른 앱의 알림을 수신하기 위해 필요 (설정에서 수동 허용) |
| SMS 발송 | 조건 매칭 시 자동 문자 전송 |
| 인터넷 | URL 호출 기능 및 광고 |
| 배터리 최적화 예외 | 백그라운드에서 서비스가 종료되지 않도록 보장 |
| 알림 표시 | 서비스 실행 상태 알림 표시 (Android 13+) |

## 사용 방법

### 1. 앱 설치 후 권한 설정
- 알림 접근 권한: 설정 → 알림 → 알림 접근 허용에서 MSG 앱 활성화
- SMS 발송 권한: 앱 최초 실행 시 자동 요청
- 배터리 최적화 제외: 앱 안내에 따라 설정

### 2. 홈 탭
- 서비스 시작/중지 버튼으로 알림 수신 제어
- 알림 리스너 권한 미설정 시 바로 설정 화면으로 이동
- 상태바 알림 재스캔 버튼으로 현재 알림 다시 캡처
- 오늘 수신 알림, 매칭 발송, 등록 규칙 수 통계 확인
- 규칙 백업/복원으로 데이터 안전하게 관리

### 3. 전달규칙 탭
- **+** 버튼으로 새 규칙 생성
- 대상 앱 선택 → 제목 키워드 입력 → 내용 키워드 입력
- AND/OR 모드 및 정규식 옵션 선택
- **전송 방식 설정**:
  - SMS 전송 ON/OFF → 수신 번호 및 메시지 템플릿 입력
  - URL 호출 ON/OFF → GET/POST 선택, URL 주소 입력, POST시 Body 입력
- 규칙별 ON/OFF 토글로 활성화 관리
- 규칙을 탭하여 편집, 삭제 아이콘으로 삭제

### 4. 알림기록 탭
- 수신된 모든 알림을 날짜별로 그룹 표시 ("오늘" / "YYYY-MM-DD")
- 각 알림에 앱 아이콘, 앱 이름, 시간, 제목, 내용 표시
- 앱별 필터링으로 특정 앱 알림만 보기
- **항목을 길게 누르면** 상세 팝업 표시:
  - 앱 아이콘, 앱 이름, 패키지명, 시간, 제목, 내용, 이미지
  - **"알림제외" 버튼**: 해당 앱을 바로 제외 목록에 추가

### 5. 알림제외 탭
- 제외된 앱 목록 확인
- **+** 버튼으로 앱 피커에서 직접 추가 가능
- 제거 버튼으로 제외 해제 (다시 알림 기록됨)

## 기술 스택

| 구분 | 기술 |
|------|------|
| 언어 | Kotlin |
| UI | Jetpack Compose + Material 3 |
| 네비게이션 | 하단 탭 (NavigationBar) + Navigation Compose |
| 아키텍처 | MVVM |
| DI | Hilt (Dagger) |
| DB | Room (SQLite) |
| 이미지 로딩 | Coil 3, Accompanist DrawablePainter |
| HTTP | java.net.HttpURLConnection (URL 호출) |
| 광고 | Google AdMob (배너 + 앱 오프닝) |
| 백그라운드 | ForegroundService, WorkManager |
| 빌드 | Gradle Kotlin DSL, AGP 8.7.3 |

## 프로젝트 구조

```
app/src/main/java/com/poke86/msg/
├── MsgApplication.kt                    # Application 클래스
├── ads/
│   └── AppOpenAdManager.kt             # 앱 오프닝 광고 관리
├── data/
│   ├── local/                           # Room DB, DAO, Entity
│   └── repository/                      # Repository 패턴
├── di/                                  # Hilt DI 모듈
├── matching/RuleMatcher.kt              # 알림-규칙 매칭 엔진
├── messaging/
│   ├── MessageSender.kt                 # 발송 인터페이스 (확장 포인트)
│   ├── SmsMessageSender.kt              # SMS 구현체
│   ├── UrlCaller.kt                     # HTTP GET/POST URL 호출
│   └── TemplateEngine.kt                # 템플릿 변수 치환
├── service/
│   ├── NotificationMonitorService.kt    # 알림 수신 서비스 (확장 데이터 캡처, 재스캔)
│   ├── AppForegroundService.kt          # 포그라운드 서비스
│   ├── BootReceiver.kt                  # 재부팅 자동 시작
│   └── worker/                          # 헬스체크, 데이터 정리
├── ui/
│   ├── navigation/AppNavigation.kt      # 하단 4탭 네비게이션
│   ├── home/                            # 홈 (서비스 상태, 통계, 백업/복원)
│   ├── rules/                           # 전달규칙 (목록, 편집)
│   ├── notifications/                   # 알림기록 (날짜 헤더, 상세 팝업)
│   ├── excluded/                        # 알림제외 (블랙리스트)
│   └── components/                      # 공통 컴포넌트 (AppPickerDialog, AdBanner 등)
└── util/                                # 유틸리티 (RuleBackupManager 등)
```

## 라이선스

이 프로젝트는 개인 프로젝트입니다.
