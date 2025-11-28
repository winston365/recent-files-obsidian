# Obsidian 플러그인 개발 가이드

이 문서는 Obsidian 플러그인을 처음부터 만드는 방법을 안내합니다.

## 목차

- [시작하기](#시작하기)
- [프로젝트 설정](#프로젝트-설정)
- [기본 구조](#기본-구조)
- [핵심 API](#핵심-api)
- [빌드 시스템](#빌드-시스템)
- [배포](#배포)

## 시작하기

### 사전 요구사항

```bash
# Node.js 설치 확인
node --version  # v16 이상 권장

# npm 설치 확인
npm --version   # v8 이상 권장

# TypeScript 지식 (필수)
# Obsidian API 기본 이해
```

### 플러그인 아이디어 검증

플러그인을 만들기 전에:
1. [커뮤니티 플러그인](https://obsidian.md/plugins) 검색으로 중복 확인
2. Obsidian 포럼에서 유사 기능 요청 확인
3. 기술적 구현 가능성 검토

## 프로젝트 설정

### 1. 샘플 플러그인 클론

```bash
# Obsidian 공식 샘플 플러그인 사용
git clone https://github.com/obsidianmd/obsidian-sample-plugin.git my-plugin
cd my-plugin

# 의존성 설치
npm install
```

### 2. 기본 파일 수정

**manifest.json**
```json
{
  "id": "my-awesome-plugin",
  "name": "My Awesome Plugin",
  "version": "1.0.0",
  "minAppVersion": "0.16.0",
  "description": "플러그인 설명",
  "author": "당신의 이름",
  "authorUrl": "https://yourwebsite.com",
  "isDesktopOnly": false
}
```

**package.json**
```json
{
  "name": "my-awesome-plugin",
  "version": "1.0.0",
  "description": "플러그인 설명",
  "main": "main.js",
  "scripts": {
    "dev": "node esbuild.config.mjs",
    "build": "tsc -noEmit -skipLibCheck && node esbuild.config.mjs production"
  }
}
```

### 3. 개발 환경 구성

```bash
# vault의 플러그인 폴더로 심볼릭 링크 생성
ln -s /path/to/my-plugin /path/to/vault/.obsidian/plugins/my-awesome-plugin

# 또는 Windows
mklink /D "C:\vault\.obsidian\plugins\my-awesome-plugin" "C:\dev\my-plugin"

# 개발 모드 시작
npm run dev
```

## 기본 구조

### 플러그인 클래스

**main.ts**
```typescript
import { Plugin, PluginManifest } from 'obsidian';

export default class MyPlugin extends Plugin {
  // 플러그인이 로드될 때 한 번 실행
  async onload(): Promise<void> {
    console.log('플러그인 로드됨');

    // 설정 로드
    await this.loadSettings();

    // 리본 아이콘 추가
    this.addRibbonIcon('dice', '플러그인', () => {
      console.log('아이콘 클릭됨');
    });

    // 명령어 추가
    this.addCommand({
      id: 'open-sample-modal',
      name: '샘플 모달 열기',
      callback: () => {
        console.log('명령어 실행됨');
      }
    });

    // 설정 탭 추가
    this.addSettingTab(new SampleSettingTab(this.app, this));
  }

  // 플러그인이 언로드될 때 실행 (정리 작업)
  onunload(): void {
    console.log('플러그인 언로드됨');
  }

  // 데이터 저장
  async loadSettings(): Promise<void> {
    this.settings = Object.assign({}, DEFAULT_SETTINGS, await this.loadData());
  }

  async saveSettings(): Promise<void> {
    await this.saveData(this.settings);
  }
}
```

### 파일 구조

```
my-plugin/
├── main.ts              # 플러그인 진입점
├── manifest.json        # 플러그인 메타데이터
├── package.json         # npm 설정
├── tsconfig.json        # TypeScript 설정
├── esbuild.config.mjs   # 빌드 설정
├── styles.css           # 스타일 (선택)
└── src/                 # 소스 코드 (선택)
    ├── settings.ts      # 설정 관련
    ├── modal.ts         # 모달 컴포넌트
    └── view.ts          # 커스텀 뷰
```

## 핵심 API

### 1. 명령어 (Commands)

```typescript
this.addCommand({
  id: 'my-command',
  name: '명령어 이름',
  // 단축키 (선택)
  hotkeys: [
    {
      modifiers: ['Mod', 'Shift'],
      key: 'k'
    }
  ],
  // 에디터가 활성화되었을 때만 사용 가능
  editorCallback: (editor: Editor, view: MarkdownView) => {
    const cursor = editor.getCursor();
    editor.replaceRange('텍스트 삽입', cursor);
  },
  // 에디터와 무관하게 사용 가능
  callback: () => {
    new Notice('명령어 실행됨');
  },
  // 명령어 활성화 조건
  checkCallback: (checking: boolean) => {
    const file = this.app.workspace.getActiveFile();
    if (file) {
      if (!checking) {
        // 실제 실행
        new Notice(`파일: ${file.name}`);
      }
      return true;
    }
    return false;
  }
});
```

### 2. 설정 (Settings)

```typescript
interface MyPluginSettings {
  mySetting: string;
  enabled: boolean;
  count: number;
}

const DEFAULT_SETTINGS: MyPluginSettings = {
  mySetting: 'default',
  enabled: true,
  count: 0
}

class MyPlugin extends Plugin {
  settings: MyPluginSettings;

  async onload() {
    await this.loadSettings();
    this.addSettingTab(new MySettingTab(this.app, this));
  }

  async loadSettings() {
    this.settings = Object.assign({}, DEFAULT_SETTINGS, await this.loadData());
  }

  async saveSettings() {
    await this.saveData(this.settings);
  }
}

class MySettingTab extends PluginSettingTab {
  plugin: MyPlugin;

  constructor(app: App, plugin: MyPlugin) {
    super(app, plugin);
    this.plugin = plugin;
  }

  display(): void {
    const { containerEl } = this;
    containerEl.empty();

    new Setting(containerEl)
      .setName('설정 이름')
      .setDesc('설정 설명')
      .addText(text => text
        .setPlaceholder('입력...')
        .setValue(this.plugin.settings.mySetting)
        .onChange(async (value) => {
          this.plugin.settings.mySetting = value;
          await this.plugin.saveSettings();
        }));

    new Setting(containerEl)
      .setName('토글 설정')
      .addToggle(toggle => toggle
        .setValue(this.plugin.settings.enabled)
        .onChange(async (value) => {
          this.plugin.settings.enabled = value;
          await this.plugin.saveSettings();
        }));
  }
}
```

### 3. 모달 (Modals)

```typescript
import { App, Modal } from 'obsidian';

class SampleModal extends Modal {
  constructor(app: App) {
    super(app);
  }

  onOpen() {
    const { contentEl } = this;
    contentEl.createEl('h2', { text: '모달 제목' });

    const input = contentEl.createEl('input', {
      type: 'text',
      placeholder: '입력...'
    });

    const button = contentEl.createEl('button', {
      text: '확인'
    });

    button.addEventListener('click', () => {
      new Notice(`입력: ${input.value}`);
      this.close();
    });
  }

  onClose() {
    const { contentEl } = this;
    contentEl.empty();
  }
}

// 사용
new SampleModal(this.app).open();
```

### 4. 커스텀 뷰 (Views)

```typescript
import { ItemView, WorkspaceLeaf } from 'obsidian';

const VIEW_TYPE_EXAMPLE = 'example-view';

class ExampleView extends ItemView {
  constructor(leaf: WorkspaceLeaf) {
    super(leaf);
  }

  getViewType() {
    return VIEW_TYPE_EXAMPLE;
  }

  getDisplayText() {
    return '예제 뷰';
  }

  getIcon() {
    return 'dice';
  }

  async onOpen() {
    const container = this.containerEl.children[1];
    container.empty();
    container.createEl('h4', { text: '커스텀 뷰' });
    container.createEl('p', { text: '내용...' });
  }

  async onClose() {
    // 정리 작업
  }
}

// 플러그인에서 등록
this.registerView(
  VIEW_TYPE_EXAMPLE,
  (leaf) => new ExampleView(leaf)
);

// 뷰 활성화
this.addCommand({
  id: 'open-example-view',
  name: '예제 뷰 열기',
  callback: () => {
    this.activateView();
  }
});

async activateView() {
  this.app.workspace.detachLeavesOfType(VIEW_TYPE_EXAMPLE);

  await this.app.workspace.getRightLeaf(false).setViewState({
    type: VIEW_TYPE_EXAMPLE,
    active: true,
  });

  this.app.workspace.revealLeaf(
    this.app.workspace.getLeavesOfType(VIEW_TYPE_EXAMPLE)[0]
  );
}
```

### 5. 파일 작업

```typescript
// 파일 읽기
const file = this.app.vault.getAbstractFileByPath('path/to/file.md');
if (file instanceof TFile) {
  const content = await this.app.vault.read(file);
  console.log(content);
}

// 파일 쓰기
await this.app.vault.create('new-file.md', '# 제목\n내용');

// 파일 수정
await this.app.vault.modify(file, '새 내용');

// 파일 삭제
await this.app.vault.delete(file);

// 파일 이벤트 리스닝
this.registerEvent(
  this.app.vault.on('create', (file) => {
    console.log('파일 생성됨:', file.path);
  })
);

this.registerEvent(
  this.app.vault.on('modify', (file) => {
    console.log('파일 수정됨:', file.path);
  })
);

this.registerEvent(
  this.app.vault.on('delete', (file) => {
    console.log('파일 삭제됨:', file.path);
  })
);
```

### 6. 에디터 작업

```typescript
// 현재 에디터 가져오기
const editor = this.app.workspace.activeEditor?.editor;
if (editor) {
  // 커서 위치
  const cursor = editor.getCursor();

  // 선택된 텍스트
  const selection = editor.getSelection();

  // 텍스트 삽입
  editor.replaceSelection('새 텍스트');

  // 특정 위치에 삽입
  editor.replaceRange('삽입', { line: 0, ch: 0 });

  // 전체 내용
  const content = editor.getValue();

  // 전체 내용 교체
  editor.setValue('새 내용');
}
```

### 7. 워크스페이스 이벤트

```typescript
// 파일 열림
this.registerEvent(
  this.app.workspace.on('file-open', (file) => {
    if (file) {
      console.log('파일 열림:', file.path);
    }
  })
);

// 레이아웃 변경
this.registerEvent(
  this.app.workspace.on('layout-change', () => {
    console.log('레이아웃 변경됨');
  })
);

// 활성 리프 변경
this.registerEvent(
  this.app.workspace.on('active-leaf-change', (leaf) => {
    console.log('활성 리프 변경됨');
  })
);
```

## 빌드 시스템

### esbuild 설정

**esbuild.config.mjs**
```javascript
import esbuild from 'esbuild';
import process from 'process';
import builtins from 'builtin-modules';

const prod = process.argv[2] === 'production';

const context = await esbuild.context({
  entryPoints: ['main.ts'],
  bundle: true,
  external: [
    'obsidian',
    'electron',
    '@codemirror/*',
    ...builtins
  ],
  format: 'cjs',
  target: 'es2018',
  logLevel: 'info',
  sourcemap: prod ? false : 'inline',
  treeShaking: true,
  outfile: 'main.js',
  minify: prod,
});

if (prod) {
  await context.rebuild();
  process.exit(0);
} else {
  await context.watch();
}
```

### TypeScript 설정

**tsconfig.json**
```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "inlineSourceMap": true,
    "inlineSources": true,
    "module": "ESNext",
    "target": "ES6",
    "allowJs": true,
    "noImplicitAny": true,
    "moduleResolution": "node",
    "importHelpers": true,
    "isolatedModules": true,
    "strictNullChecks": true,
    "lib": ["DOM", "ES5", "ES6", "ES7"]
  },
  "include": ["**/*.ts"]
}
```

## 배포

### 1. 버전 업데이트

```bash
# manifest.json의 version 업데이트
# package.json의 version 업데이트
# versions.json에 새 버전 추가
```

**versions.json**
```json
{
  "1.0.0": "0.16.0",
  "1.1.0": "0.16.3"
}
```

### 2. 빌드

```bash
npm run build
```

### 3. 릴리스 파일 준비

다음 파일들을 포함:
- `main.js` (빌드된 플러그인)
- `manifest.json`
- `styles.css` (있는 경우)

### 4. GitHub 릴리스 생성

```bash
git tag -a 1.0.0 -m "Release 1.0.0"
git push origin 1.0.0
```

GitHub에서:
1. Releases → New release
2. Tag 선택 (1.0.0)
3. 릴리스 노트 작성
4. `main.js`, `manifest.json`, `styles.css` 첨부
5. Publish release

### 5. 커뮤니티 플러그인 등록

1. [obsidian-releases](https://github.com/obsidianmd/obsidian-releases) 저장소 Fork
2. `community-plugins.json`에 플러그인 추가:
   ```json
   {
     "id": "my-awesome-plugin",
     "name": "My Awesome Plugin",
     "author": "Your Name",
     "description": "플러그인 설명",
     "repo": "username/my-awesome-plugin"
   }
   ```
3. Pull Request 생성
4. Obsidian 팀 승인 대기

## 디버깅

### 개발자 도구

```
Ctrl/Cmd + Shift + I
```

### 로깅

```typescript
console.log('일반 로그');
console.error('에러 로그');
console.warn('경고');
```

### 성능 측정

```typescript
console.time('작업');
// 작업 수행
console.timeEnd('작업');
```

## 모범 사례

### 1. 에러 처리

```typescript
try {
  const file = this.app.vault.getAbstractFileByPath(path);
  if (!file) {
    throw new Error('파일을 찾을 수 없습니다');
  }
  // 작업 수행
} catch (error) {
  console.error('에러 발생:', error);
  new Notice('작업 실패');
}
```

### 2. 메모리 관리

```typescript
onunload() {
  // 이벤트 리스너 제거
  // 타이머 정리
  // 뷰 닫기
}
```

### 3. 비동기 처리

```typescript
async onload() {
  // await 사용
  await this.loadSettings();
  await this.initializePlugin();
}
```

### 4. 타입 안정성

```typescript
// any 대신 구체적인 타입 사용
interface MyData {
  field: string;
}

const data: MyData = await this.loadData();
```

## 유용한 리소스

- [Obsidian API 문서](https://docs.obsidian.md/Home)
- [샘플 플러그인](https://github.com/obsidianmd/obsidian-sample-plugin)
- [커뮤니티 플러그인 목록](https://github.com/obsidianmd/obsidian-releases)
- [개발자 포럼](https://forum.obsidian.md/c/developers-co-op/14)
