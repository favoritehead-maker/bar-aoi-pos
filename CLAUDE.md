# Bar Aoi POS — Claude 作業メモ

## プロジェクト概要
- **アプリ URL**: https://favoritehead-maker.github.io/bar-aoi-pos/
- **リポジトリ**: favoritehead-maker/bar-aoi-pos
- **構成**: シングルファイル HTML/CSS/JS（index.html のみ）
- **シフト募集 URL**: https://favoritehead-maker.github.io/bar-aoi-pos/shift.html

## Firebase
- **プロジェクト**: aoiseisan-default-rtdb（Realtime Database）
- **本番パス**: bar_data
- **テストパス**: bar_data_test
- **リスナー**: FBREF.on('value', ...) でリアルタイム同期
- Firestore は使用していない（セキュリティルールは閉鎖済み）

## コードの重要な変数・関数

### データ構造
- DB — アプリのメインデータオブジェクト（Firebase から同期）
- DB.staff — スタッフ一覧（文字列配列）
- DB.sales — 売上データ配列
- DB.settlements — 清算履歴配列
- DB.settings — 設定（bizName, chargePrice, minGuarantee 等）
- LS.get('bar_session') / LS.set('bar_session', ...) — 営業セッション（localStorage）

### 清算フォームのフィールド ID
- s-A: スタートレジ金
- s-B: 仕入れ
- s-C: オーナー回収
- s-D: 総売上
- s-E: チャージ
- s-F: 税金
- s-G: （シャンパン原価は champItems で管理）
- s-H: キャッシュレス合計
- s-I: 未払い（つけ）
- s-I2: つけ回収入金
- s-J: 個人料理売上

### ギャラ計算式
```
L = round((D - E - F) * 0.1)          // 場代
K = max(0, round((D - E - F - G - J) / 2 - L))  // スタッフギャラ
最低保証 = 10,000円
```

### スタッフ交代（終電精算）
- openMidshiftModal() — モーダルを開く（Firebase から最新スタッフを取得）
- renderMidshiftModal() — モーダルの描画
- executeMidshiftSettlement() — 交代処理を実行
- confirmSwapDialog() — 確認ダイアログ
- 交代記録: type: 'midshift' として DB.settlements に保存
- 交代前スタッフ（スタッフA）のギャラ: K_display フィールドに保存

### QR決済
- showQrPay(method) — PayPay / d払い のQRモーダルを表示
- method: 'paypay' または 'dpay'

### テストモード
- enterTestMode() / exitTestMode() — 切替
- テストモード ON 時: bar_data_test パスを参照
- スタッフ・設定は本番から引き継ぐ（顧客・売上・清算だけリセット）

## GitHub 編集の手順（CM6 経由）
1. https://github.com/favoritehead-maker/bar-aoi-pos/edit/main/index.html を開く
2. document.querySelector('.cm-content').cmTile.view でエディタを取得
3. view.state.doc.toString() でコード全文を取得して対象箇所を検索
4. view.dispatch(state.update({ changes: { from, to, insert } })) で修正
5. 「Commit changes...」ボタン → ダイアログで「Commit changes」をクリック

## これまでの主な修正履歴
- スタッフ交代後にバナーが更新されない → renderSessionBanner() を追加
- 交代モーダルに新スタッフが出ない → Firebase から最新スタッフを取得するよう修正
- テストモードでスタッフがリセットされる → enterTestMode() でスタッフ・設定を保持
- d払いQR画面に「PayPayでお支払いください」と出る → showQrPay() の hint テキストを分岐
- 清算履歴の表示順（終電精算が最終清算より上に出る）→ ソートに副キー追加
- 清算履歴にスタッフ2人分のギャラが表示されない → 差分計算でスタッフBのギャラを追加
- 個人料理売上が前日の値を引き継ぐ → autoFillFromPOS() の if (J > 0) 条件を撤廃し常に上書き
