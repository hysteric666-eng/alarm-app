# 目覚まし時計アプリ — Claude Code 指示書

## プロジェクト概要

`index.html` に完成済みのWebプレビュー（単一HTMLファイル）がある。
これをそのまま **PWA（Progressive Web App）** として動作させ、
モバイルブラウザ（iOS Safari / Android Chrome）から
「ホーム画面に追加」で ネイティブアプリのように使えるようにする。

---

## 技術スタック

- **フロントエンド**: 純粋な HTML / CSS / Vanilla JavaScript（フレームワーク不使用）
- **PWA**: Service Worker + Web App Manifest
- **通知**: Web Notifications API + Web Audio API（アラーム音）
- **振動**: Vibration API
- **ストレージ**: localStorage（データ永続化）
- **言語**: 日本語UI

---

## やること（実装タスク一覧）

### 1. PWA化
- `manifest.json` を作成（アプリ名・アイコン・テーマカラー・display:standalone）
- `sw.js`（Service Worker）を作成してオフライン対応
- `index.html` の `<head>` に manifest リンク・meta タグを追加

### 2. データ永続化
- 現在はページリロードでデータが消える
- `alarms`・`timers`・`groups`・`groupOrder`・`collapsed`・`use12h` を `localStorage` に保存
- ページ読み込み時に `localStorage` から復元
- データ変更のたびに自動保存（`render()` 呼び出し後に保存）

### 3. アラーム通知（実機動作）
- `setInterval` で毎分現在時刻をチェック
- 有効なアラームの時刻に一致したら：
  - `Notification API` でプッシュ通知
  - `Web Audio API` でアラーム音を鳴らす（oscillator で簡易音）
  - `Vibration API` で振動
  - `skipHoliday` が true の場合は日本の祝日判定をして鳴動スキップ
- スヌーズ：通知後に「スヌーズ」ボタンで指定分後に再通知

### 4. タイマー通知
- タイマーのカウントダウンが0になったら：
  - `Notification API` でプッシュ通知
  - `Web Audio API` でアラーム音を鳴らす
  - `Vibration API` で振動

### 5. アイコン生成
- `icons/` フォルダに PWA用アイコンを作成
  - `icon-192.png`（192×192）
  - `icon-512.png`（512×512）
  - デザイン：紫背景に白い目覚まし時計アイコン（SVGから変換 or Canvas生成）

### 6. iOS対応
- `<meta name="apple-mobile-web-app-capable" content="yes">`
- `<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">`
- `<link rel="apple-touch-icon" href="icons/icon-192.png">`

---

## ファイル構成（完成形）

```
alarm-app/
├── index.html          # メインアプリ（既存・修正あり）
├── manifest.json       # PWAマニフェスト
├── sw.js               # Service Worker
├── icons/
│   ├── icon-192.png
│   └── icon-512.png
└── CLAUDE.md           # この指示書
```

---

## デザイン・UIの注意点

- **既存のUI/UXは絶対に変えない**
- CSSのカラーテーマ（`--bg:#0f0f14` など）はそのまま維持
- レイアウト・フォント・アニメーションはすべて既存通り
- `index.html` の修正は最小限（PWA対応とデータ永続化のみ）

---

## 実装の優先順位

1. データ永続化（localStorage）← 最重要・すぐ体感できる
2. PWA化（manifest + service worker）
3. アラーム通知（Web Notifications + Audio）
4. タイマー通知
5. アイコン生成

---

## 動作確認方法

```bash
# ローカルサーバーで起動（Service Workerはhttpsまたはlocalhostが必要）
npx serve .
# または
python3 -m http.server 8080
```

ブラウザで `http://localhost:8080` を開き、
Chrome DevTools > Application > Service Workers で登録確認。

---

## 重要な制約

- `index.html` 内のJSグローバル変数（`alarms`, `timers`, `groups` など）の構造は変えない
- 既存の `render()` 関数・`getGroupOrder()` などのロジックは変更しない
- 通知許可は初回起動時に `Notification.requestPermission()` で取得
- Service Worker は `fetch` イベントをキャッシュファーストで処理
