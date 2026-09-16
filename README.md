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
| 血量 | 0–255 進度條 | `Time_Ai_Handler_onItem_Collision` |
| 變大 | `scale × 1.1` 每 tick，下限 0.9 | `Action_Ai_Handler_onBallTouch` |
| 縮小 | `scale ÷ 1.3` 每 tick，下限 0.6 | `Action_Ai_Handler_onBallRelease` |
| 速度（按住） | 基礎 0.3，每 tick 加 0.003 | `onBallTouch` |
| 速度（放開） | 上限 0.2 | `onBallRelease` |
| 撞到障礙物 | **速度歸零** | `Action_Ai_Handler_onCollisionSpeed` |
| LIFE+10 / −10 | ±10，上限 255 | `onItem_Collision`（感測器 23 / 22） |
| 鯊魚 | −30 | `onItem_Collision`（感測器 30） |
| 分數 | `+= 137` 每個計分 tick | `Time_Ai_Handler_onScorecount` |
| 左右邊界 | 第1關 ±4.2、第2關 ±2.2、第3關 ±4.7 | `Action_Ai_Handler_onAutoSpeed` |
| 關卡長度 | 600 / 590 / 600 單位 | `onRoom1..4_detectz` |
| 鴨子動畫速度 | 按住 150、放開 60 | `Action_Ai_Handler_onBallBoolin` |

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
