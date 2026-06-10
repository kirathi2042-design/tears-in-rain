# TEARS // RAIN — 雨中之淚

> *"All those moments will be lost in time, like tears in rain."*

Blade Runner 風格的音樂視覺化器。**單一 HTML 檔案**，零相依套件，純 WebGL2 fragment shader + Web Audio API。打開就能用。

**Live Demo → https://kirathi2042-design.github.io/tears-in-rain/**

## 八種視覺核心

| # | 模式 | 說明 |
|---|------|------|
| 01 | SMOKE 煙霧 | 畫面四周設八個入煙口，每個節拍隨機開一口放煙進場，煙團邊前進邊翻捲膨脹 |
| 02 | FIRE 火焰 | 低音與重拍改變火焰高度、亮度、色溫與白熱核心 |
| 03 | LAVA 熔岩燈 | Metaball 熔岩燈，bass 讓蠟球膨脹 |
| 04 | WARP 蟲洞航行 | 超空間隧道：航速隨音樂、星流線隨音量拉長、頻譜化作隧道口曼陀羅、重拍從核心射出光環 |
| 05 | ROAD 都市道路 | Blade Runner 風格的濕冷都市道路、末日科技天際線、霓虹反射與濃霧 |
| 06 | AURORA 極光 | Curl-noise 光簾在極夜山稜上舞動，頻譜決定每道光簾的高度 |
| 07 | ABYSS 深海生光 | 生物發光水母群與浮游粒子，重拍讓鐘形傘收縮脈動 |
| 08 | VINYL 黑膠迴轉 | 黑膠唱片宏觀視角：旋轉針槽、頻譜刻進蠟紋、燈帶反光與灰塵微粒 |

## 音源（任選其一）

- **FILE 音檔** — 載入本機音檔，也可直接拖放到視窗。**雙引擎解碼**：先走 `<audio>` 串流（省記憶體），失敗自動 fallback 到 `decodeAudioData` 全檔解碼。實際支援格式依瀏覽器：MP3 / AAC·M4A / WAV / FLAC 全平台沒問題；OGG / Opus / WebM 在 Chrome、Firefox、新版 Safari 可用；AIFF / CAF / ALAC 在 Safari 可用。WMA / APE / WavPack 這類瀏覽器完全沒有解碼器的格式會明確提示無法播放
- **YOUTUBE 連結播放** — 貼上 YouTube 連結（支援 `watch` / `youtu.be` / `shorts`），影片嵌入本頁播放，按下 LOAD 後瀏覽器會請求分享音訊——選「**此分頁 / This Tab**」一鍵授權，視覺即時跟著影片的聲音跳動。原理：跨網域 iframe 的音訊無法直接進 Web Audio 分析器（瀏覽器安全模型），所以把影片嵌進自己的分頁再自我擷取，是靜態網頁唯一的正規解法。**桌面 Chrome / Edge 限定**（iOS / Safari 不支援分頁擷取）
- **DEMO 訊號** — 內建 96 BPM synthwave 迴圈（kick + acid bass + pad），一鍵就有東西看

iOS 上請使用 FILE 或 DEMO 模式。

檔案模式有完整播放器：播放／暫停、**0.5×–2× 倍速**、可點擊跳轉的進度條。

## 操作

| 鍵 | 功能 |
|----|------|
| `1`–`8` | 切換視覺模式 |
| `A` | 自動輪播（每 28 秒換景） |
| `H` | 隱藏 / 顯示 HUD（閒置 9 秒也會自動隱藏） |
| `F` | 全螢幕 |
| `Space` | 播放 / 暫停音檔 |

## 本機執行

```bash
# 直接開檔案即可（demo / 檔案 / 拖放都能用）
open index.html

# 麥克風與系統音訊需要 secure context，起個 server：
python3 -m http.server 8080
# → http://localhost:8080
```

## 手機操作

- 視覺模式選單在窄螢幕（≤760px）變成**左側抽屜**：點左緣的 `▶` 拉出、`◀` 收回；也可從螢幕左緣向右滑開啟、向左滑關閉。選定模式後抽屜自動收回
- 右上角顯示目前版本；按 **⟳ CHECK** 可手動讀取 `version.json` 檢查新版，有新版時按鈕會切成琥珀色 **⟳ UPDATE vX.Y.Z**，再點一下即載入新版

## 技術備忘

- WebGL2 / GLSL ES 3.0，單 pass、單三角形全螢幕 quad，八個場景都是程序化 noise/FBM/metaball，無任何貼圖資產
- `AnalyserNode`（FFT 2048）拆出 bass / mid / treble / level 四個包絡 + 能量比較式 beat detection，64-bin 頻譜以 R8 texture 餵進 shader
- 風場系統：畫面四周固定八個入煙口（E/NE/N/NW/W/SW/S/SE），每個節拍隨機選一口生成入流事件，煙頭以緩動曲線駛向中央、沿途翻捲（旋轉式 domain warp）並留下膨脹的煙尾；火焰與熔岩使用同一批事件作局部擾動。人聲中頻化為**漫遊微風**（`uBreeze`，方向緩慢漂移、空間上以 noise 調變成漣漪，絕不整面平移）
- `uHue` 色相隨時間與高頻緩慢漂移、重拍時跳一階，所有場景調色盤都會變色；道路的天空另有獨立的慢速色彩循環
- 後處理：filmic tone curve、暗角、膠片顆粒、掃描線、節拍閃光
- 檔案播放雙引擎：`<audio>` 串流優先（大檔不吃記憶體），元素拒播時自動以 `decodeAudioData` 全檔解碼改走 `AudioBufferSourceNode`（HUD 會標示 `· PCM`），兩者都解不開才報錯
- 麥克風 / 系統音訊不回送喇叭（無 feedback），檔案與 demo 正常出聲
- iOS Safari 相容：等待使用者手勢後才 `AudioContext.resume()`

## 發佈備忘

每次部署新版時，需同步 bump `index.html` 內的 `APP_VERSION` 常數與 `version.json` 的 `version` 欄位（兩者一致），線上頁面按 **⟳ CHECK** 才會偵測到更新並亮起 UPDATE 鈕。

## License

MIT
