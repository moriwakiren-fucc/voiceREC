# REC — バックグラウンド録音 PWA

## ファイル構成
```
recorder-pwa/
├── index.html      ← メインアプリ
├── manifest.json   ← PWAマニフェスト
├── sw.js           ← Service Worker（オフライン対応）
├── icon-192.svg    ← アイコン
└── README.md
```

## デプロイ方法

### 1. HTTPSサーバーが必要
PWA・マイク・Service Worker はすべて **HTTPS** が必須です。

**おすすめの無料ホスティング：**
- [Netlify Drop](https://app.netlify.com/drop) — フォルダをドラッグ&ドロップするだけ
- [Vercel](https://vercel.com) — `vercel deploy` コマンド1つ
- GitHub Pages — リポジトリにpushするだけ

### 2. ローカルテスト（Mac/PC）
```bash
# Python 3
python3 -m http.server 8080
# → http://localhost:8080 でアクセス（localhostはHTTPS不要）
```

---

## iPadでのバックグラウンド録音 — 仕組みと制約

### ✅ 動作する方法（推奨）
1. **ホーム画面に追加（必須）**
   Safari → 共有ボタン → 「ホーム画面に追加」
   → フルスクリーン（standalone）モードで起動

2. **録音を開始してから画面をロック**
   - AudioContext + MediaRecorder が動き続ける
   - iOS 16以降はバックグラウンドオーディオが許可される
   - 録音データは1秒ごとにチャンクとして保存

3. **画面が暗くなっても録音継続**
   - MediaRecorder は `ondataavailable` でデータを蓄積
   - ロック解除後にデータが結合されて保存可能

### ⚠️ iOS の制約
| 状況 | 動作 |
|------|------|
| ホーム画面追加 + 録音中にロック | ✅ 録音継続 |
| Safariブラウザ内でロック | ❌ 一時停止される可能性あり |
| バックグラウンドで別アプリ使用 | ⚠️ iOS判断による（音楽アプリが優先されることあり） |
| スリープ時間が長い | ⚠️ iOSがプロセスを終了する可能性あり |

### 💡 ベストプラクティス
- 必ずホーム画面に追加してから使用
- 長時間録音は1時間ごとに保存ボタンを押す
- 充電しながら録音するとOSによる終了を防ぎやすい

---

## 技術仕様
- **録音形式**: WebM/Opus（iOS: AAC/M4A）
- **データ保存**: localStorage（Base64）
- **オフライン**: Service Worker によりキャッシュ
- **最大保存**: ブラウザのlocalStorageに依存（通常5-10MB）
  → 長時間録音はすぐにダウンロードを推奨
