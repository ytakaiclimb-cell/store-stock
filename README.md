# 店舗の在庫ボード（Web版）

ディーボルダリングプラス 船橋・八千代の在庫管理。ブラウザで開くだけで使えます。

- `index.html` … アプリ本体（1ファイル。これだけで全部動きます）
- `manifest.json` / `icon.svg` / `icon-maskable.svg` … ホーム画面に追加するため

## 使うとき

公開URLを開き、**合言葉**を入れるだけ。一度入れれば、その端末では次から聞かれません。

スマホは、ブラウザのメニューから「ホーム画面に追加」を押すとアプリのように開けます。

## データの置き場所

Supabase プロジェクト `store-ops-board`（Y Assistant と相乗り）の `stock_docs` テーブル1つ。

| collection | 中身 |
| --- | --- |
| `items` | 商品マスタ（船橋335 / 八千代651） |
| `counts` | 棚卸・POS取り込みの在庫数 |
| `moves` | 入出庫の記録 |
| `meta` | `settings`（店舗・カテゴリ） |
| `chain` | 全店の在庫スナップショット（500商品ずつ7行） |
| `posref` | POSの正式な商品名と商品グループ（900件ずつ4行） |

在庫数はどこにも持たず、**最新の `counts` ＋ それ以降の `moves`** から毎回計算します。

## 合言葉のしくみ

`index.html` に入っているのは Supabase の URL と **anon キー**だけ。これは公開前提のキーで、
これだけでは1行も読めません。

読み書きできるのは、`x-stock-key` ヘッダに合言葉が入っているときだけです（RLS の
`public.stock_key_ok()` で判定）。**合言葉はソースに入っていません。**
なのでこのリポジトリは Public で問題ありません。

合言葉を変えるときは Supabase 側で:

```sql
create or replace function public.stock_key_ok() returns boolean
language sql stable as $$
  select coalesce(current_setting('request.headers', true)::json ->> 'x-stock-key', '') = '新しい合言葉'
$$;
```

変えたら、各端末で設定タブの「この端末の合言葉を消す」を押して入れ直してもらいます。

## 直したいとき

`index.html` を書き換えて push するだけです。ビルドはありません。

## 元になったもの

Claude の Artifact 版（`C:\Users\2023daikipc8y\claude\store_stock\index.html`）から、
データの保存先だけ差し替えたものです。画面と機能は同じです。
