# Recent Files for Obsidian

이 플러그인은 가장 최근에 열었던 파일 목록을 사이드바에 표시합니다.
선택적으로 특정 경로, frontmatter 태그 또는 북마크된 파일을 제외할 수 있습니다.

## 스크린샷

![sidebar](https://raw.githubusercontent.com/tgrosinger/recent-files-obsidian/main/resources/screenshots/sidebar.png)

## 설치 방법

### 커뮤니티 플러그인에서 설치
1. Obsidian 설정 열기 (`Ctrl/Cmd + ,`)
2. 좌측 메뉴에서 **커뮤니티 플러그인** 선택
3. **제한 모드 끄기** (처음인 경우)
4. **찾아보기** 클릭 후 "Recent Files" 검색
5. **설치** → **활성화**

### 수동 설치
1. [최신 릴리스](https://github.com/tgrosinger/recent-files-obsidian/releases) 다운로드
2. `vault/.obsidian/plugins/recent-files-obsidian/` 폴더에 압축 해제
3. Obsidian 설정에서 플러그인 활성화

## 주요 기능

### 1. 기본 사용법

플러그인을 활성화하면 좌측 사이드바에 시계 아이콘(🕐)이 나타납니다.

**파일 열기**
- **클릭**: 파일 열기
- **Ctrl/Cmd + 클릭**: 새 탭에서 열기
- **우클릭**: 컨텍스트 메뉴 표시

**파일 미리보기**
- **마우스 호버**: 파일 내용 미리보기 (Page Preview 설정 필요)
- **Ctrl/Cmd + 호버**: 강제 미리보기

### 2. 드래그 앤 드롭

**링크 삽입**
```
파일 목록에서 드래그 → 에디터에 드롭
결과: [[파일이름]] 링크 생성
```

**특정 창에서 열기**
```
파일 목록에서 드래그 → 탭 헤더에 드롭
결과: 해당 탭에서 파일 열림
```

**파일 이동**
```
파일 목록에서 드래그 → 파일 탐색기 폴더에 드롭
결과: 파일이 해당 폴더로 이동
```

### 3. 목록 관리

**목록 초기화**
1. Recent Files 패널 우상단 메뉴 클릭 (⋮)
2. "Clear list" 선택

**커맨드 팔레트**
- `Ctrl/Cmd + P` → "Recent Files: Open" 입력

## 설정 가이드

플러그인 설정: `설정 → 커뮤니티 플러그인 → Recent Files`

### 1. 제외 경로 패턴 (Omitted pathname patterns)

정규식을 사용하여 특정 경로의 파일을 목록에서 제외합니다. **한 줄에 하나의 패턴**을 입력합니다.

**예제:**
```regex
^daily/           # daily 폴더의 모든 파일 제외
\.png$            # PNG 이미지 파일 제외
template          # 경로에 "template"이 포함된 모든 파일 제외
archive/.*backup  # archive 폴더의 backup 파일 제외
```

**실전 활용 예시:**
```regex
^templates/
^daily/
^archive/
\.excalidraw$
\.canvas$
```

### 2. 제외 태그 패턴 (Omitted frontmatter-tag patterns)

frontmatter의 태그를 기준으로 파일을 제외합니다.

**frontmatter 예시:**
```yaml
---
tags: project/archived
---
```

**제외 패턴:**
```regex
archived          # "archived" 태그가 포함된 파일 제외
^private          # "private"로 시작하는 태그 제외
project/done      # 완료된 프로젝트 제외
```

**여러 태그 처리:**
```yaml
---
tags:
  - work
  - archive
---
```
이 경우 `archive` 패턴으로 제외됩니다.

### 3. 북마크 제외 (Omit bookmarked files)

토글을 켜면 Obsidian 북마크에 추가된 파일이 최근 목록에서 제외됩니다.

**사용 시나리오:**
- 자주 사용하는 파일은 북마크로 관리
- 최근 파일 목록은 일시적인 작업용 파일만 표시

### 4. 업데이트 트리거 (Update list when file is)

**Opened (파일 열기)**
- 파일을 열 때마다 목록 업데이트
- 빠른 탐색 시 유용
- **기본값**

**Changed (파일 편집)**
- 파일을 실제로 수정했을 때만 업데이트
- 읽기 전용 파일은 목록에 추가되지 않음
- 작업 중인 파일만 추적하고 싶을 때 유용

### 5. 최대 파일 수 (List length)

목록에 표시할 최대 파일 개수를 설정합니다.

- **기본값**: 50개
- **권장**: 20-100개
- 숫자가 클수록 메모리 사용량 증가

## 다른 플러그인 지원

### Front Matter Title 플러그인 연동

[Front Matter Title 플러그인](https://github.com/snezhig/obsidian-front-matter-title)을 사용하는 경우, explorer 모듈을 활성화하면 Recent Files 사이드바에서 노트의 frontmatter 제목으로 파일 이름이 표시됩니다.

**설정 방법:**
1. Front Matter Title 플러그인 설치
2. 플러그인 설정에서 **Explorer** 모듈 활성화
3. frontmatter에 title 추가:
   ```yaml
   ---
   title: 회의록 - 2025년 1월
   ---
   ```
4. Recent Files 목록에서 파일명 대신 "회의록 - 2025년 1월" 표시

## 활용 팁

### 워크플로우 예시

**시나리오 1: 프로젝트 작업 시**
```
1. 프로젝트 관련 파일만 보고 싶음
2. 설정에서 제외 경로 추가:
   ^daily/
   ^archive/
   ^templates/
3. 북마크 제외 활성화
4. 결과: 현재 작업 중인 프로젝트 파일만 표시
```

**시나리오 2: 일일 노트 제외**
```
1. 매일 자동 생성되는 일일 노트가 목록을 채움
2. 설정에서 제외 경로 추가:
   ^daily/
   또는 태그로 제외:
   daily-note
3. 결과: 실제 작업 파일만 깔끔하게 표시
```

**시나리오 3: 읽기만 한 파일 제외**
```
1. 업데이트 트리거를 "Changed"로 변경
2. 결과: 실제로 편집한 파일만 목록에 추가
```

### 키보드 단축키 설정

Recent Files를 더 빠르게 열기 위해 단축키를 설정할 수 있습니다:

1. `설정 → 단축키`
2. "Recent Files" 검색
3. "Recent Files: Open" 명령에 단축키 할당 (예: `Ctrl+Shift+R`)

## 문제 해결

### 플러그인이 보이지 않아요
1. 커뮤니티 플러그인 제한 모드가 꺼져있는지 확인
2. 플러그인이 활성화되어 있는지 확인
3. Obsidian 재시작 (`Ctrl/Cmd + R`)

### 최근 파일 목록이 업데이트되지 않아요
1. 파일이 제외 패턴에 걸리는지 확인
2. 북마크 제외 설정 확인
3. 업데이트 트리거 설정 확인 (Opened vs Changed)

### 정규식 패턴이 작동하지 않아요
- 정규식 문법 확인: [MDN 정규식 가이드](https://developer.mozilla.org/ko/docs/Web/JavaScript/Guide/Regular_Expressions)
- 역슬래시 이스케이프: `\.png$` (O) vs `.png$` (X)
- 개발자 콘솔에서 에러 확인 (`Ctrl+Shift+I`)

### 파일 이름이 길어서 잘려요
- Obsidian 테마 설정 확인
- `styles.css`에서 커스텀 스타일 추가 가능

### 드래그 앤 드롭이 작동하지 않아요
- 다른 플러그인과 충돌 가능성 확인
- Obsidian 버전 확인 (최소 0.16.3 이상)
- 플러그인 재활성화 시도

## 개발

### 프로젝트 구조

```
recent-files-obsidian/
├── main.ts              # 메인 플러그인 소스 코드
├── styles.css           # 스타일시트
├── manifest.json        # Obsidian 플러그인 매니페스트
├── package.json         # 프로젝트 설정 및 의존성
├── tsconfig.json        # TypeScript 설정
├── esbuild.config.mjs   # esbuild 번들러 설정
├── .eslintrc.js         # ESLint 설정
└── versions.json        # 버전 호환성 정보
```

### 기술 스택

- **언어**: TypeScript
- **번들러**: esbuild
- **린터**: ESLint
- **패키지 매니저**: npm 또는 yarn

### 빌드 명령어

```bash
# 의존성 설치
npm install    # 또는 yarn install

# 개발 모드 (watch 모드로 자동 리빌드)
npm run dev    # 또는 yarn dev

# 프로덕션 빌드
npm run build  # 또는 yarn build

# ESLint 검사 및 자동 수정
npm run eslint # 또는 yarn eslint
```

### 빌드 과정

1. **프로덕션 빌드** (`npm run build`):
   - TypeScript 타입 검사 (`tsc -noEmit -skipLibCheck`)
   - esbuild를 통한 번들링 및 minification
   - `main.js` 파일 생성 (최적화됨)

2. **개발 빌드** (`npm run dev`):
   - watch 모드로 파일 변경 감지
   - 변경 시 자동으로 리빌드
   - 소스맵 포함 (디버깅용)

### 로컬 개발 환경 설정

**Windows (PowerShell):**
```powershell
# 1. vault의 플러그인 폴더로 이동
cd E:\Obsidian\YourVault\.obsidian\plugins

# 2. 저장소 클론 또는 복사
git clone <repo-url> recent-files-obsidian
# 또는 프로젝트 폴더 복사

# 3. 프로젝트 폴더로 이동
cd recent-files-obsidian

# 4. 의존성 설치
npm install

# 5. 개발 모드 시작
npm run dev
```

**macOS / Linux:**
```bash
# 1. vault의 플러그인 폴더로 이동
cd ~/Documents/Obsidian/YourVault/.obsidian/plugins

# 2. 심볼릭 링크 생성 (권장)
ln -s ~/dev/recent-files-obsidian recent-files-obsidian

# 3. 프로젝트 폴더로 이동
cd recent-files-obsidian

# 4. 의존성 설치
npm install

# 5. 개발 모드 시작
npm run dev
```

**Obsidian에서 활성화:**
1. Obsidian 설정 열기
2. 커뮤니티 플러그인 → Recent Files 활성화
3. 코드 수정 후 `Ctrl/Cmd + R`로 새로고침

### 개발 팁

- **코드 수정 → 자동 빌드**: `npm run dev` 실행 중이면 파일 저장 시 자동 리빌드
- **Obsidian 새로고침**: `Ctrl/Cmd + R` 또는 플러그인 껐다 켜기
- **개발자 도구**: `Ctrl/Cmd + Shift + I`로 콘솔 확인
- **디버깅**: `console.log()` 활용

## 라이선스

GPL-3.0

## 후원

이 플러그인은 무료로 제공되지만, 감사의 표시나 지속적인 개발 지원을 원하신다면 아래 방법으로 후원해 주세요:

[![GitHub Sponsors](https://img.shields.io/github/sponsors/tgrosinger?style=social)](https://github.com/sponsors/tgrosinger)
[![Paypal](https://img.shields.io/badge/paypal-tgrosinger-yellow?style=social&logo=paypal)](https://paypal.me/tgrosinger)
[<img src="https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png" alt="BuyMeACoffee" width="100">](https://www.buymeacoffee.com/tgrosinger)
