# Recent Files for Obsidian

이 플러그인은 가장 최근에 열었던 파일 목록을 사이드바에 표시합니다.
선택적으로 특정 경로, frontmatter 태그 또는 북마크된 파일을 제외할 수 있습니다.

그게 전부입니다!

파일 탐색기 뷰와 마찬가지로 다음 기능을 사용할 수 있습니다:

* 항목을 클릭하여 열기, Ctrl+클릭으로 새 창에서 열기, 우클릭으로 메뉴 표시
* 항목을 에디터로 드래그하여 링크 삽입, 헤더로 드래그하여 특정 창에서 열기, 파일 탐색기 폴더로 드래그하여 파일 이동
* 호버 또는 Ctrl+호버로 콘텐츠 미리보기 ("Page Preview" 설정의 "Recent Files" 토글로 구성)

## 스크린샷

![sidebar](https://raw.githubusercontent.com/tgrosinger/recent-files-obsidian/main/resources/screenshots/sidebar.png)

## 설정 옵션

| 옵션 | 설명 |
|------|------|
| 제외 경로 | 목록에서 제외할 파일 경로 패턴 (정규식 지원) |
| 제외 태그 | 목록에서 제외할 frontmatter 태그 |
| 북마크 제외 | 북마크된 파일을 목록에서 제외 |
| 업데이트 트리거 | 파일 열기 또는 파일 편집 시 목록 업데이트 |
| 최대 파일 수 | 표시할 최근 파일의 최대 개수 (기본값: 50) |

## 다른 플러그인 지원

[Front Matter Title 플러그인](https://github.com/snezhig/obsidian-front-matter-title)을 사용하는 경우, explorer 모듈을 활성화하면 Recent Files 사이드바에서 노트의 frontmatter 제목으로 파일 이름이 표시됩니다.

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
- **패키지 매니저**: Yarn

### 빌드 명령어

```bash
# 의존성 설치
yarn install

# 개발 모드 (watch 모드로 자동 리빌드)
yarn dev

# 프로덕션 빌드
yarn build

# ESLint 검사 및 자동 수정
yarn eslint
```

### 빌드 과정

1. `yarn build` 실행 시:
   - TypeScript 타입 검사 (`tsc -noEmit -skipLibCheck`)
   - esbuild를 통한 번들링 및 minification
   - `main.js` 파일 생성

2. `yarn dev` 실행 시:
   - watch 모드로 파일 변경 감지
   - 변경 시 자동으로 리빌드
   - 소스맵 포함 (디버깅용)

### 로컬 개발 환경 설정

1. 이 저장소를 Obsidian vault의 `.obsidian/plugins/recent-files-obsidian` 폴더에 클론합니다.
2. `yarn install`로 의존성을 설치합니다.
3. `yarn dev`로 개발 모드를 시작합니다.
4. Obsidian에서 플러그인을 활성화합니다.

## 라이선스

GPL-3.0

## 후원

이 플러그인은 무료로 제공되지만, 감사의 표시나 지속적인 개발 지원을 원하신다면 아래 방법으로 후원해 주세요:

[![GitHub Sponsors](https://img.shields.io/github/sponsors/tgrosinger?style=social)](https://github.com/sponsors/tgrosinger)
[![Paypal](https://img.shields.io/badge/paypal-tgrosinger-yellow?style=social&logo=paypal)](https://paypal.me/tgrosinger)
[<img src="https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png" alt="BuyMeACoffee" width="100">](https://www.buymeacoffee.com/tgrosinger)
