# Design Context：從已有上下文出發

**這是這個skill最重要的one thing。**

好的hi-fi設計一定是從已有design context長出來的。**憑空做hi-fi是last resort，一定會產出generic的作品**。所以每次設計任務開始，先問：有沒有可以參考的東西？

## 什麼是Design Context

按優先級從高到低：

### 1. 用戶的Design System/UI Kit
用戶自己產品已有的組件庫、色彩token、字型規範、icon系統。**最完美的情況**。

### 2. 用戶的Codebase
如果用戶給了代碼庫，裡面就有活生生的組件實現。Read那些組件文件：
- `theme.ts` / `colors.ts` / `tokens.css` / `_variables.scss`
- 具體的組件（Button.tsx、Card.tsx）
- Layout scaffold（App.tsx、MainLayout.tsx）
- Global stylesheets

**讀代碼抄exact values**：hex codes、spacing scale、font stack、border radius。不要憑記憶重畫。

### 3. 用戶已發佈的產品
如果用戶有上線的產品但沒給代碼，用Playwright或讓用戶提供截圖。

```bash
# 用Playwright截圖一個公開URL
npx playwright screenshot https://example.com screenshot.png --viewport-size=1920,1080
```

讓你看到真實的視覺vocabulary。

### 4. 品牌指南/Logo/已有素材
用戶可能有：Logo文件、品牌色規範、營銷物料、slide模板。這些都是context。

### 5. 競品參考
用戶說"像XX網站那樣"——讓他提供URL或截圖。**不要**憑你訓練數據裡的模糊印象做。

### 6. 已知的design system（fallback）
如果以上都沒有，用公認的設計系統作為base：
- Apple HIG
- Material Design 3
- Radix Colors（配色）
- shadcn/ui（組件）
- Tailwind默認palette

明確告訴用戶你用的什麼，讓他知道這是起點不是定稿。

## 獲取Context的流程

### Step 1：問用戶

任務開始時的必問清單（來自`workflow.md`）：

```markdown
1. 你有現成的design system/UI kit/組件庫嗎？在哪？
2. 有品牌指南、色彩/字體規範嗎？
3. 可以給我現有產品的截圖或URL嗎？
4. 有codebase我可以讀嗎？
```

### Step 2：用戶說"沒有"時，幫他找

別直接放棄。嘗試：

```markdown
讓我看看有沒有線索：
- 你之前的項目有相關設計嗎？
- 公司的marketing網站用什麼色彩/字型？
- 你產品的Logo什麼風格？能給我一張嗎？
- 有什麼你欣賞的產品作為參考？
```

### Step 3：Read所有能找到的context

如果用戶給了codebase路徑，你讀：
1. **先list文件結構**：找style/theme/component相關的文件
2. **讀theme/token文件**：lift具體的hex/px values
3. **讀2-3個代表性組件**：看視覺vocabulary（hover state、shadow、border、padding node pattern）
4. **讀global stylesheet**：基礎重置、font loading
5. **如果有Figma鏈接/截圖**：看圖，但**更相信代碼**

**重要**：**不要**看了一眼就憑印象做。讀下來有30+個具體values才真的lift到了。

### Step 4：Vocalize你要用的系統

看完context後，告訴用戶你要用的系統：

```markdown
根據你的codebase和產品截圖，我提煉的設計系統：

**色彩**
- Primary: #C27558（從tokens.css）
- Background: #FDF9F0
- Text: #1A1A1A
- Muted: #6B6B6B

**字型**
- Display: Instrument Serif（從global.css的@font-face）
- Body: Geist Sans
- Mono: JetBrains Mono

**Spacing**（來自你的scale系統）
- 4, 8, 12, 16, 24, 32, 48, 64

**Shadow pattern**
- `0 1px 2px rgba(0,0,0,0.04)`（subtle card）
- `0 10px 40px rgba(0,0,0,0.1)`（elevated modal）

**Border-radius**
- 小組件 4px，卡片 12px，按鈕 8px

**component vocabulary**
- Button：filled primary，outlined secondary，ghost tertiary，全部圓角8px
- Card：白色背景，subtle shadow，無border

我按這套系統開始做。確認沒問題？
```

用戶確認後再動手。

## 憑空做設計（沒Context時的 fallback）

**強烈警告**：這種情況下的產出質量會顯著下降。明確告訴用戶。

```markdown
你沒有design context，我就只能基於通用直覺做。
產出會是"看起來OK但缺乏獨特性"的東西。
你願意繼續，還是先補一些參考材料？
```

用戶執意要你做，按這個順序做決策：

### 1. 選一個aesthetic direction
不要給generic結果。挑一個明確方向：
- brutally minimal
- editorial/magazine
- brutalist/raw
- organic/natural
- luxury/refined
- playful/toy
- retro-futuristic
- soft/pastel

告訴用戶你選了哪個。

### 2. 選一個known design system作為骨架
- 用Radix Colors做配色（https://www.radix-ui.com/colors）
- 用shadcn/ui做組件vocabulary（https://ui.shadcn.com）
- 用Tailwind spacing scale（4的倍數）

### 3. 選有特點的字體配對

不要用Inter/Roboto。建議組合（從Google Fonts白嫖）：
- Instrument Serif + Geist Sans
- Cormorant Garamond + Inter Tight
- Bricolage Grotesque + Söhne（付費）
- Fraunces + Work Sans（注意Fraunces已經被AI用爛）
- JetBrains Mono + Geist Sans（technical feel）

### 4. 每個關鍵決策都有reasoning

不要默默選。在HTML的comment裡寫：

```html
<!--
Design decisions:
- Primary color: warm terracotta (oklch 0.65 0.18 25) — fits the "editorial" direction  
- Display: Instrument Serif for humanist, literary feel
- Body: Geist Sans for cleanness contrast
- No gradients — committed to minimal, no AI slop
- Spacing: 8px base, golden ratio friendly (8/13/21/34)
-->
```

## Import策略（用戶給了codebase）

如果用戶說"import這個codebase做參考"：

### 小型（<50文件）
全部Read，把context內化。

### 中型（50-500文件）
Focus在：
- `src/components/` 或 `components/`
- 所有styles/tokens/theme相關的文件
- 2-3個代表性的整頁組件（Home.tsx、Dashboard.tsx）

### 大型（>500文件）
讓用戶指明focus：
- "我要做settings頁面" → 讀現有的settings相關
- "我要做一個新的feature" → 讀整體shell + 最接近的參考
- 不求全，求準

## 和Figma/設計稿的配合

如果用戶給了Figma鏈接：

- **不要**期望你能直接"轉Figma為HTML"——那需要額外工具
- Figma鏈接通常不公開可訪問
- 讓用戶：導出為**截圖**發給你 + 告訴你具體的color/spacing values

如果只給了Figma截圖，告訴用戶：
- 我能看到視覺，但取不到精確values
- 關鍵數字（hex、px）請告訴我，或者export as code（Figma支持）

## 最後的提醒

**一個項目的設計質量上限，由你拿到的context質量決定**。

花10分鐘收集context，比花1小時憑空畫hi-fi更有價值。

**遇到沒context的情況，優先問用戶要，而不是硬上**。
