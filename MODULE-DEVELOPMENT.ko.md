# Obsidian 플러그인 모듈 개발 가이드

이 문서는 Obsidian 플러그인을 모듈화하고 재사용 가능한 컴포넌트로 구조화하는 방법을 안내합니다.

## 목차

- [모듈화란?](#모듈화란)
- [프로젝트 구조](#프로젝트-구조)
- [모듈 패턴](#모듈-패턴)
- [재사용 가능한 컴포넌트](#재사용-가능한-컴포넌트)
- [상태 관리](#상태-관리)
- [이벤트 시스템](#이벤트-시스템)
- [테스트](#테스트)

## 모듈화란?

### 왜 모듈화가 필요한가?

**단일 파일 구조의 문제점:**
```typescript
// main.ts (1000+ 줄)
export default class MyPlugin extends Plugin {
  // 모든 기능이 한 파일에...
  async onload() { /* ... */ }
  onunload() { /* ... */ }
  // 설정, 뷰, 모달, 유틸리티 등 모두 포함
}
```

**모듈화의 장점:**
- ✅ 코드 가독성 향상
- ✅ 유지보수 용이
- ✅ 재사용성 증가
- ✅ 테스트 가능성
- ✅ 협업 효율성

## 프로젝트 구조

### 권장 디렉토리 구조

```
my-plugin/
├── src/
│   ├── main.ts              # 플러그인 진입점
│   ├── settings/            # 설정 관련
│   │   ├── settings.ts      # 설정 인터페이스
│   │   ├── settingsTab.ts   # 설정 UI
│   │   └── defaults.ts      # 기본값
│   ├── views/               # 커스텀 뷰
│   │   ├── BaseView.ts      # 기본 뷰 클래스
│   │   ├── ListView.ts      # 리스트 뷰
│   │   └── DetailView.ts    # 상세 뷰
│   ├── modals/              # 모달
│   │   ├── BaseModal.ts     # 기본 모달
│   │   ├── ConfirmModal.ts  # 확인 모달
│   │   └── InputModal.ts    # 입력 모달
│   ├── services/            # 서비스 (비즈니스 로직)
│   │   ├── FileService.ts   # 파일 작업
│   │   ├── DataService.ts   # 데이터 관리
│   │   └── SyncService.ts   # 동기화
│   ├── utils/               # 유틸리티
│   │   ├── string.ts        # 문자열 처리
│   │   ├── date.ts          # 날짜 처리
│   │   └── validation.ts    # 유효성 검사
│   ├── types/               # 타입 정의
│   │   ├── index.ts         # 메인 타입
│   │   └── events.ts        # 이벤트 타입
│   └── constants/           # 상수
│       └── index.ts
├── styles/                  # 스타일
│   └── main.css
├── tests/                   # 테스트
│   ├── unit/
│   └── integration/
├── main.ts                  # esbuild 진입점 (import from src)
├── manifest.json
├── package.json
└── tsconfig.json
```

### 진입점 설정

**main.ts (루트)**
```typescript
// esbuild 진입점
export { default } from './src/main';
```

**src/main.ts**
```typescript
import { Plugin } from 'obsidian';
import { SettingsTab } from './settings/settingsTab';
import { FileService } from './services/FileService';
import { DataService } from './services/DataService';
import { DEFAULT_SETTINGS } from './settings/defaults';
import type { MyPluginSettings } from './types';

export default class MyPlugin extends Plugin {
  settings: MyPluginSettings;
  fileService: FileService;
  dataService: DataService;

  async onload() {
    await this.loadSettings();

    // 서비스 초기화
    this.fileService = new FileService(this.app);
    this.dataService = new DataService(this);

    // 설정 탭 추가
    this.addSettingTab(new SettingsTab(this.app, this));

    // 기능 로드
    this.registerCommands();
    this.registerViews();
  }

  async loadSettings() {
    this.settings = Object.assign({}, DEFAULT_SETTINGS, await this.loadData());
  }

  async saveSettings() {
    await this.saveData(this.settings);
  }

  private registerCommands() {
    // 명령어 등록
  }

  private registerViews() {
    // 뷰 등록
  }
}
```

## 모듈 패턴

### 1. 설정 모듈

**src/types/index.ts**
```typescript
export interface MyPluginSettings {
  apiKey: string;
  enabled: boolean;
  maxItems: number;
  theme: 'light' | 'dark';
}
```

**src/settings/defaults.ts**
```typescript
import type { MyPluginSettings } from '../types';

export const DEFAULT_SETTINGS: MyPluginSettings = {
  apiKey: '',
  enabled: true,
  maxItems: 50,
  theme: 'light'
};
```

**src/settings/settingsTab.ts**
```typescript
import { App, PluginSettingTab, Setting } from 'obsidian';
import type MyPlugin from '../main';

export class SettingsTab extends PluginSettingTab {
  plugin: MyPlugin;

  constructor(app: App, plugin: MyPlugin) {
    super(app, plugin);
    this.plugin = plugin;
  }

  display(): void {
    const { containerEl } = this;
    containerEl.empty();

    this.addApiKeySetting(containerEl);
    this.addEnabledSetting(containerEl);
    this.addMaxItemsSetting(containerEl);
  }

  private addApiKeySetting(containerEl: HTMLElement): void {
    new Setting(containerEl)
      .setName('API 키')
      .setDesc('서비스 API 키를 입력하세요')
      .addText(text => text
        .setPlaceholder('API 키')
        .setValue(this.plugin.settings.apiKey)
        .onChange(async (value) => {
          this.plugin.settings.apiKey = value;
          await this.plugin.saveSettings();
        }));
  }

  private addEnabledSetting(containerEl: HTMLElement): void {
    new Setting(containerEl)
      .setName('활성화')
      .setDesc('플러그인 활성화/비활성화')
      .addToggle(toggle => toggle
        .setValue(this.plugin.settings.enabled)
        .onChange(async (value) => {
          this.plugin.settings.enabled = value;
          await this.plugin.saveSettings();
        }));
  }

  private addMaxItemsSetting(containerEl: HTMLElement): void {
    new Setting(containerEl)
      .setName('최대 항목 수')
      .setDesc('표시할 최대 항목 수')
      .addText(text => text
        .setPlaceholder('50')
        .setValue(String(this.plugin.settings.maxItems))
        .onChange(async (value) => {
          const parsed = parseInt(value, 10);
          if (!isNaN(parsed) && parsed > 0) {
            this.plugin.settings.maxItems = parsed;
            await this.plugin.saveSettings();
          }
        }));
  }
}
```

### 2. 서비스 모듈

**src/services/FileService.ts**
```typescript
import { App, TFile, TFolder, Notice } from 'obsidian';

export class FileService {
  private app: App;

  constructor(app: App) {
    this.app = app;
  }

  /**
   * 파일 읽기
   */
  async readFile(path: string): Promise<string | null> {
    try {
      const file = this.app.vault.getAbstractFileByPath(path);
      if (file instanceof TFile) {
        return await this.app.vault.read(file);
      }
      return null;
    } catch (error) {
      console.error('파일 읽기 실패:', error);
      new Notice('파일을 읽을 수 없습니다');
      return null;
    }
  }

  /**
   * 파일 쓰기
   */
  async writeFile(path: string, content: string): Promise<boolean> {
    try {
      const file = this.app.vault.getAbstractFileByPath(path);
      if (file instanceof TFile) {
        await this.app.vault.modify(file, content);
        return true;
      } else {
        await this.app.vault.create(path, content);
        return true;
      }
    } catch (error) {
      console.error('파일 쓰기 실패:', error);
      new Notice('파일을 저장할 수 없습니다');
      return false;
    }
  }

  /**
   * 파일 삭제
   */
  async deleteFile(path: string): Promise<boolean> {
    try {
      const file = this.app.vault.getAbstractFileByPath(path);
      if (file instanceof TFile) {
        await this.app.vault.delete(file);
        return true;
      }
      return false;
    } catch (error) {
      console.error('파일 삭제 실패:', error);
      new Notice('파일을 삭제할 수 없습니다');
      return false;
    }
  }

  /**
   * 폴더의 모든 마크다운 파일 가져오기
   */
  getMarkdownFilesInFolder(folderPath: string): TFile[] {
    const folder = this.app.vault.getAbstractFileByPath(folderPath);
    if (!(folder instanceof TFolder)) {
      return [];
    }

    const files: TFile[] = [];
    const traverse = (folder: TFolder) => {
      for (const child of folder.children) {
        if (child instanceof TFile && child.extension === 'md') {
          files.push(child);
        } else if (child instanceof TFolder) {
          traverse(child);
        }
      }
    };

    traverse(folder);
    return files;
  }

  /**
   * 모든 마크다운 파일 가져오기
   */
  getAllMarkdownFiles(): TFile[] {
    return this.app.vault.getMarkdownFiles();
  }
}
```

**src/services/DataService.ts**
```typescript
import type MyPlugin from '../main';
import type { MyPluginSettings } from '../types';

export interface DataItem {
  id: string;
  title: string;
  content: string;
  created: number;
  modified: number;
}

export class DataService {
  private plugin: MyPlugin;
  private cache: Map<string, DataItem>;

  constructor(plugin: MyPlugin) {
    this.plugin = plugin;
    this.cache = new Map();
  }

  /**
   * 데이터 추가
   */
  async addItem(item: Omit<DataItem, 'id' | 'created' | 'modified'>): Promise<DataItem> {
    const newItem: DataItem = {
      ...item,
      id: this.generateId(),
      created: Date.now(),
      modified: Date.now()
    };

    this.cache.set(newItem.id, newItem);
    await this.persist();
    return newItem;
  }

  /**
   * 데이터 업데이트
   */
  async updateItem(id: string, updates: Partial<DataItem>): Promise<boolean> {
    const item = this.cache.get(id);
    if (!item) return false;

    const updated = {
      ...item,
      ...updates,
      modified: Date.now()
    };

    this.cache.set(id, updated);
    await this.persist();
    return true;
  }

  /**
   * 데이터 삭제
   */
  async deleteItem(id: string): Promise<boolean> {
    const result = this.cache.delete(id);
    if (result) {
      await this.persist();
    }
    return result;
  }

  /**
   * 데이터 조회
   */
  getItem(id: string): DataItem | undefined {
    return this.cache.get(id);
  }

  /**
   * 모든 데이터 조회
   */
  getAllItems(): DataItem[] {
    return Array.from(this.cache.values());
  }

  /**
   * 데이터 검색
   */
  searchItems(query: string): DataItem[] {
    const lowerQuery = query.toLowerCase();
    return this.getAllItems().filter(item =>
      item.title.toLowerCase().includes(lowerQuery) ||
      item.content.toLowerCase().includes(lowerQuery)
    );
  }

  /**
   * 데이터 정렬
   */
  sortItems(sortBy: 'created' | 'modified' | 'title', order: 'asc' | 'desc' = 'desc'): DataItem[] {
    const items = this.getAllItems();
    return items.sort((a, b) => {
      let comparison = 0;
      if (sortBy === 'title') {
        comparison = a.title.localeCompare(b.title);
      } else {
        comparison = a[sortBy] - b[sortBy];
      }
      return order === 'asc' ? comparison : -comparison;
    });
  }

  /**
   * 데이터 저장
   */
  private async persist(): Promise<void> {
    const data = Array.from(this.cache.values());
    await this.plugin.saveData({ items: data });
  }

  /**
   * 데이터 로드
   */
  async load(): Promise<void> {
    const data = await this.plugin.loadData();
    if (data?.items) {
      this.cache.clear();
      for (const item of data.items) {
        this.cache.set(item.id, item);
      }
    }
  }

  /**
   * ID 생성
   */
  private generateId(): string {
    return `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
  }
}
```

### 3. 유틸리티 모듈

**src/utils/string.ts**
```typescript
/**
 * 문자열을 슬러그로 변환
 */
export function slugify(text: string): string {
  return text
    .toLowerCase()
    .replace(/[^\w\s-]/g, '')
    .replace(/[\s_-]+/g, '-')
    .replace(/^-+|-+$/g, '');
}

/**
 * 문자열 자르기 (말줄임표 포함)
 */
export function truncate(text: string, maxLength: number): string {
  if (text.length <= maxLength) return text;
  return text.substring(0, maxLength - 3) + '...';
}

/**
 * 문자열에서 마크다운 제거
 */
export function stripMarkdown(text: string): string {
  return text
    .replace(/#{1,6}\s/g, '')           // 헤딩
    .replace(/\*\*(.+?)\*\*/g, '$1')    // 볼드
    .replace(/\*(.+?)\*/g, '$1')        // 이탤릭
    .replace(/\[(.+?)\]\(.+?\)/g, '$1') // 링크
    .replace(/`(.+?)`/g, '$1')          // 코드
    .trim();
}

/**
 * 카멜케이스를 공백으로 구분된 문자열로 변환
 */
export function camelToTitle(text: string): string {
  return text
    .replace(/([A-Z])/g, ' $1')
    .replace(/^./, str => str.toUpperCase())
    .trim();
}
```

**src/utils/date.ts**
```typescript
/**
 * 상대 시간 표시 (예: "2시간 전")
 */
export function getRelativeTime(timestamp: number): string {
  const now = Date.now();
  const diff = now - timestamp;

  const seconds = Math.floor(diff / 1000);
  const minutes = Math.floor(seconds / 60);
  const hours = Math.floor(minutes / 60);
  const days = Math.floor(hours / 24);
  const months = Math.floor(days / 30);
  const years = Math.floor(months / 12);

  if (years > 0) return `${years}년 전`;
  if (months > 0) return `${months}개월 전`;
  if (days > 0) return `${days}일 전`;
  if (hours > 0) return `${hours}시간 전`;
  if (minutes > 0) return `${minutes}분 전`;
  return '방금 전';
}

/**
 * 날짜 포맷팅
 */
export function formatDate(timestamp: number, format: string = 'YYYY-MM-DD'): string {
  const date = new Date(timestamp);

  const year = date.getFullYear();
  const month = String(date.getMonth() + 1).padStart(2, '0');
  const day = String(date.getDate()).padStart(2, '0');
  const hours = String(date.getHours()).padStart(2, '0');
  const minutes = String(date.getMinutes()).padStart(2, '0');
  const seconds = String(date.getSeconds()).padStart(2, '0');

  return format
    .replace('YYYY', String(year))
    .replace('MM', month)
    .replace('DD', day)
    .replace('HH', hours)
    .replace('mm', minutes)
    .replace('ss', seconds);
}

/**
 * 오늘인지 확인
 */
export function isToday(timestamp: number): boolean {
  const date = new Date(timestamp);
  const today = new Date();
  return date.toDateString() === today.toDateString();
}
```

**src/utils/validation.ts**
```typescript
/**
 * 이메일 유효성 검사
 */
export function isValidEmail(email: string): boolean {
  const re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return re.test(email);
}

/**
 * URL 유효성 검사
 */
export function isValidUrl(url: string): boolean {
  try {
    new URL(url);
    return true;
  } catch {
    return false;
  }
}

/**
 * 파일 경로 유효성 검사
 */
export function isValidPath(path: string): boolean {
  // Obsidian 파일 경로 규칙
  const invalidChars = /[<>:"|?*\x00-\x1F]/;
  return !invalidChars.test(path);
}

/**
 * 필수 필드 검사
 */
export function validateRequired<T>(
  data: T,
  requiredFields: (keyof T)[]
): { valid: boolean; missing: string[] } {
  const missing: string[] = [];

  for (const field of requiredFields) {
    if (data[field] === undefined || data[field] === null || data[field] === '') {
      missing.push(String(field));
    }
  }

  return {
    valid: missing.length === 0,
    missing
  };
}
```

## 재사용 가능한 컴포넌트

### 1. 기본 모달

**src/modals/BaseModal.ts**
```typescript
import { App, Modal } from 'obsidian';

export abstract class BaseModal extends Modal {
  protected onSubmit?: () => void;
  protected onCancel?: () => void;

  constructor(app: App) {
    super(app);
  }

  abstract renderContent(containerEl: HTMLElement): void;

  onOpen() {
    const { contentEl } = this;
    contentEl.empty();

    this.renderContent(contentEl);
    this.renderButtons(contentEl);
  }

  protected renderButtons(containerEl: HTMLElement): void {
    const buttonContainer = containerEl.createDiv({ cls: 'modal-button-container' });

    const cancelButton = buttonContainer.createEl('button', {
      text: '취소',
      cls: 'mod-cancel'
    });
    cancelButton.addEventListener('click', () => {
      this.onCancel?.();
      this.close();
    });

    const submitButton = buttonContainer.createEl('button', {
      text: '확인',
      cls: 'mod-cta'
    });
    submitButton.addEventListener('click', () => {
      this.onSubmit?.();
      this.close();
    });
  }

  onClose() {
    const { contentEl } = this;
    contentEl.empty();
  }
}
```

**src/modals/ConfirmModal.ts**
```typescript
import { App } from 'obsidian';
import { BaseModal } from './BaseModal';

export class ConfirmModal extends BaseModal {
  private message: string;
  private onConfirm: () => void;

  constructor(app: App, message: string, onConfirm: () => void) {
    super(app);
    this.message = message;
    this.onConfirm = onConfirm;
    this.onSubmit = onConfirm;
  }

  renderContent(containerEl: HTMLElement): void {
    containerEl.createEl('h2', { text: '확인' });
    containerEl.createEl('p', { text: this.message });
  }
}

// 사용 예
// new ConfirmModal(
//   this.app,
//   '정말 삭제하시겠습니까?',
//   () => { console.log('삭제 확인됨'); }
// ).open();
```

### 2. 기본 뷰

**src/views/BaseView.ts**
```typescript
import { ItemView, WorkspaceLeaf } from 'obsidian';

export abstract class BaseView extends ItemView {
  abstract getViewType(): string;
  abstract getDisplayText(): string;
  abstract getIcon(): string;

  constructor(leaf: WorkspaceLeaf) {
    super(leaf);
  }

  protected getContainer(): HTMLElement {
    return this.containerEl.children[1] as HTMLElement;
  }

  protected renderHeader(text: string): HTMLElement {
    const container = this.getContainer();
    return container.createEl('div', {
      cls: 'view-header',
      text: text
    });
  }

  protected renderContent(cls?: string): HTMLElement {
    const container = this.getContainer();
    return container.createEl('div', {
      cls: cls || 'view-content'
    });
  }

  protected clearContent(): void {
    const container = this.getContainer();
    container.empty();
  }
}
```

## 상태 관리

**src/services/StateService.ts**
```typescript
type StateListener<T> = (newState: T, oldState: T) => void;

export class StateService<T> {
  private state: T;
  private listeners: Set<StateListener<T>>;

  constructor(initialState: T) {
    this.state = initialState;
    this.listeners = new Set();
  }

  getState(): T {
    return { ...this.state };
  }

  setState(updates: Partial<T>): void {
    const oldState = { ...this.state };
    this.state = { ...this.state, ...updates };
    this.notifyListeners(this.state, oldState);
  }

  subscribe(listener: StateListener<T>): () => void {
    this.listeners.add(listener);
    return () => this.listeners.delete(listener);
  }

  private notifyListeners(newState: T, oldState: T): void {
    this.listeners.forEach(listener => listener(newState, oldState));
  }
}

// 사용 예
// interface AppState {
//   isLoading: boolean;
//   items: Item[];
//   selectedId: string | null;
// }
//
// const stateService = new StateService<AppState>({
//   isLoading: false,
//   items: [],
//   selectedId: null
// });
//
// const unsubscribe = stateService.subscribe((newState, oldState) => {
//   console.log('상태 변경:', newState);
// });
```

## 이벤트 시스템

**src/services/EventBus.ts**
```typescript
type EventHandler<T = any> = (data: T) => void;

export class EventBus {
  private events: Map<string, Set<EventHandler>>;

  constructor() {
    this.events = new Map();
  }

  on<T = any>(event: string, handler: EventHandler<T>): () => void {
    if (!this.events.has(event)) {
      this.events.set(event, new Set());
    }

    this.events.get(event)!.add(handler);

    // 구독 해제 함수 반환
    return () => this.off(event, handler);
  }

  off<T = any>(event: string, handler: EventHandler<T>): void {
    const handlers = this.events.get(event);
    if (handlers) {
      handlers.delete(handler);
    }
  }

  emit<T = any>(event: string, data?: T): void {
    const handlers = this.events.get(event);
    if (handlers) {
      handlers.forEach(handler => handler(data));
    }
  }

  clear(): void {
    this.events.clear();
  }
}

// 사용 예
// const eventBus = new EventBus();
//
// const unsubscribe = eventBus.on('file-created', (file) => {
//   console.log('파일 생성됨:', file);
// });
//
// eventBus.emit('file-created', { path: 'test.md' });
// unsubscribe();
```

## 테스트

### Jest 설정

**package.json**
```json
{
  "devDependencies": {
    "@types/jest": "^29.0.0",
    "jest": "^29.0.0",
    "ts-jest": "^29.0.0"
  },
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch"
  }
}
```

**jest.config.js**
```javascript
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  roots: ['<rootDir>/tests'],
  testMatch: ['**/*.test.ts'],
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1'
  }
};
```

### 유닛 테스트 예제

**tests/unit/utils/string.test.ts**
```typescript
import { slugify, truncate, stripMarkdown } from '@/utils/string';

describe('String Utils', () => {
  describe('slugify', () => {
    it('소문자로 변환', () => {
      expect(slugify('Hello World')).toBe('hello-world');
    });

    it('특수문자 제거', () => {
      expect(slugify('Hello @#$ World!')).toBe('hello-world');
    });

    it('공백을 하이픈으로', () => {
      expect(slugify('hello   world')).toBe('hello-world');
    });
  });

  describe('truncate', () => {
    it('짧은 문자열은 그대로', () => {
      expect(truncate('Hello', 10)).toBe('Hello');
    });

    it('긴 문자열은 자르기', () => {
      expect(truncate('Hello World', 8)).toBe('Hello...');
    });
  });

  describe('stripMarkdown', () => {
    it('헤딩 제거', () => {
      expect(stripMarkdown('# Title')).toBe('Title');
    });

    it('볼드 제거', () => {
      expect(stripMarkdown('**bold**')).toBe('bold');
    });

    it('링크 제거', () => {
      expect(stripMarkdown('[text](url)')).toBe('text');
    });
  });
});
```

## 모범 사례

1. **단일 책임 원칙**: 각 모듈은 하나의 책임만
2. **의존성 주입**: 생성자를 통해 의존성 전달
3. **타입 안정성**: 모든 함수와 변수에 타입 지정
4. **에러 처리**: 모든 비동기 작업에 try-catch
5. **문서화**: JSDoc으로 공개 API 문서화
6. **테스트**: 핵심 기능에 대한 유닛 테스트 작성

## 참고 리소스

- [TypeScript 모듈](https://www.typescriptlang.org/docs/handbook/modules.html)
- [SOLID 원칙](https://en.wikipedia.org/wiki/SOLID)
- [디자인 패턴](https://refactoring.guru/design-patterns)
