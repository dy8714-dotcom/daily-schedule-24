# 毎日日程表24 — セットアップガイド

## はじめに
このガイドでは、以下の2つのクラウドサービスを設定します。
- **Google Calendar API** — Googleカレンダーとの双方向同期
- **Firebase (Firestore)** — 複数デバイス間のデータ同期

所要時間：約15〜20分（両方合わせて）
費用：無料（両サービスとも無料枠で運用可能）

---

## PART 1: Google Calendar API の設定

### ステップ1: Google Cloud Console でプロジェクトを作成

1. https://console.cloud.google.com/ にアクセス（Googleアカウントでログイン）
2. 画面上部の「プロジェクトを選択」→「新しいプロジェクト」をクリック
3. プロジェクト名に「DailySchedule24」と入力 →「作成」
4. 作成完了後、そのプロジェクトが選択されていることを確認

### ステップ2: Google Calendar API を有効化

1. 左メニュー「APIとサービス」→「ライブラリ」をクリック
2. 検索ボックスに「Google Calendar API」と入力
3. 「Google Calendar API」を選択 →「有効にする」をクリック

### ステップ3: OAuth 同意画面の設定

1. 左メニュー「APIとサービス」→「OAuth 同意画面」
2. 「外部」を選択 →「作成」
3. 必須項目を入力:
   - アプリ名: DailySchedule24
   - ユーザーサポートメール: ご自身のGmailアドレス
   - デベロッパーの連絡先: ご自身のGmailアドレス
4. 「保存して次へ」
5. 「スコープを追加」→ フィルターに「calendar」と入力
6. 以下の2つにチェック:
   - `../auth/calendar.readonly`（読み取り）
   - `../auth/calendar.events`（読み書き）
7. 「更新」→「保存して次へ」
8. テストユーザーに「+ ADD USERS」→ ご自身のGmailアドレスを追加
9. 「保存して次へ」→「ダッシュボードに戻る」

### ステップ4: OAuth クライアントIDを作成

1. 左メニュー「APIとサービス」→「認証情報」
2. 「+ 認証情報を作成」→「OAuthクライアントID」
3. アプリケーションの種類:「ウェブ アプリケーション」
4. 名前: DailySchedule24
5. **「承認済みの JavaScript 生成元」** に以下を追加:
   - ローカルで使う場合: `http://localhost`
   - GitHub Pagesで使う場合: `https://あなたのユーザー名.github.io`
   - その他のホスティング先のURL
6. 「作成」をクリック
7. **「クライアントID」が表示されます** → これをコピーして控えておく
   （例: `123456789-abcdefg.apps.googleusercontent.com`）

★ このクライアントIDを、毎日日程表24のコード内の `GOOGLE_CLIENT_ID` に貼り付けます。

---

## PART 2: Firebase (Firestore) の設定

### ステップ1: Firebase プロジェクトを作成

1. https://console.firebase.google.com/ にアクセス
2. 「プロジェクトを追加」をクリック
3. プロジェクト名:「DailySchedule24」（先ほどのGCPプロジェクトを選ぶこともできます）
4. Googleアナリティクスは「無効」でOK →「プロジェクトを作成」

### ステップ2: Firestore Database を作成

1. 左メニュー「Firestore Database」をクリック
2. 「データベースを作成」をクリック
3. ロケーション:「asia-northeast1（東京）」を選択
4. セキュリティルール:「テストモードで開始」を選択 →「作成」

※ テストモードは30日間のみ有効です。後でルールを更新する手順もご案内します。

### ステップ3: ウェブアプリを登録

1. プロジェクトの概要ページで「</>」（ウェブ）アイコンをクリック
2. アプリのニックネーム:「DailySchedule24-Web」
3. 「Firebase Hosting も設定する」はチェック不要
4. 「アプリを登録」をクリック
5. **Firebase 設定情報が表示されます** → 以下の値をすべてコピー:

```
apiKey: "AIzaSy..."
authDomain: "dailyschedule24-xxxxx.firebaseapp.com"
projectId: "dailyschedule24-xxxxx"
storageBucket: "dailyschedule24-xxxxx.appspot.com"
messagingSenderId: "1234567890"
appId: "1:1234567890:web:abcdef..."
```

★ これらの値を、毎日日程表24のコード内の `FIREBASE_CONFIG` に貼り付けます。

### ステップ4: セキュリティルールの設定（推奨）

テストモード終了後は、以下のルールを設定してください:
Firestore →「ルール」タブで以下を入力:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /schedules/{docId} {
      allow read, write: if true;
    }
  }
}
```

※ パスワード認証で保護しているため、上記の簡易ルールで問題ありません。
   より厳密にしたい場合はFirebase Authenticationを追加できます。

---

## PART 3: コードへの設定値の反映

毎日日程表24のHTMLファイルを開き、以下の2箇所を書き換えてください:

### 1. Google Calendar API のクライアントID

```javascript
const GOOGLE_CLIENT_ID = 'ここにクライアントIDを貼り付け';
```

### 2. Firebase の設定情報

```javascript
const FIREBASE_CONFIG = {
    apiKey: "ここにapiKeyを貼り付け",
    authDomain: "ここにauthDomainを貼り付け",
    projectId: "ここにprojectIdを貼り付け",
    storageBucket: "ここにstorageBucketを貼り付け",
    messagingSenderId: "ここにmessagingSenderIdを貼り付け",
    appId: "ここにappIdを貼り付け"
};
```

両方を設定したら、ファイルを保存してブラウザで開いてください。
「Googleでログイン」ボタンが機能し、Firebaseの同期ステータスが「● Cloud同期中」に変われば成功です。

---

## トラブルシューティング

**Q: 「Googleでログイン」が動かない**
→ OAuth同意画面のテストユーザーにご自身のGmailが登録されているか確認
→ 承認済みJavaScript生成元にアクセス元のURLが登録されているか確認

**Q: カレンダーの予定が表示されない**
→ Google Calendar API が有効になっているか確認
→ スコープに `calendar.readonly` が含まれているか確認

**Q: Firebase同期が「ローカル保存」のまま**
→ FIREBASE_CONFIG の値がすべて正しくコピーされているか確認
→ Firestoreデータベースが作成されているか確認

**Q: 「このアプリは確認されていません」と表示される**
→ 正常です。「詳細」→「DailySchedule24（安全でない）に移動」をクリックしてください。
   個人利用の場合はこの警告は問題ありません。
