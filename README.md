# HW3 Report

## Catalogue

0. [前情提要](#前情提要)
1. [有做的功能列表](#有做的功能都統整在這了)
2. [結構](#結構)
3. [功能](#功能)
4. [遊戲規則](#遊戲規則)
5. [快捷鍵](#快捷鍵)
6. [設定](#settings)
7. [Storage](#storage)
8. [Sound Code說明](#sound-code說明)
9. [桌面Texture](#桌面texture)
10. [Code Implementation Structure](#code-implement-structure)
11. [能調的地方](#能調的地方)

---

## 前情提要
>這是 <mark>1 player vs 1 dealer</mark> 的 BlackJack Game

> 開啟方式是 直接開 <mark>blackjack.html</mark>

## 有做的功能跟特色都統整在這了
- 所有的BlackJack規則 (包含Stands on soft17 開關)
- 發牌動畫
- 籌碼動畫
- Penetration 跟 Decks
- 音效 ( bgm & 發牌 & Win/Lose & 下籌碼 )
- Display button (localStorage & sessionStorage)
- Cheat button (dealer win)
- Strategy hint button
- 鍵盤快捷鍵(畫面上有使用方式)
- 嘗試模仿毛氈桌面的效果
- 上面做了一疊牌 (雖然這好像不是功能)
- Win/Lose亮框框
- 右上角統計 **手數** 跟 **Win/Tie/Loss** 跟 **ROI (投報率)**
- R可以一直加錢 (課金換籌碼的意思) 一直玩一直爽

---

## 結構

```
HW3_112550026/
├── blackjack.html          # main page (開這個)   
├── sounds/                 # 音效
│   ├── bgm.mp3             # background music 
│   ├── deal.mp3            # 發牌
│   ├── chips.mp3           # 下注籌碼
│   ├── win.mp3             # win
│   └── fail.mp3            # lose
└── README.md               # this file
```

---

## 功能

### 遊戲功能說明

| 功能 | 說明 |
|------|------|
| **發牌 (Deal)** | 開新回合，發放玩家和莊家初始牌 |
| **要牌 (Hit)** | 往手中要一張新牌 |
| **停牌 (Stand)** | 保持當前牌，結束玩家回合 |
| **加倍 (Double)** | 加倍下注並獲得一張牌（只能在初始手牌用） |
| **分牌 (Split)** | 將相同點數的兩張牌分兩手（如果一開始發到成對卡牌的時候能點） |
| **投降 (Surrender)** | 放棄當前牌，返還一半下注金額 |
| **保險 (Insurance)** | 莊家亮 A 時可下注（最高為50%基本注） |

### 下注

**籌碼面額：** $5, $25, $50, $100, $500

**操作方式：**
- 點擊籌碼按鈕增加下注
- 顯示當前下注額
- 按Backspace清空下注

### 統計 (Display按鈕)

**即時顯示：**
| 欄位 | 說明 |
|------|------|
| **資金 (Bankroll)** | 可用於下注的金額 |
| **已完成的牌 (Hands)** | 完成的回合數 |
| **勝利 (W)** | 贏得的手牌數 |
| **平局 (P)** | 平的手牌數 |
| **失敗 (L)** | 輸的手牌數 |
| **ROI (%)** | 投資回報率（利潤/下注總額） |
| **保險 (Insurance)** | 莊家亮 A 時可下注（最高為50%基本注） |

### Reset 按鈕

**功能：完全重置遊戲**

會清空所有數據：

| 清空項目 | 效果 |
|---------|------|
| 資金 | 重置為 $1,000 |
| 統計數據 | W/P/L/Hands 全部歸零 |
| 當前遊戲進度 | 停止並清空 |
| 回合歷史記錄 | 清空 Storage 中的所有記錄 |
| 牌面數據 | 當前會話數據清空 |
| 設置 | 恢復到預設 |


---

### Cheat 按鈕

**功能：啟用作弊模式，保證莊家贏得本回合**

**使用方式：**
```
1. 進行下注（放置籌碼）
2. 點 "Cheat" 按鈕
3. 提示：「Cheat enabled: Dealer will win this round.」
4. 點 "Deal" 開始遊戲
5. 莊家會自動要牌並必然贏得本回合
6. 下一回合作弊模式自動關閉
```

**原理：**

1. **計算玩家最好點數**
   - 評估所有玩家手牌的最高有效點數

2. **尋找致勝卡牌**
   - 在牌堆中找能打敗玩家但不爆牌的牌
   - 滿足：`玩家點數 < 卡牌點數 ≤ 21`

3. **重排**
   - 如無合適的牌，會自動將牌堆中的最優牌移到頂部
   - 用 `bringCardToTop()` 重排

4. **持續要牌**
   - 直到莊家：
     - ≥ 17（滿足停牌條件）
     - **且 > 玩家最好點數（確保贏）**

**Code:** `dealerPlay()` 內的 `if (CHEAT_ON)` 

---

### 動畫

- 籌碼彈跳動畫（下注時）
- 贏/輸/平局的框框發光效果
- 發牌動畫
- 正在操作的牌有框框（Split時有分割線）
- 按鈕啟用/禁用效果


---

## 遊戲規則

### 規則說明

**目標：** 讓手牌總點數盡可能接近 21，但不超過 21

**卡牌點值：**
- 2-10 - 點數跟牌數字一樣
- J、Q、K - 10 點
- A(Ace) - 11 點或 1 點（自動選最佳值）

**牌的判定：**

```
總點數 ≤ 21     - 有效
總點數 > 21     - Bust，牌失敗
包含soft 17     - 有A當11的牌（例：A+6 = 17）
包含hard 17     - A全部都不當11的牌（例：10+A+6 = 17）
```

### 遊戲流程

#### 1. 下注
```
→ 點籌碼或鍵盤數字鍵 (1-5) 下注
→ 或是可以Backspace清空
→ 最小下注 ≥ $1
→ 不能超過當前Bankroll
```

#### 2. 發牌
```
→ 點 "Deal" 按鈕或按空格開始
→ 玩家發 2 張卡牌（正面）
→ 莊家發 2 張卡牌（1正1背）
→ 檢查 Blackjack（BJ）
```

#### 3. 玩家操作
```
→ 可以選：
  - Hit (H)：要牌
  - Stand (S)：停牌
  - Double (D)：加倍
  - Split (X)：分牌
  - Surrender (U)：投降
  - Insurance (I)：買保險（if莊家亮A）
→ 點數到21或爆牌就自動結束
```

#### 4. 莊家操作
```
→ 翻開莊家的底牌
→ 自動要/停牌：(要求是Hit on Soft17所以預設關掉了)
  - 莊家總點數 < 17：必須要牌
  - 莊家總點數 ≥ 17：停牌
  （if啟用下方按鈕 "Stand on Soft 17"，則soft 17 就停牌了）
```

#### 5. 結算
```
→ 比較點數，判定結果：
  - Win-BJ：玩家 21 點，莊家非 21 點 → 賠率 3:2
  - Win：玩家點數 > 莊家 → 賠率 1:1
  - Tie：雙方點數相同 → 返還下注
  - Lose：玩家點數 < 莊家或爆牌 → 失敗
  - Surrender：已投降 → 損失50%下注
```

### 特殊情況

#### Blackjack
- **條件：** 初始 2 張牌點數為 21（10 值卡 + A）
- **賠率：** 1.5 倍下注（pay 1 + 1.5）
- **特例：** 如莊家也是 BJ，則為平局

#### Split（分牌）
- **條件：** 初始 2 張牌點數相同或都是 10 (10/J/Q/K任意配)的牌
- **操作：** 將兩張牌分成兩手，下注額相同
- **限制：** 每回合只能分一次，分手後可再要牌或加倍
- **特例：** A-A 分後只能各要一張牌（無法再分或加倍）

#### Double Down（加倍）
- **條件：** 初始 2 張牌時
- **操作：** 加倍下注並恰好獲得 1 張卡牌
- **限制：** 加倍後不能再加倍或分牌

#### Insurance（保險）
- **觸發：** 莊家亮 A 時
- **下注：** 最高為基本下注的一半
- **賠率：** 如莊家為 BJ → 賠率 2:1；否則損失保險額
- **獨立結算：** 保險結果與主牌組結果獨立

#### Surrender（投降）
- **條件：** 初始 2 張牌且**未進行操作** (操作後就無法投降)
- **效果：** 損失下注的一半，該局結束

### 牌堆規則

**牌堆配置：**
- 使用 **6 副牌** 的標準牌堆（0-6 副牌可調）
- 每副牌包含 52 張（13 種數字 × 4 種花色）
- 總計 312 張牌

**洗牌規則：**
1. **啟動洗牌**：牌堆空或到達切牌線 (Penetration 75%)
2. **切牌線**：預設為 75% 位置（可調）
3. **Fisher-Yates 洗牌算法**：確保隨機分布

---

## 快捷鍵

### 操作

| 快捷鍵 | 功能 | 適用場景 |
|------|------|---------|
| **空格** | Deal（發牌） | 下注後，準備階段 |
| **H** | Hit（要牌） | 遊戲進行中 |
| **S** | Stand（停牌） | 遊戲進行中 |
| **D** | Double（加倍） | 有初始手牌時 |
| **X** | Split（分牌） | 有配對手牌時 |
| **U** | Surrender（投降） | 初始手牌時 |
| **I** | Insurance（保險） | 莊家亮 A 時 |
| **N** | New Round（新回合） | 結算後 |

### 下注

| 快捷鍵 | 功能 | 等同於 |
|------|------|--------|
| **1** | 下注 $5 | 點擊 $5 籌碼 |
| **2** | 下注 $25 | 點擊 $25 籌碼 |
| **3** | 下注 $50 | 點擊 $50 籌碼 |
| **4** | 下注 $100 | 點擊 $100 籌碼 |
| **5** | 下注 $500 | 點擊 $500 籌碼 |
| **Backspace** | 清空下注 | 返還所有籌碼 |

### 特殊

| 快捷鍵 | 功能 |
|------|------|
| **R** | 課金！+$1,000 (Rebuy) |
| **B** | 切換基本策略提示 |

---

## Settings

### Settings 在左下角

### 1. Stands on Soft 17 Hit (軟 17 停牌，預設關)
```
☑ 關閉（莊家軟 17 停牌 - 對玩家有利）
☐ 開啟（莊家軟 17 要牌 - 對莊家有利）
```
>補充一下 關了才是符合PPT要求的

**例如：**<br>
- A + 6 = 軟 17 
- 10 + A + 6 = 硬 17
>看有沒有牌 A當11，有就是soft

### 2. Sounds (預設開)
```
☐ 關閉 (no 音效 & 背景音樂)
☑ 開啟 (所有音效on)
```

**音效：**
- 背景音樂 - 循環播（音量 0.25）
- 發牌 - 發牌播（音量 1.0）
- 籌碼 - 下注播（音量 0.5）
- 勝利 - 贏了播（音量 0.6）
- 失敗 - 輸了播（音量 0.6）

### 3. Basic Strategy

```
☑ 關閉（不顯示提示）
☐ 開啟（顯示最優決策提示）
```
> 不會玩可以開，他會告訴你可以怎麼操作

### 剛剛說可以調的參數（以下是預設）

```javascript
// 在 init 前面
decks: 6                    // 用的牌幾套
penetration: 0.75          // 洗牌點（0.0-1.0）
s17: false                 // 莊家軟 17 是否停牌
audio: true                // 聲音開
basic: false               // 策略關
```

---

## Storage

### localStorage（本地存）

**key 1:** `bj_state_v1`

**Contents** :
```javascript
{
  bankroll: 1000,          
  decks: 6,               
  penetration: 0.75,      
  stats: {
    hands: 0,               // 已玩幾手
    w: 0,                   // 勝場
    p: 0,                   // 平場
    l: 0,                   // 敗場
    profit: 0,              // 利潤
    wagered: 0              // 下注總額
  },
  settings: {
    s17: false,             // stand on soft 17?
    audio: true,            // 聲音
    basic: false            // 基本策略
  }
}
```

**存法：**
- init時read
- 下注/操作/結算後立刻存
- 設置更改時存

### sessionStorage（當前窗口存）

**key 2:** `bj_session_v1`

**Contents** : Game Status
```javascript
{
  round: {
    shoe: [],              // 剩下的牌堆
    index: 0,              // 當前抽牌位置
    discard: [],           // 已用牌
    playerHands: [],       // 玩家牌
    dealer: [],            // 莊家牌
    active: false,         // 遊戲是進行中？
    activeHandIndex: 0,    // 當前操作的牌索引
    baseBet: 0,            // 基本下注
    insuranceBet: 0,       // 保險額
    splitPerformed: false  // 分牌？
  }
}
```

**key 3:** `bj_session_rounds_v1`

**Contents** : 當前session每round牌的記錄
```javascript
[
  {
    round: 1,
    playerCards: "K♠ 5♥",    // 玩家牌
    dealerCards: ["10♦"]       // 莊家亮牌
  }
  // ...more rounds
]
```

### localStorage（歷史記錄）

**key 4:** `bj_rounds_v1`

**Contents** : 所有round的摘要
```javascript
[
  {
    round: 1,
    time: "2025-11-04T10:30:00.000Z",  // ISO 時間戳
    playerPoints: 19,       // 玩家點數
    dealerPoints: 17,       // 莊家點數
    winner: "Player",       // 贏家
    bankrollAfter: 1050     // 玩家剩餘Bankroll
  }
  // ... more rounds 
]
```

### 看數據

**看的方式：**
1. 點 `Display`按鈕
2. 有兩個表格：
   - **Local Storage Rounds** - Round History
   - **Session Storage Rounds** - 當前session的牌細節

**清空data：**
- 點擊 "Reset" 按鈕 → 清掉所有data

---

## Sound Code說明：

### Sound

```javascript
const SND = {
  bgm: { Audio object, volume: 0.25, loop: true },
  deal: { Audio object, volume: 1.0 },
  chips: { Audio object, volume: 0.5 },
  win: { Audio object, volume: 0.6 },
  fail: { Audio object, volume: 0.6 }
};
playBGM()     
stopBGM()     
toggleBGM()    
```

### 播放方式

#### 背景音樂（BGM）
```javascript
// init自動播
init() → playBGM()

audioEl.addEventListener('change', () => {
  state.settings.audio = audioEl.checked
  toggleBGM()
})
```

#### 音效
```javascript
// 標準播放（可重疊）
if (state.settings.audio) {
  SND[type].play()
}

// 中斷播放- 用於籌碼（以防點太快）
if (state.settings.audio) {
  SND.chips.currentTime = 0  // 重置到開始
  SND.chips.play()           // 播
}
```

---

## Code Implement Structure

### 結構寫法

```javascript
(function() {
  // tools
  const $ = selector => document.querySelector(selector)
  const $$ = selector => document.querySelectorAll(selector)
  
  // const
  const RANKS = ['A','2',...,'K']
  const SUITS = ['♠','♥','♦','♣']
  
  // status
  const state = { bankroll, decks, stats, settings }
  const round = { shoe, dealer, playerHands, ... }
  
  // logic
  function deal() { ... }
  function playerHit() { ... }
  function settle() { ... }
  
  // animation
  async function dealAnim(card, target) { ... }
  
  // events
  dealBtn.addEventListener('click', deal)
  
  // init
  function init() { ... }
  init()
})()
```

### 算法

#### 1. Fisher-Yates 洗牌算法
```javascript
function newShoe(decks) {
  const cards = []
  // 生成 52*decks 張牌
  
  // Fisher-Yates 隨機排
  for (let i = cards.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
    [cards[i], cards[j]] = [cards[j], cards[i]]
  }
  return cards
}
```

#### 2. 牌組評估
```javascript
function handValue(cards) {
  let base = 0, aces = 0
  
  for (const c of cards) {
    if (c.r === 'A') {
      aces++
      base += 1
    } else {
      base += cardValue(c.r)
    }
  }
  
  // 如有 A，試試當 11
  const soft = aces > 0 && base + 10 <= 21
  const total = soft ? base + 10 : base
  
  return { total, soft }
}
```

#### 3. 發牌動畫
```javascript
// 從牌堆飛過去
const makeTransform = (x, y, extraY, rot, scale) =>
  `translate3d(${x}px, ${y + extraY}px, 0) 
   translateZ(0) scale(${scale}) rotate(${rot}deg)`

// 開始：高度up + rotate
startY += verticalLift (220px)

// 結束：降低 + rotate
endY -= verticalLift / 5
```


#### 4. 莊家 - `dealerPlay()`

**標準（no cheat）：**
```javascript
while (true) {
  const v = handValue(round.dealer);
  const soft = v.soft;
  const total = v.total;
  
  const needHit = 
    total < 17 ||
    (!state.settings.s17 && total === 17 && soft);
  
  if (needHit) {
    const c = drawCard();
    round.dealer.push(c);
    await dealAnim(c, dealerHandEl, true);
    await sleep(200);
  } else {
    break;
  }
}
```

**作弊（CHEAT_ON = true）：**
```javascript
if (CHEAT_ON) {
  // 算玩家最好點數
  const playerBest = Math.max(
    ...round.playerHands.map(h => {
      const v = handValue(h.cards);
      return v.total > 21 ? 0 : v.total;
    })
  );
  
  // 找能贏但不爆的牌
  while (true) {
    const v = handValue(round.dealer);
    if (v.total >= 17 && v.total > playerBest) break;
    
    // 找最佳牌：贏又不爆
    const minNeed = Math.max(playerBest + 1 - v.total, 1);
    const maxSafe = 21 - v.total;
    
    const ok = bringCardToTop((c) => {
      const vv = valueOfRank(c.r);
      return (vv >= minNeed && vv <= maxSafe) ||
             (c.r === 'A' && 1 >= minNeed && 1 <= maxSafe);
    });
    
    const c = drawCard();
    round.dealer.push(c);
    await dealAnim(c, dealerHandEl, true);
  }
}
```

#### 5. 結果
```javascript
for (const outcome of outcomes) {
  const pv = handValue(playerHand).total
  const dv = handValue(dealerHand).total
  
  if (pv > 21) result = 'lose'           // 爆牌
  else if (dv > 21) result = 'win'       // 莊家爆
  else if (pv > dv) result = 'win'       // 點數更高
  else if (pv < dv) result = 'lose'      // 點數更低
  else result = 'tie'                    // 相等
}
```


---

## 能調的地方

### 遊戲內容

#### 改變牌堆數量
```html
decks: 6
```

#### 洗牌點
```html
enetration: 0.75
改為：penetration: 0.5  // 更早洗牌
```

#### 改籌碼
```javascript
const CHIP_SET = [5, 25, 50, 100, 500]
改為：const CHIP_SET = [10, 50, 100, 500, 1000]
```

#### 改初始Bankroll
```javascript
bankroll: 1000
改為：bankroll: 5000
```

### 視覺效果

#### 改桌面顏色
```css
--felt: #065f46
改為：--felt: #1a472a
```

#### 改牌尺寸
```css
--card-w: 86px
改為：--card-w: 100px
```

#### 改動畫速度
```css
--deal-ms: 600
改為：--deal-ms: 1000  // 更慢的動畫
```

### 改音頻

#### 改音量
```javascript
audio.volume = 0.25
改為：audio.volume = 0.5
```

#### 加音效
```javascript
// 在 SND 加
newSound: (() => {
  const audio = new Audio('sounds/newsound.mp3')
  audio.volume = 0.4
  return audio
})()
```

---

## 桌面Texture

### 整體設計理念

背景用**多層紋理疊加**，去模擬賭場毛氈桌面

### Body Background

**8 層紋理疊起來：**

#### 1. 紋理層 1 - 斜紋 (45°)
```css
repeating-linear-gradient(
  45deg,
  transparent,
  transparent 1px,
  rgba(255, 255, 255, 0.08) 1px,
  rgba(255, 255, 255, 0.08) 3px
)
```

#### 2. 紋理層 2 - 反斜紋 (-45°)
```css
repeating-linear-gradient(
  -45deg,
  transparent,
  transparent 2px,
  rgba(0, 0, 0, 0.12) 2px,
  rgba(0, 0, 0, 0.12) 5px
)
```
#### 3. 毛氈 - 孔洞(垂直)
#### 4. 毛氈 - 波紋(水平)
#### 5. 細微凹凸 
#### 6. 光線層 
#### 7. 陰影層 
#### 8. 中心聚焦

**Background Size 配置：**
```css
background-size: 
  100% 100%,    /* 1-4 */
  100% 100%,
  4px 100%,     /* 3 */
  100% 4px,     /* 4 */
  4px 4px,      /* 5 */
  100% 100%,    /* 6-8 */
  100% 100%,
  100% 100%;
```

### 操作桌面毛氈效果（.felt）
**6 層紋理**

---
