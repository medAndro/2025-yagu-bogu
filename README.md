# ⚾️ 야구보구 👀

## 📍 프로젝트 소개
야구 팬의 직관 활동을 기록하고 통계화하여 성취감과 경쟁심을 자극하는 **야구 팬 커뮤니티 앱**

"오늘이 몇 번째 직관이었지?", "내가 직관했을 때 우리 팀 승률이 어떻게 되더라?"

**야구보구**는 클릭 한 번으로 구장 방문을 인증하면 경기 정보, 결과, 나의 직관 통계까지 자동으로 기록됩니다. 오늘 구장별 팬 분포를 실시간으로 확인하고, 내 직관 승률과 기록을 한눈에 볼 수 있습니다.

- **팀 구성**: Android 3명 (본인 포함), Backend 4명
- **개발 기간**: 2025년 8월 ~ 진행중
- **담당 역할**: Android 개발 (UI/UX 구현, 네트워크 통신, 이미지 처리)

## 👥 팀원 소개

|<img src="https://github.com/medAndro.png" width="125" />|<img src="https://github.com/ijh1298.png" width="125" />|<img src="https://github.com/jiyuneel.png" width="125" />|<img src="https://github.com/jjunh0.png" width="125" />|<img src="https://github.com/Starlight258.png" width="125" />|<img src="https://github.com/bowook.png" width="125" />|<img src="https://github.com/nourzoo.png" width="125" />|
|:---------:|:---------:|:---------:|:---------:|:---------:|:---------:|:---------:|
|[메다(장지형)](https://github.com/medAndro)|[크림(임준혁)](https://github.com/ijh1298)|[포르(이지윤)](https://github.com/jiyuneel)|[두리(김준호)](https://github.com/jjunh0)|[밍트(김명지)](https://github.com/Starlight258)|[우가(민보욱)](https://github.com/bowook)|[포라(이승연)](https://github.com/nourzoo)|
|Android|Android|Android|Backend|Backend|Backend|Backend|


## 💻 주요 기여 사항

### 1️⃣ 프로필 이미지 업로드 파이프라인 구현

**문제 상황**
- 앱→서버 직접 업로드 방식으로 인한 서버 트래픽 과부하
- 이미지 크롭 기능 부재로 불필요한 배경 이미지가 프로필에 포함
- 고해상도 원본 업로드로 인한 저장 공간 낭비 및 로딩 지연

**기술적 해결**
- **Presigned URL 기반 S3 직접 업로드**: 서버를 경유하지 않고 S3에 직접 업로드하여 네트워크 트래픽 **70% 이상 절감**
- **uCrop 라이브러리 통합**: 원형 가이드 크롭으로 1:1 비율 프로필 이미지 생성
- **이미지 압축 및 리사이징**: 업로드 전 클라이언트에서 최적화하여 저장 공간 절약

**성과**
- 서버 트래픽 부담 감소 및 사용자 업로드 속도 개선
- 일관된 프로필 이미지 UI/UX 제공

### 2️⃣ 채팅 화면 Jetpack Compose 마이그레이션

**문제 상황**
- XML 기반 RecyclerView의 높은 코드 복잡도
- LiveData 기반 상태 관리의 라이프사이클 이슈
- KMP(Kotlin Multiplatform) 전환 불가능한 레거시 구조

**기술적 해결**
- **선언형 UI 전환**: RecyclerView → LazyColumn 마이그레이션으로 UI 로직 코드 **30% 감소**
- **반응형 상태 관리**: LiveData → Flow(StateFlow/SharedFlow) 전환으로 코루틴 기반 비동기 처리 구현

**성과**
- 코드 가독성 및 유지보수성 향상
- Compose Multiplatform 호환 구조로 향후 크로스 플랫폼 확장 가능성 확보

### 3️⃣ 실시간 채팅 폴링 시스템 구축 및 동시성 제어

**문제 상황**
- 비동기 폴링 환경에서 다수의 코루틴이 동시에 채팅 기록 State를 수정하는 Race Condition 발생
- 실시간 대량 메시지 추가 시 전체 리스트 리렌더링으로 인한 UI 끊김(Jank) 현상

**기술적 해결**
- **Coroutine Mutex 기반 동시성 제어**: 공유 자원 접근 시 Mutex로 Critical Section 보호, 데이터 정합성 보장
- **DiffUtil을 활용한 효율적 UI 갱신**: 변경된 메시지만 부분 갱신하여 전체 렌더링 비용 최소화

**성과**
- 데이터 경합 상태 제거 및 안정적인 실시간 채팅 구현
- 메모리 및 CPU 사용률 최적화로 부드러운 UX 제공

### 4️⃣ 팀 응원 기능 개발 (클릭 배칭 시스템)

**문제 상황**
- 응원 버튼 연타 시 매 클릭마다 API 호출로 서버 부하 급증
- 앱 생명주기 변경 시 미전송 데이터 유실 또는 중복 요청 발생

**기술적 해결**
- **시간 기반 클릭 배칭 전략**: 
  - 클릭 발생 시 `StateHolder`에서 로컬 카운트만 증가
  - 배칭 Job이 없을 때만 5초 타이머 시작 (`delay(5000L)`)
  - 타이머 종료 시 누적된 클릭 횟수를 `LikeBatchRequest`로 일괄 전송
- Lifecycle 인식형 데이터 보존:
  - ViewModel.onCleared() 오버라이드로 화면 종료 시점 감지
  - 미전송 데이터가 있을 경우 CoroutineScope(Dispatchers.IO)로 백그라운드 전송
  - viewModelScope 종료 후에도 데이터 유실 방지

**성과**
- 서버 API 호출 횟수 획기적 감소로 백엔드 부하 완화
- 데이터 유실 없는 안정적인 실시간 집계 구현

## 📱 주요 화면

- 🏟️ **홈**: 위치 기반 경기장 자동 인증, 오늘 팬 현황, 승리요정 랭킹
- 💬 **현장톡**: 실시간 채팅으로 같은 경기장 팬들과 소통
- 📊 **통계**: 직관 승률, 구장별 방문 횟수, 상세 통계 시각화
- 🎟️ **직관 내역**: 경기별 직관 기록 조회 및 필터링
- 👤 **설정**: 프로필 이미지 업로드 및 내 정보 관리

### Android
- **Language**: Kotlin
- **UI**: Jetpack Compose, XML
- **Architecture**: MVVM
- **Async**: Coroutines, Flow
- **DI**: Hilt
- **Network**: Retrofit2, OkHttp
- **Image**: Coil, uCrop
- **Storage**: DataStore

### Backend
- **Language**: Java 
- **Framework**: Spring Boot
- **Database**: MySQL
- **Storage**: AWS S3
- **ORM**: JPA, QueryDSL

## 📂 프로젝트 구조
```bash
app/
├── data/ # 데이터 레이어 (Repository, DataSource)
├── domain/ # 도메인 레이어 (UseCase, Entity)
├── ui/ # UI 레이어 (Compose, ViewModel)
└── di/ # 의존성 주입 모듈
```
## 🎯 프로젝트를 통해 배운 점

- **성능 최적화**: Presigned URL을 활용한 네트워크 트래픽 절감 경험
- **동시성 제어**: Coroutine Mutex를 활용한 Race Condition 해결 경험
- **아키텍처 설계**: XML에서 Jetpack Compose로의 마이그레이션을 통한 선언형 UI 이해
- **협업 역량**: Android 3명, Backend 4명과의 협업을 통한 커뮤니케이션 및 문서화 역량 향상
- **문제 해결 능력**: 실시간 데이터 처리 및 배칭 시스템 구현을 통한 성능 최적화 경험

## 🔗 링크
- [GitHub Repository (Team)](https://github.com/woowacourse-teams/2025-yagu-bogu)

## 💫 규칙들

### 🧑‍🤝‍🧑 TEAM CULTURE

- **🐲 친절하게 말해주세용**
  → 논의가 과열되었을 때 용용체로 대화해용

- 🙉 **너의 목소리가 안 들려**
  → 무엇이든지 말해주세요

- **🍗 반마리보다 🐓한마리예요**
  → 회의는 존댓말로 진행해요

- **🤗 되면 대면해요**
  → 대면 협업을 우선해요

---

### ⏰ WORKFLOW

- **📅 데일리 스크럼: 데일리 미팅 끝나고 바로**
  → 어제 한 일과 오늘 할 일을 공유해요

- 🍚 **수요일 == 외식 데이**
  → 다 같이 밥 먹어요

---

### 🗣️ COMMUNICATION

- 🤔 **설득하거나 설득 당하거나**
  → 논의할 때 최소한의 입장을 가져요

- ✋ **세상에서 바보같은 질문은 없다**
  → 묻지 않고 넘기는 게 더 위험해요

- ⌛ **일과 시간은 팀과 시간을**
  → 일과 시간은 팀과 함께 움직여요

- ❤️ **좋아요가 좋아요**
  → 긍정적인 리액션이 좋아요

---

### 📚 KNOWLEDGE SHARING

- ⬆️ **우리 함께 레벨업**
  → 알게 된 지식은 혼자 갖지 말고 공유해요

- 🌱 **기록하면 새록새록**
  → 모든 일을 기록해요

