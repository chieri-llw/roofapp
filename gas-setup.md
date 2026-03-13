# GAS セットアップ手順（フェーズ1・マルチ写真対応版）

積算アプリ → Google スプレッドシート 自動保存の設定手順です。

---

## Step 1: Google スプレッドシートを作成

1. [Google スプレッドシート](https://sheets.google.com) を開き、「空白のスプレッドシート」を新規作成
2. ファイル名を `roofapp 案件管理` などにリネーム
3. デフォルトの「シート1」のタブ名を **`案件台帳`** に変更（タブをダブルクリックで編集）
4. **案件台帳** の1行目（ヘッダー行）に以下を入力：

| A | B | C | D | E | F | G | H | I | J | K | L | M | N | O | P | Q |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 案件番号 | 日付 | 時刻 | 依頼主名 | 電話番号 | メール | 現場住所 | 建物種別 | 階数 | 工事種別 | 担当者 | 外壁面積(m²) | 窓面積(m²) | 正味面積(m²) | 外壁箇所数 | 窓箇所数 | 備考 |

5. 新しいシートを追加し、タブ名を **`積算明細`** に変更
6. **積算明細** の1行目に以下を入力：

| A | B | C | D | E | F | G | H |
|---|---|---|---|---|---|---|---|
| 案件番号 | 日付 | 写真番号 | 外壁面積(m²) | 窓面積(m²) | 正味面積(m²) | 外壁箇所数 | 窓箇所数 |

> ※ シートに手動でヘッダーを入れなくても、GASが初回実行時に自動で作成します。

7. スプレッドシートのURLをメモしておく（後でアプリに貼り付ける）

---

## Step 2: Google Apps Script を作成

1. スプレッドシートのメニューから **「拡張機能」→「Apps Script」** を開く
2. 左側のファイルリストで `コード.gs` を選択し、中身を **全て削除** して以下を貼り付ける：

```javascript
function doPost(e) {
  try {
    var data = JSON.parse(e.postData.contents);
    var ss = SpreadsheetApp.getActiveSpreadsheet();

    // ===== 案件台帳：サマリー行を追記 =====
    var sh = ss.getSheetByName('案件台帳');
    if (!sh) sh = ss.insertSheet('案件台帳');
    sh.appendRow([
      data.projectNum     || '',
      data.date           || '',
      data.time           || '',
      data.clientName     || '',
      data.clientTel      || '',
      data.clientEmail    || '',
      data.siteAddress    || '',
      data.buildingType   || '',
      data.buildingFloors || '',
      data.workType       || '',
      data.staff          || '',
      data.totalWallArea  || 0,
      data.totalWinArea   || 0,
      data.totalNetArea   || 0,
      data.totalWallCount || 0,
      data.totalWinCount  || 0,
      data.note           || ''
    ]);

    // ===== 積算明細：写真ごとの明細行を追記 =====
    var detail = ss.getSheetByName('積算明細');
    if (!detail) {
      detail = ss.insertSheet('積算明細');
      // 初回作成時にヘッダー行を追加
      detail.appendRow([
        '案件番号', '日付', '写真番号',
        '外壁面積(m²)', '窓面積(m²)', '正味面積(m²)',
        '外壁箇所数', '窓箇所数'
      ]);
    }
    if (data.photos && Array.isArray(data.photos)) {
      data.photos.forEach(function(p) {
        detail.appendRow([
          data.projectNum || '',
          data.date       || '',
          p.num           || '',
          p.wallArea      || 0,
          p.winArea       || 0,
          p.netArea       || 0,
          p.wallCount     || 0,
          p.winCount      || 0
        ]);
      });
    }

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

3. 💾 保存（`Ctrl+S`）する

---

## Step 3: ウェブアプリとしてデプロイ

1. 右上の **「デプロイ」→「新しいデプロイ」** をクリック
   （既存のデプロイを更新する場合は「デプロイを管理」→「編集（✏️）」→「新バージョン」→「デプロイ」）
2. 歯車アイコン → **「ウェブアプリ」** を選択
3. 設定を以下にする：

| 項目 | 設定値 |
|------|--------|
| 説明 | roofapp v2（マルチ写真対応） |
| 次のユーザーとして実行 | 自分（自分のGoogleアカウント） |
| アクセスできるユーザー | 全員 |

4. **「デプロイ」** をクリック
5. Googleアカウントへのアクセス許可を求められたら **「許可」** する
6. 表示された **「ウェブアプリのURL」** をコピーしておく
   形式：`https://script.google.com/macros/s/AKfycb.../exec`

---

## Step 4: アプリにURLを設定

1. ブラウザで [https://chieri-llw.github.io/roofapp/](https://chieri-llw.github.io/roofapp/) を開く
2. 右上の ⚙️（設定）アイコンをタップ
3. 「GAS Deploy ID」欄に Step 3 でコピーしたURLの `/s/` と `/exec` の間の部分を貼り付ける
4. **「保存」** をタップ

---

## Step 5: 動作確認

1. アプリで写真を読み込み、外壁・窓を測定する
2. 複数写真を追加して「写真リスト」で確認
3. 「案件登録確認」セクションで内容を確認して「スプレッドシートに保存」ボタンをタップ
4. ✅ が表示されたら成功
5. スプレッドシートを確認：
   - **案件台帳**：合計値が1行で追加されていること
   - **積算明細**：写真ごとの明細が複数行で追加されていること

---

## トラブルシューティング

| 症状 | 原因 | 対処 |
|------|------|------|
| ❌ 送信エラー | GAS URLが未設定・間違い | ⚙️から正しいURLを設定 |
| データが入らない | シート名が違う | シート名が `案件台帳` / `積算明細` であることを確認 |
| 権限エラー | デプロイ設定の誤り | Step 3 でアクセスを「全員」に設定してから再デプロイ |
| URLが古い | コードを修正後に再デプロイが必要 | 「デプロイ」→「デプロイを管理」→「編集（✏️）」→「新バージョン」→「デプロイ」 |

---

## 完了後のシート構成

```
roofapp 案件管理
├── 案件台帳      ← 案件ごとのサマリー（合計面積など）✅
├── 積算明細      ← 写真ごとの測定明細 ✅
├── 見積書        ← （フェーズ2で追加予定）
├── 工程表        ← （フェーズ3で追加予定）
└── 請求管理      ← （フェーズ4で追加予定）
```
