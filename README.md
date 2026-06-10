# TEARS // RAIN — 雨中之淚

> *"All those moments will be lost in time, like tears in rain."*

Blade Runner 風格的音樂視覺化器。**單一 HTML 檔案**，零相依套件，純 WebGL2 fragment shader + Web Audio API。打開就能用。

**Live Demo → https://kirathi2042-design.github.io/tears-in-rain/**

## 六種視覺核心

| # | 模式 | 說明 |
|---|------|------|
| 01 | SMOKE 煙霧 | 風洞式入流束吹向中心交會區，形成剪切層與 Kármán 渦街 |
| 02 | FIRE 火焰 | 低音與重拍改變火焰高度、亮度、色溫與白熱核心 |
| 03 | RAIN 雨滴玻璃 | 從玻璃下方向上看雨滴撞擊玻璃，上表面開出透明水花、漣漪與 caustic 光環 |
| 04 | LAVA 熔岩燈 | Metaball 熔岩燈，bass 讓蠟球膨脹 |
| 05 | MIST 迷霧雨林 | 冠層、闊葉、蕨葉、遠山與低霧交疊，炫光從自然林隙穿出 |
| 06 | ROAD 都市道路 | Blade Runner 風格的濕冷都市道路、末日科技天際線、霓虹反射與濃霧 |

## 音源（任選其一）

- **FILE 音檔** — 載入本機音檔，也可直接拖放到視窗。**雙引擎解碼**：先走 `<audio>` 串流（省記憶體），失敗自動 fallback 到 `decodeAudioData` 全檔解碼。實際支援格式依瀏覽器：MP3 / AAC·M4A / WAV / FLAC 全平台沒問題；OGG / Opus / WebM 在 Chrome、Firefox、新版 Safari 可用；AIFF / CAF / ALAC 在 Safari 可用。WMA / APE / WavPack 這類瀏覽器完全沒有解碼器的格式會明確提示無法播放
- **MIC 麥克風** — 用任何裝置外放音樂（iOS 音樂 app、Spotify、黑膠、現場樂器），麥克風收音即時驅動視覺。**iPhone / iPad 上聽串流就用這個模式**（串流 DRM 音訊無法直接解碼，這是通用解法）
- **SYSTEM 系統音訊** — 桌面 Chrome / Edge 透過分頁擷取（記得勾「分享分頁音訊」），可直接吃 Spotify Web Player / YouTube 的聲音，零延遲零雜訊
- **DEMO 訊號** — 內建 96 BPM synthwave 迴圈（kick + acid bass + pad），一鍵就有東西看

檔案模式有完整播放器：播放／暫停、**0.5×–2× 倍速**、可點擊跳轉的進度條。

## 操作

| 鍵 | 功能 |
|----|------|
| `1`–`6` | 切換視覺模式 |
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

- WebGL2 / GLSL ES 3.0，單 pass、單三角形全螢幕 quad，六個場景都是程序化 noise/FBM/metaball，無任何貼圖資產
- `AnalyserNode`（FFT 2048）拆出 bass / mid / treble / level 四個包絡 + 能量比較式 beat detection，64-bin 頻譜以 R8 texture 餵進 shader
- 風場系統：重拍生成**獨立入流事件**；煙霧模式將事件解讀為風洞入流束與交替渦街，火焰與熔岩則使用同一批事件作局部擾動。人聲中頻化為**漫遊微風**（`uBreeze`，方向緩慢漂移、空間上以 noise 調變成漣漪，絕不整面平移）
- 雨滴系統：重拍生成**玻璃上表面撞擊事件**（`uRain` 事件佇列），依重音強度改變水花半徑、花瓣狀裂波、衛星水珠與 caustic 光環；視角是人在玻璃下往上看，不做低品質滑落粒子
- `uHue` 色相隨時間與高頻緩慢漂移、重拍時跳一階，所有場景調色盤都會變色；道路的天空另有獨立的慢速色彩循環
- 後處理：filmic tone curve、暗角、膠片顆粒、掃描線、節拍閃光
- 檔案播放雙引擎：`<audio>` 串流優先（大檔不吃記憶體），元素拒播時自動以 `decodeAudioData` 全檔解碼改走 `AudioBufferSourceNode`（HUD 會標示 `· PCM`），兩者都解不開才報錯
- 麥克風 / 系統音訊不回送喇叭（無 feedback），檔案與 demo 正常出聲
- iOS Safari 相容：等待使用者手勢後才 `AudioContext.resume()`

## 發佈備忘

每次部署新版時，需同步 bump `index.html` 內的 `APP_VERSION` 常數與 `version.json` 的 `version` 欄位（兩者一致），線上頁面按 **⟳ CHECK** 才會偵測到更新並亮起 UPDATE 鈕。

## License

MIT
