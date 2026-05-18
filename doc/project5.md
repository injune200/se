# [프로젝트 VII] 대상 시스템의 요구사항 분석서: CodeBuddy

## 1. 기능 모델링 (Functional Modeling)

시스템이 제공해야 하는 핵심 기능과 사용자의 상호작용을 정의합니다.

### 1.1 유스케이스 다이어그램
<img width="522" height="722" alt="화면 캡처 2026-05-18 141014" src="https://github.com/user-attachments/assets/6043761c-5fe5-4c20-8f17-0c4df809aa3a" />


### 1.2 주요 유스케이스 설명서

| 항목 | 상세 내용 |
| :--- | :--- |
| **유스케이스명** | 실시간 코드 편집 및 AI 리팩토링 (Real-time Code Editing & AI Code Refactoring) |
| **액터(Actor)** | 주 액터: 개발자(Developer)<br>보조 액터: AI 어드바이저 엔진(AI Advisor Engine) |
| **개요** | 개발자가 에디터 상에서 코드를 입력하면 실시간으로 동기화되며, AI가 백그라운드에서 코드를 분석하여 최적화 및 리팩토링 방안을 제안한다. |
| **정상 이벤트 흐름** | 1. 개발자가 프로젝트 워크스페이스에 접속하여 코드를 입력한다.<br>2. 시스템은 변경된 코드 내역을 캡처하여 동기화 세션에 반영한다.<br>3. 시스템이 AI 어드바이저 엔진에 비동기적으로 코드 분석을 요청한다.<br>4. AI 엔진이 코드 최적화 및 리팩토링 제안을 시스템으로 반환한다.<br>5. 개발자 화면에 리팩토링 제안이 표시되며, 개발자가 이를 수락/거절할 수 있다. |
| **확장(Extend)** | AI 코딩 지원 중 '코드 최적화(Code Optimization)' 기능이 조건에 따라 추가로 호출될 수 있다. |

---

## 2. 구조 모델링 (Structural Modeling)

비즈니스 도메인의 명사를 바탕으로 시스템 내부의 정적 구조와 클래스 간의 관계를 정의합니다.

### 2.1 클래스 다이어그램
<img width="526" height="720" alt="화면 캡처 2026-05-18 141107" src="https://github.com/user-attachments/assets/24b25e57-d58a-414f-a0d3-580c47d300ad" />


### 2.2 핵심 클래스 명세
* **사용자 (User / Developer)**: 시스템을 이용하는 개발자 객체입니다. 세션에 접속(`joinSession()`)하고 코드를 편집(`editCode()`)하는 행위를 수행합니다.
* **프로젝트 (Project)**: 작업을 담는 최상위 컨테이너로, 다수의 파일(File)을 포함하는 집합(Aggregation) 관계를 가집니다.
* **에디터 세션 (EditorSession)**: 특정 시점에 접속한 활성 사용자(activeUsers)를 1:N으로 관리하며, 코드 변경 사항을 모든 클라이언트에 브로드캐스트(`broadcastChange()`)하는 책임을 가집니다.
* **파일 (File)**: 실제 코드 데이터(`content`)를 캡슐화하여 보유하며, 세션 내에서 변경될 때마다 내용을 갱신(`updateContent()`)합니다.
* **AI 어드바이저 (AI Advisor)**: 분석 요청이 들어온 코드를 파싱하고(`analyzeCode()`), 향상된 코드 스니펫을 제안(`suggestRefactoring()`)하는 독립적 모듈입니다.
* **Git 서비스 (GitService)**: 작성 완료된 프로젝트를 원격 저장소와 동기화(`syncRepo()`)하거나 PR을 생성(`createPR()`)하는 연동 클래스입니다.

---

## 3. 행위 모델링 (Behavioral Modeling)

객체 간의 시간적 상호작용과 메시지 패싱 흐름을 정의하여 시스템의 동적 행위를 나타냅니다.

### 3.1 순차 다이어그램 (실시간 편집 및 동기화)
<img width="521" height="716" alt="화면 캡처 2026-05-18 141138" src="https://github.com/user-attachments/assets/d42346ef-349e-41ed-890b-3f0ffb016e99" />


### 3.2 흐름(메시지 패싱) 설명
1. **코드 입력 (User Inputs Code)**: 사용자 A가 브라우저의 에디터(`EditorBoundary`)에 소스코드를 타이핑합니다.
2. **변경 캡처 (Capture Change)**: 에디터 뷰는 변경된 텍스트 데이터를 세션 컨트롤러(`SyncController`)로 전달합니다.
3. **비동기 분석 요청 (Asynchronous Analysis Request)**: 컨트롤러는 입력 흐름을 방해하지 않기 위해 백그라운드로 `AIAdvisor`에 분석을 요청합니다.
4. **실시간 브로드캐스트 (Broadcast Changes)**: 컨트롤러는 동시에 웹소켓 채널을 통해 공동 편집 중인 사용자 B의 클라이언트로 변경 사항을 즉각 전파합니다.
5. **AI 제안 반환 및 파일 저장**: `AIAdvisor`가 코드 분석을 마친 후 리팩토링 제안을 컨트롤러로 리턴하면, 제안 사항이 적용됨과 동시에 영구 저장소의 `FileEntity`에 저장됩니다.
6. **UI 업데이트**: 최종적으로 사용자 A와 사용자 B의 에디터 화면(UI)이 동기화 및 업데이트되어 최신 상태를 유지하게 됩니다.
