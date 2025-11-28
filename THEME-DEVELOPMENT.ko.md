# Obsidian 테마 개발 가이드

이 문서는 Obsidian 커스텀 테마를 처음부터 만드는 방법을 안내합니다.

## 목차

- [시작하기](#시작하기)
- [기본 구조](#기본-구조)
- [CSS 변수](#css-변수)
- [테마 기능](#테마-기능)
- [고급 기법](#고급-기법)
- [배포](#배포)

## 시작하기

### 테마란?

Obsidian 테마는 CSS 파일로 Obsidian의 외관을 변경합니다.
- 색상, 폰트, 간격, 애니메이션 등 커스터마이징
- 다크/라이트 모드 지원
- 플러그인 UI 스타일링

### 사전 요구사항

```
- CSS 기본 지식
- Obsidian 사용 경험
- 개발자 도구 사용 능력 (Ctrl+Shift+I)
```

## 기본 구조

### 필수 파일

```
my-theme/
├── theme.css           # 메인 CSS 파일 (필수)
├── manifest.json       # 테마 메타데이터 (필수)
└── screenshot.png      # 스크린샷 (선택, 권장)
```

### manifest.json

```json
{
  "name": "My Awesome Theme",
  "version": "1.0.0",
  "minAppVersion": "0.16.0",
  "author": "Your Name",
  "authorUrl": "https://yourwebsite.com"
}
```

### 기본 theme.css 구조

```css
/*
  Theme Name: My Awesome Theme
  Author: Your Name
  Version: 1.0.0
*/

/* ============================================ */
/* 다크 모드 설정 */
/* ============================================ */
.theme-dark {
  /* 기본 색상 */
  --background-primary: #1e1e1e;
  --background-secondary: #252525;
  --background-modifier-border: #424242;
  --text-normal: #dcddde;
  --text-muted: #999;
  --text-faint: #666;

  /* 강조 색상 */
  --interactive-accent: #7c3aed;
  --interactive-accent-hover: #8b5cf6;
}

/* ============================================ */
/* 라이트 모드 설정 */
/* ============================================ */
.theme-light {
  /* 기본 색상 */
  --background-primary: #ffffff;
  --background-secondary: #f5f5f5;
  --background-modifier-border: #ddd;
  --text-normal: #2e3338;
  --text-muted: #666;
  --text-faint: #999;

  /* 강조 색상 */
  --interactive-accent: #7c3aed;
  --interactive-accent-hover: #8b5cf6;
}

/* ============================================ */
/* 폰트 설정 */
/* ============================================ */
:root {
  --font-text: 'Pretendard', -apple-system, sans-serif;
  --font-monospace: 'JetBrains Mono', 'Courier New', monospace;
  --font-interface: 'Pretendard', -apple-system, sans-serif;
}

/* ============================================ */
/* 커스텀 스타일 */
/* ============================================ */

/* 타이틀바 */
.titlebar {
  background-color: var(--background-secondary);
}

/* 사이드바 */
.workspace-leaf-content {
  background-color: var(--background-primary);
}

/* 에디터 */
.markdown-source-view,
.markdown-preview-view {
  font-family: var(--font-text);
  line-height: 1.8;
}

/* 헤딩 */
.markdown-preview-view h1,
.cm-header-1 {
  font-size: 2em;
  font-weight: 700;
  color: var(--interactive-accent);
}
```

## CSS 변수

### 주요 색상 변수

```css
/* 배경 */
--background-primary          /* 메인 배경 */
--background-primary-alt       /* 대체 배경 */
--background-secondary         /* 사이드바 배경 */
--background-secondary-alt     /* 사이드바 대체 배경 */
--background-modifier-border   /* 테두리 */
--background-modifier-hover    /* 호버 배경 */
--background-modifier-active   /* 활성 배경 */

/* 텍스트 */
--text-normal                  /* 일반 텍스트 */
--text-muted                   /* 흐린 텍스트 */
--text-faint                   /* 매우 흐린 텍스트 */
--text-on-accent               /* 강조 위 텍스트 */

/* 강조 색상 */
--interactive-accent           /* 메인 강조 색 */
--interactive-accent-hover     /* 강조 호버 */
--interactive-success          /* 성공 */
--interactive-accent-rgb       /* RGB 값 */

/* 링크 */
--link-color                   /* 링크 색상 */
--link-color-hover             /* 링크 호버 */
--link-external-color          /* 외부 링크 */

/* 태그 */
--tag-color                    /* 태그 색상 */
--tag-background               /* 태그 배경 */
```

### 크기/간격 변수

```css
/* 폰트 크기 */
--font-smallest: 0.8em;
--font-smaller: 0.875em;
--font-small: 0.933em;
--font-ui-small: 13px;
--font-ui-medium: 14px;
--font-ui-larger: 16px;

/* 간격 */
--size-2-1: 2px;
--size-2-2: 4px;
--size-2-3: 6px;
--size-4-1: 4px;
--size-4-2: 8px;
--size-4-3: 12px;
--size-4-4: 16px;
--size-4-5: 20px;
--size-4-6: 24px;

/* 둥근 모서리 */
--radius-s: 4px;
--radius-m: 8px;
--radius-l: 10px;
--radius-xl: 16px;
```

### 폰트 변수

```css
/* 폰트 패밀리 */
--font-text                    /* 본문 폰트 */
--font-text-theme              /* 테마 폰트 */
--font-monospace               /* 코드 폰트 */
--font-interface               /* UI 폰트 */
--font-interface-theme         /* UI 테마 폰트 */

/* 폰트 굵기 */
--font-thin: 100;
--font-extralight: 200;
--font-light: 300;
--font-normal: 400;
--font-medium: 500;
--font-semibold: 600;
--font-bold: 700;
--font-extrabold: 800;
--font-black: 900;
```

## 테마 기능

### 1. 커스텀 폰트 추가

```css
/* 웹폰트 로드 */
@font-face {
  font-family: 'CustomFont';
  src: url('https://fonts.googleapis.com/css2?family=Noto+Sans+KR');
}

/* 로컬 폰트 */
@font-face {
  font-family: 'MyFont';
  src: local('MyFont'),
       url('fonts/MyFont.woff2') format('woff2');
}

/* 적용 */
:root {
  --font-text: 'CustomFont', sans-serif;
}
```

### 2. 헤딩 스타일링

```css
/* 모든 헤딩 기본 스타일 */
.markdown-preview-view h1,
.markdown-preview-view h2,
.markdown-preview-view h3,
.markdown-preview-view h4,
.markdown-preview-view h5,
.markdown-preview-view h6 {
  font-weight: 700;
  margin-top: 1.5em;
  margin-bottom: 0.5em;
}

/* 개별 헤딩 */
.markdown-preview-view h1 {
  font-size: 2.2em;
  color: var(--interactive-accent);
  border-bottom: 2px solid var(--interactive-accent);
  padding-bottom: 0.3em;
}

.markdown-preview-view h2 {
  font-size: 1.8em;
  color: var(--text-normal);
  border-bottom: 1px solid var(--background-modifier-border);
  padding-bottom: 0.2em;
}

/* 에디터 모드 (CodeMirror) */
.cm-header-1 { font-size: 2.2em; }
.cm-header-2 { font-size: 1.8em; }
.cm-header-3 { font-size: 1.5em; }
```

### 3. 코드 블록 스타일링

```css
/* 인라인 코드 */
code {
  background-color: var(--background-secondary);
  color: var(--text-accent);
  padding: 2px 6px;
  border-radius: var(--radius-s);
  font-family: var(--font-monospace);
  font-size: 0.9em;
}

/* 코드 블록 */
pre code {
  display: block;
  padding: 1em;
  background-color: var(--background-secondary);
  border-radius: var(--radius-m);
  overflow-x: auto;
  line-height: 1.6;
}

/* 언어 태그 */
.copy-code-button {
  opacity: 0;
  transition: opacity 0.2s;
}

pre:hover .copy-code-button {
  opacity: 1;
}
```

### 4. 리스트 스타일링

```css
/* 순서 없는 리스트 */
.markdown-preview-view ul {
  list-style-type: none;
  padding-left: 2em;
}

.markdown-preview-view ul > li::before {
  content: '•';
  color: var(--interactive-accent);
  font-weight: bold;
  display: inline-block;
  width: 1em;
  margin-left: -1em;
}

/* 순서 있는 리스트 */
.markdown-preview-view ol {
  counter-reset: list-counter;
  list-style: none;
  padding-left: 2em;
}

.markdown-preview-view ol > li {
  counter-increment: list-counter;
}

.markdown-preview-view ol > li::before {
  content: counter(list-counter) '.';
  color: var(--interactive-accent);
  font-weight: bold;
  margin-right: 0.5em;
}

/* 체크박스 */
.markdown-preview-view .task-list-item-checkbox {
  width: 1.2em;
  height: 1.2em;
  border: 2px solid var(--interactive-accent);
  border-radius: var(--radius-s);
}

.markdown-preview-view .task-list-item-checkbox:checked {
  background-color: var(--interactive-accent);
}
```

### 5. 링크 스타일링

```css
/* 내부 링크 */
a.internal-link {
  color: var(--interactive-accent);
  text-decoration: none;
  border-bottom: 1px solid transparent;
  transition: border-color 0.2s;
}

a.internal-link:hover {
  border-bottom-color: var(--interactive-accent);
}

/* 외부 링크 */
a.external-link {
  color: var(--link-external-color);
  text-decoration: none;
}

a.external-link:hover {
  text-decoration: underline;
}

/* 깨진 링크 */
a.internal-link.is-unresolved {
  color: var(--text-muted);
  opacity: 0.7;
}
```

### 6. 테이블 스타일링

```css
/* 테이블 */
.markdown-preview-view table {
  border-collapse: collapse;
  width: 100%;
  margin: 1em 0;
}

.markdown-preview-view th {
  background-color: var(--interactive-accent);
  color: var(--text-on-accent);
  font-weight: 600;
  padding: 0.75em;
  text-align: left;
}

.markdown-preview-view td {
  padding: 0.75em;
  border-bottom: 1px solid var(--background-modifier-border);
}

.markdown-preview-view tr:hover {
  background-color: var(--background-modifier-hover);
}

/* 줄무늬 */
.markdown-preview-view tr:nth-child(even) {
  background-color: var(--background-secondary-alt);
}
```

### 7. 블록인용 스타일링

```css
.markdown-preview-view blockquote {
  border-left: 4px solid var(--interactive-accent);
  margin: 1em 0;
  padding: 0.5em 1em;
  background-color: var(--background-secondary);
  border-radius: var(--radius-s);
  font-style: italic;
}

.markdown-preview-view blockquote p {
  margin: 0.5em 0;
}
```

## 고급 기법

### 1. 사이드바 커스터마이징

```css
/* 왼쪽 사이드바 */
.workspace-split.mod-left-split {
  background-color: var(--background-secondary);
}

/* 파일 탐색기 */
.nav-folder-title,
.nav-file-title {
  padding: 4px 8px;
  border-radius: var(--radius-s);
  transition: background-color 0.2s;
}

.nav-folder-title:hover,
.nav-file-title:hover {
  background-color: var(--background-modifier-hover);
}

.nav-file-title.is-active {
  background-color: var(--interactive-accent);
  color: var(--text-on-accent);
}

/* 폴더 아이콘 */
.nav-folder-collapse-indicator {
  color: var(--interactive-accent);
}
```

### 2. 탭 스타일링

```css
/* 탭 컨테이너 */
.workspace-tab-header-container {
  background-color: var(--background-secondary);
  border-bottom: 1px solid var(--background-modifier-border);
}

/* 개별 탭 */
.workspace-tab-header {
  padding: 8px 16px;
  border-radius: var(--radius-s) var(--radius-s) 0 0;
  transition: background-color 0.2s;
}

.workspace-tab-header:hover {
  background-color: var(--background-modifier-hover);
}

.workspace-tab-header.is-active {
  background-color: var(--background-primary);
  border-bottom: 2px solid var(--interactive-accent);
}

/* 탭 제목 */
.workspace-tab-header-inner-title {
  font-weight: 500;
}
```

### 3. 모달 스타일링

```css
/* 모달 배경 */
.modal-container {
  background-color: rgba(0, 0, 0, 0.5);
  backdrop-filter: blur(4px);
}

/* 모달 */
.modal {
  background-color: var(--background-primary);
  border-radius: var(--radius-l);
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.3);
  padding: 2em;
}

/* 모달 제목 */
.modal-title {
  font-size: 1.5em;
  font-weight: 700;
  color: var(--interactive-accent);
  margin-bottom: 1em;
}

/* 모달 버튼 */
.modal button {
  background-color: var(--interactive-accent);
  color: var(--text-on-accent);
  border: none;
  padding: 0.5em 1.5em;
  border-radius: var(--radius-m);
  cursor: pointer;
  transition: background-color 0.2s;
}

.modal button:hover {
  background-color: var(--interactive-accent-hover);
}
```

### 4. 애니메이션 추가

```css
/* 페이드 인 */
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

.workspace-leaf {
  animation: fadeIn 0.3s ease-in-out;
}

/* 슬라이드 인 */
@keyframes slideIn {
  from {
    transform: translateY(-10px);
    opacity: 0;
  }
  to {
    transform: translateY(0);
    opacity: 1;
  }
}

.modal {
  animation: slideIn 0.2s ease-out;
}

/* 호버 효과 */
.nav-file-title {
  transition: transform 0.2s, background-color 0.2s;
}

.nav-file-title:hover {
  transform: translateX(4px);
}
```

### 5. 반응형 디자인

```css
/* 모바일 */
@media (max-width: 768px) {
  :root {
    --font-text-size: 14px;
  }

  .markdown-preview-view h1 {
    font-size: 1.8em;
  }

  .workspace-split.mod-left-split,
  .workspace-split.mod-right-split {
    width: 100% !important;
  }
}

/* 태블릿 */
@media (max-width: 1024px) {
  :root {
    --font-text-size: 15px;
  }
}
```

### 6. 다크 모드 전환 효과

```css
/* 부드러운 색상 전환 */
* {
  transition: background-color 0.3s ease,
              color 0.3s ease,
              border-color 0.3s ease;
}

/* 특정 요소 제외 */
.markdown-source-view,
.cm-editor {
  transition: none;
}
```

## 배포

### 1. 로컬 테스트

```bash
# vault의 테마 폴더에 복사
cp -r my-theme /path/to/vault/.obsidian/themes/

# Obsidian 설정 → 테마에서 선택
```

### 2. GitHub 저장소 생성

```bash
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/username/my-theme.git
git push -u origin main
```

### 3. 릴리스 생성

```bash
git tag -a 1.0.0 -m "Release 1.0.0"
git push origin 1.0.0
```

### 4. 커뮤니티 테마 등록

1. [obsidian-releases](https://github.com/obsidianmd/obsidian-releases) Fork
2. `community-css-themes.json`에 추가:
   ```json
   {
     "name": "My Awesome Theme",
     "author": "Your Name",
     "repo": "username/my-theme",
     "screenshot": "screenshot.png",
     "modes": ["dark", "light"]
   }
   ```
3. Pull Request 생성

## 유용한 도구

### 개발자 도구 사용법

```
1. Ctrl/Cmd + Shift + I로 개발자 도구 열기
2. Elements 탭에서 요소 검사
3. Styles 패널에서 CSS 실시간 수정
4. Computed 탭에서 적용된 스타일 확인
```

### CSS 스니펫 활용

```css
/* .obsidian/snippets/custom.css */
/* 테마를 수정하지 않고 개별 스타일 추가 */

.custom-class {
  /* 커스텀 스타일 */
}
```

## 모범 사례

1. **변수 활용**: 하드코딩된 값 대신 CSS 변수 사용
2. **일관성**: 일관된 간격과 색상 팔레트 유지
3. **성능**: 과도한 애니메이션과 효과 자제
4. **접근성**: 충분한 색상 대비 유지
5. **호환성**: 다양한 플러그인과의 호환성 테스트

## 참고 리소스

- [Obsidian CSS 가이드](https://docs.obsidian.md/Themes/App+themes/Build+a+theme)
- [커뮤니티 테마](https://github.com/obsidianmd/obsidian-releases/blob/master/community-css-themes.json)
- [CSS 변수 참조](https://docs.obsidian.md/Reference/CSS+variables/CSS+variables)
