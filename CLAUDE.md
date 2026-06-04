# 登入系統與排行榜實作計畫

## 需求摘要
1. **登入頁面**：玩家輸入暱稱 + 密碼登入
2. **註冊功能**：新玩家可註冊，暱稱不可重複
3. **排行榜**：顯示各遊戲 TOP 10 最高分玩家
4. **遊戲大廳**：
   - 新增「登入」與「註冊」按鈕
   - 未登入玩家點擊遊戲時顯示警告提示
5. **分數整合**：遊戲結束時回傳分數至大廳

---

## 資料儲存架構

使用 `localStorage`，資料結構如下：

```javascript
// 所有玩家資料
localStorage.setItem('players', JSON.stringify({
  "玩家A": { password: "密碼雜湊", scores: { "2048": 1000, "snake": 500, "maze": 120, "minesweeper": 45 } },
  "玩家B": { ... }
}))

// 目前登入玩家
localStorage.setItem('currentUser', JSON.stringify({ username: "玩家A" }))
```

> **注意**：密碼僅做簡單雜湊（實際專案建議後端處理）

---

## 實作項目

### 1. 建立 `login.html`（登入/註冊頁面）

**功能**：
- 切換「登入」與「註冊」模式
- 暱稱輸入（必填，長度 2-20 字）
- 密碼輸入（必填，長度 4-20 字）
- 驗證暱稱是否已存在（註冊模式）
- 登入成功 → 導向 `index.html`

**流程**：
- 點擊「註冊」→ 檢查暱稱是否已存在 → 存在顯示錯誤 / 不存在則寫入並自動登入
- 點擊「登入」→ 檢查暱稱+密碼是否正確 → 錯誤顯示提示 / 正確則寫入 currentUser 並跳轉

---

### 2. 修改 `index.html`（遊戲大廳）

**新增元素**：
- 右上角：使用者資訊（顯示暱稱、「登出」按鈕）或「登入/註冊」按鈕
- 排行榜面板（可折叠/展開）

**點擊遊戲卡片的邏輯**：
```
如果已登入 → 開啟遊戲
如果未登入 → 顯示警告 Modal：「不登入將無法記錄遊戲紀錄」
```

**排行榜 UI**：
- 每個遊戲一個 Tab（2048 / 貪吃蛇 / 逃出深淵 / 踩地雷）
- 顯示該遊戲 TOP 10：排名、暱稱、分數
- 分數高者在上面

---

### 3. 修改各遊戲 HTML（分數回傳）

每個遊戲結束時，需要一個機制將分數傳回 parent 頁面（index.html）。

**使用 postMessage 機制**：
```javascript
// 遊戲結束時發送分數
window.parent.postMessage({
  type: 'gameScore',
  game: '2048',      // 或 'snake', 'maze', 'minesweeper'
  score: 1234,
  username: currentUser
}, '*');

// 當 index.html 接收到訊息，更新排行榜資料
window.addEventListener('message', (e) => {
  if (e.data.type === 'gameScore') {
    updateScore(e.data.game, e.data.username, e.data.score);
  }
});
```

**各遊戲分數定義**：
| 遊戲 | 分數意義 | 備註 |
|------|---------|------|
| 2048 | 遊戲結束時的 `game.score` | 數字越大越高 |
| 蛇 | 遊戲結束時的 `score` | 每吃食物 +10 |
| 迷宮 | 通關秒數 | 越快越高（存為負值或用時間計算） |
| 地雷 | 通關秒數 | 越快越高（存為負值或用時間計算）|

> **注意**：對於時間類遊戲，為了「分數高者排名前面」，儲存時可存為「負的秒數」或用公式轉換。

---

## 頁面流程圖

```
login.html <---> index.html（遊戲大廳）
                   |
                   +--> 2048.html（postMessage 分數）
                   +--> snake.html（postMessage 分數）
                   +--> maze.html（postMessage 分數）
                   +--> minesweeper.html（postMessage 分數）
```

---

## 實作順序

1. **登入頁面 (`login.html`)**
   - 表單 UI（登入/註冊切換）
   - 暱稱、密碼驗證
   - localStorage 讀寫

2. **修改大廳 (`index.html`)**
   - 新增 header（使用者資訊 / 登入按鈕）
   - 排行榜 UI + 切換 Tab
   - 遊戲點擊前的登入檢查邏輯
   - 監聽 postMessage 更新分數

3. **修改各遊戲（回傳分數）**
   - 2048：遊戲結束時 postMessage
   - 蛇：遊戲結束時 postMessage
   - 迷宮：勝利時 postMessage
   - 地雷：勝利時 postMessage

---

## 檔案清單

| 檔案 | 變更 |
|------|------|
| `login.html` | 新建 |
| `index.html` | 修改：新增 header、排行榜、未登入警告 |
| `2048.html` | 修改：遊戲結束 postMessage |
| `snake.html` | 修改：遊戲結束 postMessage |
| `maze.html` | 修改：勝利 postMessage |
| `minesweeper.html` | 修改：勝利 postMessage |

---

## 驗證清單

- [ ] 註冊新帳號（暱稱已存在時顯示錯誤）
- [ ] 登入成功後自動跳轉至大廳
- [ ] 大廳顯示當前登入的暱稱
- [ ] 點擊「登出」可清除登入狀態
- [ ] 未登入時點擊遊戲顯示警告
- [ ] 排行榜正確顯示各遊戲 TOP 10
- [ ] 遊戲結束後分數正確寫入排行榜