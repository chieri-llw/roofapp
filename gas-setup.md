# GAS セットアップ手順（フェーズ1）

積算アプリ → Google スプレッドシート 自動保存の設定手順です。

---

## Step 1: Google スプレッドシートを作成

1. [Google スプレッドシート](https://sheets.google.com) を開き、**「空白のスプレッドシート」** を新規作成
2. ファイル名を `roofapp 案件管理` などにリネーム
3. デフォルトの「シート1」のタブ名を **`案件台帳`** に変更（タブをダブルクリックで編集）
4. 1行目（ヘッダー行）に以下を入力：

| A | B | C | D | E | F | G | H | I | J | K | L | M | N | O | P | Q |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 案件番号 | 日付 | 時刻 | 依頼主名 | 電話番号 | メール | 現場住所 | 建物種別 | 階数 | 工事種別 | 担当者 | 外壁面積(m²) | 窓面積(m²) | 正味面積(m²) | 外壁箇所数 | 窓箇所数 | 備考 |

5. スプレッドシートのURLをメモしておく（後でアプリに貼り付ける）

---

## Step 2: Google Apps Script を作成

1. スプレッドシートのメニューから **「拡張機能」→「Apps Script」** を開く
2. 左側のファイルリストで `コード.gs` を選択し、中身を **全て削除** して以下を貼り付ける：

```javascript
function doPost(e) {
  try {
    var data = JSON.parse(e.postData.contents);
    var ss   = SpreadsheetApp.getActiveSpreadsheet();
    var sh   = ss.getSheetByName('案件台帳');
    if (!sh) sh = ss.insertSheet('案件台帳');

    sh.appendRow([
      data.projectNum    || '',
      data.date          || '',
      data.time          || '',
      data.clientName    || '',
      data.clientTel     || '',
      data.clientEmail   || '',
      data.siteAddress   || '',
      data.buildingType  || '',
      data.buildingFloors|| '',
      data.workType      || '',
      data.staff         || '',
      data.wallArea      || '',
      data.winArea       || '',
      data.netArea       || '',
      data.wallCount     || '',
      data.winCount      || '',
      data.note          || ''
    ]);

    return ContentService
      .createTextOutput(JSON.stringify({ result: 'ok' }))
      .setMimeType(ContentService.MimeType.JSON);

  } catch(err) {
    return ContentService
      .createTextOutput(JSON.stringify({ result: 'error', message: err.message }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}
```

3. 💾 **保存（Ctrl+S）** する

---

## Step 3: ウェブアプリとしてデプロイ

1. 右上の **「デプロイ」→「新しいデプロイ」** をクリック
2. 歯車アイコン → **「ウェブアプリ」** を選択
3. 設定を以下にする：

   | 項目 | 設定値 |
   |------|--------|
   | 説明 | roofapp v1 |
   | 次のユーザーとして実行 | **自分（自分のGoogleアカウント）** |
   | アクセスできるユーザー | **全員** |

4. **「デプロイ」** をクリック
5. Googleアカウントへのアクセス許可を求められたら **「許可」** する
6. 表示された **「ウェブアプリのURL」** をコピーしておく

   形式：`https://script.google.com/macros/s/AKfycb.../exec`

---

## Step 4: アプリにURLを設定

1. ブラウザで [https://chieri-llw.github.io/roofapp/](https://chieri-llw.github.io/roofapp/) を開く
2. 右上の **⚙️（設定）** アイコンをタップ
3. **「案件番号プレフィックス」** に会社略称を入力（例：`NK`）
4. **「GAS URL」** 欄に Step 3 でコピーしたURLを貼り付ける
5. （任意）**「スプレッドシートURL」** に Step 1 でメモしたURLを貼り付ける
6. **「保存」** をタップ

---

## Step 5: 動作確認

1. アプリで写真を読み込み、外壁・窓を測定する
2. 「案件台帳＋保存」セクションで案件情報を入力
3. **「スプレッドシートに保存」** ボタンをタップ
4. ✅ が表示されたら成功
5. スプレッドシートの「案件台帳」シートにデータが追加されていることを確認

---

## トラブルシューティング

| 症状 | 原因 | 対処 |
|------|------|------|
| ❌ 送信エラー | GAS URLが未設定・間違い | ⚙️から正しいURLを設定 |
| データが入らない | シート名が違う | シート名が `案件台帳` であることを確認 |
| 権限エラー | デプロイ設定の誤り | Step 3 でアクセスを「全員」に設定してから再デプロイ |
| URLが古い | コードを修正後に再デプロイが必要 | 「デプロイ」→「デプロイを管理」→「編集（✏️）」→「新バージョン」→「デプロイ」 |

---

## 完了後のシート構成

```
roofapp 案件管理
├── 案件台帳     ← アプリが自動書き込み（フェーズ1 ✅）
├── 積算記録     ← （フェーズ2で追加予定）
├── 見積書       ← （フェーズ2で追加予定）
├── 工程表       ← （フェーズ3で追加予定）
└── 請求管理     ← （フェーズ4で追加予定）
```
