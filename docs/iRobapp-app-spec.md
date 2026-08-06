# iRobapp Ver.1 アプリケーション仕様書（正式版）

---

## 1. 接続・通信系（BLE / リンク）

### 1-1. iRobapp とのリンク処理
- BLE Central：iPhone  
- BLE Peripheral：ロボット（XIAO ESP32S3）  
- 通信方式：GATT（Notify / Write）  
- MTU：最大 185 bytes（iOS 標準）

### 1-2. 圧縮音声送信（ADPCM）
iRobapp Ver.1 の音声送信は **ADPCM 圧縮 → BLE → ロボット側デコード → I2S 出力** の流れで行う。

#### 音声処理フロー（1〜9）
1. ユーザーがロボットに話しかける  
2. iPhone が音声をテキストに変換（STT）  
3. テキストを AI に送信し返答を取得  
4. 返答テキストを iPhone で TTS して音声生成  
5. 生成した音声を ADPCM などで圧縮  
6. 圧縮データを BLE でロボットへ送信  
7. ロボ側（XIAO）が ADPCM をデコード  
8. デコードした PCM を I2S で MAX98357 へ送る  
9. スピーカーから音声が出力される

---

## 2. モード管理・動作制御

### 2-1. モード切替処理
アプリは以下のモードを管理する：

- **ホームモード**（接続・状態表示）
- **会話モード**（STT / AI / TTS）
- **モーションモード**（基本動作指示）
- **設定モード**（音声・AI・ユーザー設定）

### 2-2. 基本動作指示処理
アプリはロボットへ以下の動作指示を送信する：

- 姿勢：立つ / 座る / 伏せ  
- 移動：歩く / 後退 / 回る  
- 感情：尻尾ふり / 首ふり / 喜びモーション  

BLE CMD_WRITE によりモーションコマンドを送信し、  
ロボット側は STATUS_NOTIFY で進行状況を返す。

---

## 3. 音声関連（STT / TTS / 設定）

### 3-1. 音声認識（STT）
- iOS 標準の音声認識 API  
- オンライン／オフライン切替（将来対応）

### 3-2. 音声生成（TTS）
- AVSpeechSynthesizer による TTS  
- 将来的に「ロボット専用ボイスモデル」追加予定

### 3-3. 音声設定項目
- 音声の自然さ  
- 話す速さ  
- 声の高さ  
- 口調  
- 会話履歴 ON/OFF  
- テーマカラー  
- フォント  
- API キー入力（ChatGPT 利用時）

---

## 4. AI 連携（頭脳モード）

iRobapp Ver.1 の AI モードは以下の 3 種類。

### 4-1. めっちゃ賢いさん（ChatGPT・オンライン・有料）
- ChatGPT API を使用  
- ユーザーが API キーを設定  
- 最も自然で高品質な会話が可能

### 4-2. おしゃべりさん（Phi-3-mini・オフライン・無料）
- iPhone 内で LLM を動作  
- Core ML / MLX による高速推論  
- ChatGPT より自然さは落ちるが無料で利用可能

### 4-3. 賢いさん（Llama 3.2・オフライン・無料）
- iPhone 内で LLM を動作  
- Core ML / MLX  
- Phi-3 と同等のオフライン会話体験

---

## 5. モデル配布方式（重要）

### 5-1. モデル同梱方式 vs ダウンロード方式
- アプリサイズ肥大化（300〜800MB）を避けるため  
- **ダウンロード方式を採用**

### 5-2. 配布方法
- GitHub Release にモデルを配置  
- リポジトリ例：  
  `https://github.com/idelity/iRobapp-models`

### 5-3. ダウンロード例（Swift）
```swift
let url = URL(string: "https://github.com/idelity/iRobapp-models/releases/download/v1.0/phi-3-mini-q4.gguf")!
let task = URLSession.shared.downloadTask(with: url) { localURL, response, error in
    // 保存処理
}
task.resume()
```

### 5-4. 保存場所
```
Documents/Models/Phi3/
```

### 5-5. 分割 zip 対応
以下のファイルが存在しない場合、GitHub からダウンロードする：

- `Phi-3-mini-4k-instruct-q4.zip.001`  
- `Phi-3-mini-4k-instruct-q4.zip.002`

### 5-6. Core ML 変換について
- Ver.1 では **gguf のまま配布**  
- 将来的に Core ML 変換版も検討

---

## 6. 音声フォーマット仕様（TTS → ADPCM）

| 項目 | 値 |
|------|------|
| フォーマット | WAV (RIFF) |
| エンコーディング | リニアPCM（非圧縮） |
| サンプルレート | 44.1kHz |
| チャンネル数 | 1（モノラル） |
| ビット深度 | 16bit |

---

## 7. 設定（ユーザー設定）

- 音声設定（速さ・高さ・自然さ）  
- 口調設定  
- テーマカラー  
- フォント  
- 会話履歴 ON/OFF  
- AI モード選択  
- ChatGPT API キー入力  

---

## 8. 今後の拡張予定（Ver.2 以降）

- カメラ連携（ロボットの視界をアプリに表示）  
- 家族モード（複数デバイスで同一ロボットを管理）  
- 感情学習モデルの強化  
- モーションエディタ機能（ユーザーが動きを作成）  

---
