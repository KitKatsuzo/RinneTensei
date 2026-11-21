# 輪廻転生

閻魔さまに1週間・1年・これまでの人生の行いを裁いてもらい、来世を決めるゲーム。
懺悔と誓いで、来世は書き換えられる。

## しくみ

```
ブラウザ（GitHub Pages の index.html）
   │  「何を裁いてほしいか」だけを送る
   ▼
中継サーバー（Cloudflare Workers の worker/worker.js）
   │  APIキーはここにだけ置く。AIへの指示文もここで組み立てる
   ▼
Gemini API
```

- APIキーはページに一切書かない。
- 中継サーバーは「裁き」と「誓い」の2種類しか受け付けないので、他人に汎用のAI窓口として使われない。
- 中継サーバーに届かないときは、ページがキーワードを数える簡易判定に切り替わる。

## 公開の手順

### 1. Gemini の APIキーを作る
1. Google AI Studio（https://aistudio.google.com）を開き、APIキーを作る。
2. 無料枠の範囲と料金は AI Studio で確認しておく。

### 2. 中継サーバーを Cloudflare Workers に置く
1. Cloudflare のアカウントを作る（無料）。
2. ターミナルで:
   ```bash
   cd worker
   npm install -g wrangler        # 初回だけ
   wrangler login
   wrangler secret put GEMINI_API_KEY   # ここで APIキーを貼る（ファイルには残らない）
   wrangler deploy
   ```
3. 表示された URL（`https://rinne-tensei-api.○○.workers.dev`）を控える。
4. `wrangler.toml` の `ALLOWED_ORIGIN` を、自分の GitHub Pages の URL にしておく
   （例：`https://kitkatsuzo.github.io`。末尾の `/` は付けない）。変えたら `wrangler deploy` し直す。

> 回数制限（1つのIPから60秒に20回まで）の設定が `wrangler.toml` に入っています。
> デプロイでエラーになったら、Cloudflare の「Rate Limiting」ドキュメントで最新の書き方を確認するか、
> `[[ratelimits]]` の4行を消してもう一度デプロイしてください（消すと回数制限なしで動きます）。

### 3. ページを GitHub Pages で公開する
1. `index.html` の上のほうにある `API_URL` を、手順2で控えた URL に書き換える。
2. GitHub に新しいリポジトリ（例：`rinne-tensei`）を作り、`index.html` と `README.md` を置く。
   `worker/` フォルダも置いてよい（APIキーは入っていない）。
3. リポジトリの Settings → Pages で、Branch を `main`、フォルダを `/ (root)` にして保存。
4. 数分後に `https://ユーザー名.github.io/rinne-tensei/` で開けるようになる。

## 動作確認
- 閻魔帳に書いて「閻魔さまに裁いてもらう」を押し、判決の下に
  「AIの閻魔さまに会えなかったので…」と出なければ、AI がつながっている。
- 出る場合は、`API_URL` の書き間違い、`ALLOWED_ORIGIN` の不一致、APIキーの設定漏れを確認する。
  Cloudflare のダッシュボードで Worker のログ（`wrangler tail`）を見ると原因が分かる。

## モデルを変えるとき
`wrangler.toml` の `GEMINI_MODEL` を書き換えて `wrangler deploy`。
