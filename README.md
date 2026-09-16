# Rushing（當奇大脫走）— Web 重製版

2012 年世新大學新聞傳播學院數位多媒體設計學系畢業製作《Rushing》的瀏覽器重製版。

**▶️ 直接玩：https://fangwentsai.github.io/rushing-2012/**

打得開就能玩，手機和電腦都行，不用安裝。

---

## 這是什麼

你是一隻上了發條的浴室玩具鴨，**當奇**。待在浴缸裡一輩子太浪費生命了，
於是順著家裡排水的流向，經過下水道、排水溝、河道，一路衝向大海的小島。

原作 2012 年 5 月 4 日以「當奇大脫走」上架 Google Play，用 ShiVa3D 開發，
3ds Max 建模，支援 Android 2.2 以上。

## 玩法

整個遊戲只有一個難題：**衝得快**還是**閃得過**。

| 操作 | 效果 |
|---|---|
| 按住畫面 / `Space` | 當奇變大，速度變快，但碰撞體積也變大 |
| 放開 | 縮小，好閃躲，但速度掉下來 |
| `←` `→` / `A` `D` | 左右閃避 |
| 傾斜手機 | 陀螺儀操控（需先在首頁授權，iOS 必要） |

撞到障礙物扣血，血歸零就結束。

### 問號方塊

撞下去會隨機開出一個道具，好壞都有：

| 道具 | 效果 |
|---|---|
| **SPEED UP** | 短暫加速，期間分數雙倍 |
| **JUMP** | 起飛越過障礙物，滯空時無敵 |
| **LIFE+10** | 補血十點 |
| **LIFE−10** | 扣血十點 |

### 四個場景

跟著家庭用水的流向走：**下水道 → 排水溝 → 河道 → 大海**。

---

## 關於這個重製版

`Rushing2.0.stk` **已經解開了**。它不是加密，是 XOR 混淆加 zlib 壓縮，
392 個檔案全部取出且 CRC 全過（解包工具與原始檔在私有封存庫）。所以這個
Web 版用的是**當年真正的素材**，玩法數值也是從原始的 Lua bytecode 讀出來的。

### 沿用的 2012 年原始素材

- 標題主視覺、遊戲圖示、THYY logo
- 當奇的 HUD 頭像三種狀態（`life1` 正常 / `life3` 受擊 / `life2` 血量過低）
- 水流貼圖、樹、鯊魚、WARNING
- 四個道具的原圖：SPEED UP、JUMP、LIFE+10、LIFE−10
- **五首配樂**，組員當年用 Garage Band 自己做的（`.mus` 本身就是 OGG Vorbis）

### 從 Lua bytecode 還原的玩法數值

腳本是 Lua 5.0 bytecode（ShiVa 1.x 用的版本），反組譯後讀到：

| 項目 | 原始值 | 出處 |
|---|---|---|
| 畫面上方那條 | **時間**，0–255，每秒掉 1 | `Time_Ai_Handler_onTimeProcess` |
| 時間過低警示 | 低於 **50** 開始閃 `life3` | `onTimeProcess` → `onLife` |
| 結算加成 | 剩餘時間 **× 500** | `Time_Ai_Handler_onTimeCount` |
| 撞到障礙物 | `Speed = 0` 且 `bAutospeed_stat = false`，**前進暫停 0.5 秒後自動恢復**，不扣時間、不改體積 | `onCollisionSpeed` + `onAutoSpeed` 尾段 |
| 撞到時的回饋 | HUD 鴨臉閃 `life2` | `Time_Ai_Handler_onCollision_HUDtrue` |
| 變大 | `scale × 1.1`，下限 0.9 | `Action_Ai_Handler_onBallTouch` |
| 縮小 | `scale ÷ 1.3`，下限 0.6 | `Action_Ai_Handler_onBallRelease` |
| 速度（按住） | 基礎 0.3，每次 −0.003 | `onBallTouch` |
| 速度（放開） | 上限 0.2 | `onBallRelease` |
| 左右移動 | 每次 **0.1** 單位 | `Action_Ai_Handler_onLeftClick` |
| 攝影機側傾 | 轉彎時每次 +0.5 度，**上限 10 度** | `onLeftClick` |
| 攝影機基準俯角 | −15 度 | `onitem_disable` 還原值 |
| 計時器起始值 | **200**（進度條上限 255） | `Time_Ai.aim` 的 `nValue` 預設值 |
| 分數 | 每過一個區段檢查點加 `nfloor_time × 137`，然後 `nfloor_time` 重設回 20；`nfloor_time` 每秒 −1 —— **越快通過拿越多** | `onScorecount` + `onTimeProcess` + `Time_Ai.aim` |
| 鯊魚速度 | −1.5 | `Shark_ai.aim` 的 `nSpeed` |
| `Speed` 初值 | −0.2 | `Action_Ai.aim` |
| 左右邊界 | ±4.2 / ±2.2 / ±4.7 | `Action_Ai_Handler_onAutoSpeed` |
| 鴨子動畫速度 | 按住 150、放開 60 | `Action_Ai_Handler_onBallBoolin` |

### 五個道具

`onitem` 用 `math.random(0, 4)` 從五個裡**等機率**挑一個，每個效果持續 **1 秒**
（`onitem_enable` 會排一秒後的 `onitem_disable`）。

| 感測器 | 道具 | 效果 | 出處 |
|---|---|---|---|
| 21 | SPEED UP | `Speed = -1.5`，一秒後降到 `-1`（基礎值 0.3） | `onitem_enable` / `onitem_disable` |
| 22 | LIFE−10 | 時間 −10 | `Time_Ai_Handler_onItem_Collision` |
| 23 | LIFE+10 | 時間 +10，上限 255 | 同上 |
| 24 | JUMP | `translateTo(x, y+1, z−1.5)` 每 0.01 秒 —— 浮起來往前衝，不是拋物線跳躍 | `Action_Ai_Handler_onItem_moving` |
| 25 | **CAMERA** | `setRotation(Action_camera, 0, y, 180)` —— **畫面上下顛倒** | `onitem_enable` |
| 30 | 鯊魚 | 時間 −30 | `onItem_Collision` |
| 87 | WARNING | 顯示 `GameDesign.warning` | `onitem_disable` |

### 唯一調過的參數

`×1.1`、`÷1.3`、`−0.003` 都是**每次事件**的變化量。原始碼用
`user.postEvent(..., 0.001, ...)` 讓處理器自己重排，所以事件多久觸發一次是
ShiVa 執行期的設定，**不在 `.stk` 裡**。直接假設 60Hz 的話 `1.1^60` ≈ 每秒 300 倍，
完全不是遊戲的樣子。

所以比例和門檻全部照抄，只把事件率當成唯一的自由參數，調到符合當年的手感：

```js
var STEP_HZ = 7;    // 縮放與加速的速率，連續套用
```

原始碼是每次事件跳一次 `×1.1`；照著離散跳會一格一格很鈍，所以改成連續套用
`pow(1.1, dt × STEP_HZ)` —— 同樣的時間到達同樣的大小，但在任何更新率下都平滑。

按住約 1 秒從 0.9 放大到上限 1.8，放開約 0.6 秒縮回 0.6。移動則固定 60Hz，
因為那才能讓 2390 單位的總長度落在 255 秒的計時器內。想調手感就改 `STEP_HZ`。

感測器 ID 也照原本的：1/2/3 換關、4 抵達終點、21–25 是五個道具、30 是鯊魚
（見 `Player_Ai_Handler_onSensorCollisionBegin`）。

### 沒能照搬的部分

- **3D 模型**：30 個 `.msh` 都在，但那是 ShiVa 自己的二進位格式，還沒寫解析器，
  所以場景和當奇仍然是用 Three.js 程式生成的
- **體積上限**：原始碼只有下限沒有上限，這裡加了 2.2 的上限，否則會撐爆畫面
- **左右移動速度**：沒能從常數中確定，是自己調的

## 技術

單一 HTML 檔，[Three.js](https://threejs.org/) r128，沒有 build step。
本機要跑的話，因為有讀圖片，需要一個伺服器而不能直接開檔案：

```bash
python3 -m http.server 8000
```

然後開 http://localhost:8000

---

## 原作團隊（2012）

| | |
|---|---|
| ShiVa 主程式 | 蔡方文 |
| 場景模型建置 · ShiVa 副程式 | 洪偉強 |
| 遊戲企劃 · 場地模型建置 | 楊嘉瑋 |
| 美術風格 · 遊戲介面設計 | 楊國瑜 |
| 指導教授 | 施保旭、劉明昆 |

世新大學新聞傳播學院 數位多媒體設計學系 · 2012

本專案裡的 2012 年美術素材著作權屬於原作團隊全體成員。
