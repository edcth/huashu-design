# Tweaks：設計變體實時調參

Tweaks是這個skill裡很核心的能力——讓用戶不改代碼就能實時切換variations/調整參數。

**跨 agent 環境適配**：某些 design-agent 原生環境（如 Claude.ai Artifacts）依賴 host 的 postMessage 把 tweak 值回寫源碼做持久化。本 skill 採用**純前端 localStorage 方案**——效果一致（刷新保留狀態），但持久化發生在瀏覽器 localStorage 而不是源碼文件。這個方案在任何 agent 環境（Claude Code / Codex / Cursor / Trae / etc.）都能工作。

## 何時加 Tweaks

- 用戶明確要求"能調參"/"多個版本切換"
- 設計有多個variations需要對比時
- 用戶沒明說，但你主觀判斷**加幾個有啟發性的tweaks能幫用戶看到可能性**

默認推薦：**每個設計都加2-3個tweaks**（顏色主題/字號/layout變體）即使用戶沒要求——讓用戶看到可能性空間是設計服務的一部分。

## 實現方式（純前端版）

### 基本結構

```jsx
const TWEAK_DEFAULTS = {
  "primaryColor": "#D97757",
  "fontSize": 16,
  "density": "comfortable",
  "dark": false
};

function useTweaks() {
  const [tweaks, setTweaks] = React.useState(() => {
    try {
      const stored = localStorage.getItem('design-tweaks');
      return stored ? { ...TWEAK_DEFAULTS, ...JSON.parse(stored) } : TWEAK_DEFAULTS;
    } catch {
      return TWEAK_DEFAULTS;
    }
  });

  const update = (patch) => {
    const next = { ...tweaks, ...patch };
    setTweaks(next);
    try {
      localStorage.setItem('design-tweaks', JSON.stringify(next));
    } catch {}
  };

  const reset = () => {
    setTweaks(TWEAK_DEFAULTS);
    try {
      localStorage.removeItem('design-tweaks');
    } catch {}
  };

  return { tweaks, update, reset };
}
```

### Tweaks面板UI

右下角浮動面板。可摺疊：

```jsx
function TweaksPanel() {
  const { tweaks, update, reset } = useTweaks();
  const [open, setOpen] = React.useState(false);

  return (
    <div style={{
      position: 'fixed',
      bottom: 20,
      right: 20,
      zIndex: 9999,
    }}>
      {open ? (
        <div style={{
          background: 'white',
          border: '1px solid #e5e5e5',
          borderRadius: 12,
          padding: 20,
          boxShadow: '0 10px 40px rgba(0,0,0,0.12)',
          width: 280,
          fontFamily: 'system-ui',
          fontSize: 13,
        }}>
          <div style={{ 
            display: 'flex', 
            justifyContent: 'space-between', 
            alignItems: 'center',
            marginBottom: 16,
          }}>
            <strong>Tweaks</strong>
            <button onClick={() => setOpen(false)} style={{
              border: 'none', background: 'none', cursor: 'pointer', fontSize: 16,
            }}>×</button>
          </div>

          {/* 顏色 */}
          <label style={{ display: 'block', marginBottom: 12 }}>
            <div style={{ marginBottom: 4, color: '#666' }}>主色</div>
            <input 
              type="color" 
              value={tweaks.primaryColor} 
              onChange={e => update({ primaryColor: e.target.value })}
              style={{ width: '100%', height: 32 }}
            />
          </label>

          {/* 字號slider */}
          <label style={{ display: 'block', marginBottom: 12 }}>
            <div style={{ marginBottom: 4, color: '#666' }}>字號 ({tweaks.fontSize}px)</div>
            <input 
              type="range" 
              min={12} max={24} step={1}
              value={tweaks.fontSize}
              onChange={e => update({ fontSize: +e.target.value })}
              style={{ width: '100%' }}
            />
          </label>

          {/* 密度選項 */}
          <label style={{ display: 'block', marginBottom: 12 }}>
            <div style={{ marginBottom: 4, color: '#666' }}>密度</div>
            <select 
              value={tweaks.density}
              onChange={e => update({ density: e.target.value })}
              style={{ width: '100%', padding: 6 }}
            >
              <option value="compact">緊湊</option>
              <option value="comfortable">舒適</option>
              <option value="spacious">寬鬆</option>
            </select>
          </label>

          {/* 暗黑模式toggle */}
          <label style={{ 
            display: 'flex', 
            alignItems: 'center',
            gap: 8,
            marginBottom: 16,
          }}>
            <input 
              type="checkbox" 
              checked={tweaks.dark}
              onChange={e => update({ dark: e.target.checked })}
            />
            <span>暗黑模式</span>
          </label>

          <button onClick={reset} style={{
            width: '100%',
            padding: '8px 12px',
            background: '#f5f5f5',
            border: 'none',
            borderRadius: 6,
            cursor: 'pointer',
            fontSize: 12,
          }}>重置</button>
        </div>
      ) : (
        <button 
          onClick={() => setOpen(true)}
          style={{
            background: '#1A1A1A',
            color: 'white',
            border: 'none',
            borderRadius: 999,
            padding: '10px 16px',
            fontSize: 12,
            cursor: 'pointer',
            boxShadow: '0 4px 12px rgba(0,0,0,0.15)',
          }}
        >⚙ Tweaks</button>
      )}
    </div>
  );
}
```

### 應用Tweaks

在主組件裡用Tweaks：

```jsx
function App() {
  const { tweaks } = useTweaks();

  return (
    <div style={{
      '--primary': tweaks.primaryColor,
      '--font-size': `${tweaks.fontSize}px`,
      background: tweaks.dark ? '#0A0A0A' : '#FAFAFA',
      color: tweaks.dark ? '#FAFAFA' : '#1A1A1A',
    }}>
      {/* 你的內容 */}
      <TweaksPanel />
    </div>
  );
}
```

CSS裡用變量：

```css
button.cta {
  background: var(--primary);
  color: white;
  font-size: var(--font-size);
}
```

## 典型 Tweak 選項

給不同類型的設計加什麼tweaks：

### 通用
- 主色（color picker）
- 字號（slider 12-24px）
- 字型（select：display font vs body font）
- 暗黑模式（toggle）

### 幻燈片deck
- 主題（light/dark/brand）
- 背景樣式（solid/gradient/image）
- 字體對比（更裝飾 vs 更剋制）
- 信息密度（minimal/standard/dense）

### 產品原型
- 佈局變體（layout A / B / C）
- 交互速度（animation speed 0.5x-2x）
- 數據量（mock數據條數 5/20/100）
- 狀態（empty/loading/success/error）

### 動畫
- 速度（0.5x-2x）
- 循環（once/loop/ping-pong）
- Easing（linear/easeOut/spring）

### Landing page
- Hero風格（image/gradient/pattern/solid）
- CTA文案（幾種變體）
- 結構（single column / two column / sidebar）

## Tweaks設計原則

### 1. 有意義的選項，不是折騰人的

每個tweak必須展示**真實的設計選項**。別加那種誰都不會真切換的tweak（比如border-radius 0-50px的slider——用戶調完發現所有中間值都醜）。

好的tweak暴露**離散的、有思考的variations**：
- "圓角風格"：無圓角 / 微圓角 / 大圓角（三個選項）
- 不是："圓角"：0-50px slider

### 2. 少即是多

一個設計的Tweaks面板**最多5-6個**選項。再多就變成"配置頁面"，失去了快速探索variations的意義。

### 3. 默認值是完成設計

Tweaks是**錦上添花**。默認值必須本身就是一個完整、可發佈的設計。用戶關閉Tweaks面板後看到的就是產出。

### 4. 合理分組

選項多時分組顯示：

```
---- 視覺 ----
主色 | 字號 | 暗黑模式

---- 佈局 ----
密度 | 側欄位置

---- 內容 ----
顯示數據量 | 狀態
```

## 向前兼容源碼級持久化 host

如果你以後想把設計上傳到支持源碼級 tweaks（如 Claude.ai Artifacts）的環境也能跑，保留 **EDITMODE 標記塊**：

```jsx
const TWEAK_DEFAULTS = /*EDITMODE-BEGIN*/{
  "primaryColor": "#D97757",
  "fontSize": 16,
  "density": "comfortable",
  "dark": false
}/*EDITMODE-END*/;
```

標記塊在 localStorage 方案裡**無作用**（只是個普通註釋），但在支持源碼回寫的 host 裡會被讀取，實現源碼級持久化。加上這個對當前環境無害，同時保持向前兼容。

## 常見問題

**Tweaks面板擋住設計內容**
→ 讓它可關閉。默認關閉，顯示一個小按鈕，用戶點了才展開。

**用戶切換tweaks後還要重複設置**
→ 已經用localStorage。如果刷新後不持久，檢查localStorage是否可用（無痕模式會失敗，要catch）。

**多個HTML頁面想共享tweaks**
→ 給localStorage key加project name：`design-tweaks-[projectName]`。

**我想讓tweak之間有聯動關係**
→ 在`update`里加邏輯：

```jsx
const update = (patch) => {
  let next = { ...tweaks, ...patch };
  // 聯動：選dark mode時自動切換字體配色
  if (patch.dark === true && !patch.textColor) {
    next.textColor = '#F0EEE6';
  }
  setTweaks(next);
  localStorage.setItem(...);
};
```
