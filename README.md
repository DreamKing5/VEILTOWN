# VEIL TOWN v2 — Server Authority Beta

現行のブラウザ内ホスト方式を廃止し、Supabase PostgreSQLをゲームの唯一の状態管理者にした再設計版です。

## 重要な改善

- `players(room_id,user_id)`の一意制約により、参加要求を何度送っても1人
- Supabase Anonymous AuthのユーザーIDを使うため、再読み込みしても同じプレイヤー
- 投票と夜行動はフェーズごとに一意で、連打しても上書きのみ
- フェーズ進行はPostgreSQLの行ロック＋advisory lockで二重実行を防止
- 役職情報はRLSにより本人のみ取得可能
- ゲーム状態はDBへ保存され、作成者のタブが閉じても消えない
- Realtimeが切れても3秒ポーリングで復帰

## 公開前セットアップ

1. Supabaseで無料プロジェクトを作成する。
2. `Authentication → Providers → Anonymous Sign-Ins`を有効化する。
3. `SQL Editor`を開き、`schema.sql`の全文を一度実行する。
4. `Project Settings → API`からProject URLとanon / publishable keyを取得する。
5. `config.js`の2か所へ貼り付ける。
6. `index.html`、`styles.css`、`app.js`、`config.js`を同じフォルダでGitHub Pagesへ公開する。
7. 公開URLを開き、「認証済み」と表示されることを確認する。

`service_role` keyは絶対にブラウザへ入れないでください。

## 配布ファイル

- `index.html` — 画面
- `styles.css` — レスポンシブUI
- `app.js` — 認証、RPC、同期、描画
- `config.js` — 公開用接続設定
- `schema.sql` — テーブル、RLS、RPC、Realtime設定
- `tests/invariants.sql` — 重複防止などの確認項目

## 招待と復帰

招待URLは `#ROOMCODE.INVITE_SECRET` 形式です。Supabaseキーは招待URLへ含みません。同じブラウザで再読み込みした場合、Anonymous Authのセッションとローカル保存したルーム情報を使って同じプレイヤーへ復帰します。

## 現在のゲームエンジン範囲

町長選挙、夜、夜明け、議論、追放投票、勝敗判定をサーバー側で進行します。医師、警官、生存者、連続殺人犯、怪盗、反逆陣営襲撃、調査員、追跡者の主要処理を収録しています。その他の高度な役職効果はデータ上配役されますが、完全な相互作用は今後のサーバーエンジン拡張対象です。

この版は安定基盤を優先したBetaです。本家相当の全役職挙動を名乗る完成版ではありません。

## 公開判定条件

- 同じユーザーの100回参加でプレイヤー1行
- 20回再読み込みしても同じプレイヤー
- 12端末同時参加
- 投票・夜行動の連打で各1行
- フェーズ進行100回同時要求で1遷移
- 他プレイヤーの`player_secrets`を取得不可
- PC＋iPhone／Android／iPadの別回線試験
- 1時間耐久試験

上記実機試験が完了するまでは、旧版を置き換えず別URLで公開してください。
