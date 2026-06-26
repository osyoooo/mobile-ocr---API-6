# 書籍申込 合計計算

スマホで書籍申込表を撮影し、画像全体から **「税務研究会」** の表を探して、右端の **申込冊数** だけを OpenAI API で読み取る Next.js アプリです。

読み取り後、No1〜No22の冊数を確認・修正し、アプリ側の固定価格マスタで **合計冊数** と **合計金額** を計算します。さらに **助成金・優待券を差し引いた支払額** も表示します。

## できること

- iOS / Android のブラウザでスマホ標準カメラを起動
- 広めに撮影した画像全体を OpenAI API へ送信
- 画像内の「税務研究会」の表だけを対象にする
- No1〜No22 の右端「申込冊数」列だけを読み取る
- 読み取り結果を手修正できる
- 合計冊数と合計金額を即時計算
- 助成金 -5,000円を固定適用
- 優待券を -4,000円 / -2,000円 / -6,000円 / 0円 から選択
- 差引支払金額と優待券消化額を表示
- Vercel にデプロイ可能

## 重要な設計

OpenAI APIには、書籍名や価格の計算を任せません。

- OpenAI API: 画像から No1〜No22 の申込冊数だけを読む
- アプリ: 固定価格マスタで合計冊数・合計金額を計算し、助成金・優待券を差し引く

これにより、価格の誤読や計算ミスを避けます。

## セットアップ

```bash
npm install
cp .env.example .env.local
```

`.env.local` に OpenAI API キーを設定します。

```bash
OPENAI_API_KEY=sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
OPENAI_MODEL=gpt-4.1-mini
OPENAI_IMAGE_DETAIL=high
```

起動します。

```bash
npm run dev
```

ブラウザで `http://localhost:3000` を開きます。

## Vercel 環境変数

Vercel にデプロイする場合は、Project Settings > Environment Variables に以下を設定してください。

```text
OPENAI_API_KEY
OPENAI_MODEL
OPENAI_IMAGE_DETAIL
```

最低限必要なのは `OPENAI_API_KEY` です。

## 使い方

1. 「カメラで撮影」を押す
2. 「税務研究会」の表が入るように広めに撮影する
3. 撮影画像を確認する
4. 「AIで自動読取」を押す
5. No1〜No22の冊数を確認・修正する
6. 合計冊数・合計金額を確認する
7. 優待券を選択し、差引支払金額と優待券消化額を確認する

## 撮影のコツ

- 「税務研究会」のタイトルが写るようにする
- No1〜No22が切れないようにする
- 右端の「申込冊数」列が切れないようにする
- 紙が斜めすぎる、暗すぎる、影が強い場合は撮り直す
- 隣の表が写ってもよいが、対象表が大きく写っているほうが安定する


## 助成金・優待券の計算仕様

読み取り後の合計金額に対して、以下の順に計算します。

```ts
const amountAfterSubsidy = Math.max(0, totalAmount - 5000);
const voucherFaceValue = Math.abs(voucherAmount);
const voucherUsed = Math.min(amountAfterSubsidy, voucherFaceValue);
const payableAmount = Math.max(0, amountAfterSubsidy - voucherUsed);
```

- 助成金は常に `-5,000円` 固定です。
- 優待券のデフォルトは `-4,000円` です。
- 優待券は `-4,000円 / -2,000円 / -6,000円 / 0円` から選択できます。
- 差引支払金額がマイナスになる場合は `0円` として表示します。
- 優待券消化額は、助成金適用後の残額を上限として表示します。

## 価格マスタ

価格は `lib/books.ts` に固定で入れています。

| No | 価格 |
|---:|---:|
| 1 | 10,890円 |
| 2 | 9,702円 |
| 3 | 1,881円 |
| 4 | 6,237円 |
| 5 | 2,277円 |
| 6 | 8,613円 |
| 7 | 2,970円 |
| 8 | 2,772円 |
| 9 | 3,564円 |
| 10 | 2,178円 |
| 11 | 3,465円 |
| 12 | 1,980円 |
| 13 | 2,178円 |
| 14 | 3,465円 |
| 15 | 4,950円 |
| 16 | 4,356円 |
| 17 | 3,465円 |
| 18 | 2,970円 |
| 19 | 3,267円 |
| 20 | 1,881円 |
| 21 | 4,455円 |
| 22 | 5,445円 |

## Ver3 差引計算

AI読取後の確認画面では、合計冊数・合計金額に加えて以下を表示します。

- 助成金: -5,000円固定
- 優待券: -4,000 / -2,000 / -6,000 / 0 から選択。初期値は -4,000
- 差引支払金額: 助成金と実際に使えた優待券を差し引いた金額。マイナスになる場合は0円
- 優待券消化額: 助成金適用後の残額に対して、実際に使えた優待券の金額

合計欄と差引計算欄は固定表示ではなく、No22の確認行の後に通常表示します。

## API

`app/api/ai-ocr-full-table/route.ts` が OpenAI API を呼び出します。

ブラウザ側に `OPENAI_API_KEY` は置きません。必ずサーバー側環境変数で管理してください。

## エラー対応

### You exceeded your current quota

OpenAI Platform 側の API クレジット、月間上限、Project budget を確認してください。ChatGPT の有料契約と API クレジットは別です。

### OPENAI_API_KEY が設定されていません

Vercel の Environment Variables に `OPENAI_API_KEY` を設定し、再デプロイしてください。

### 対象表の検出に不安があります

画像内で「税務研究会」表を特定できなかった、または表が切れている可能性があります。冊数を確認・修正してください。

## 新しいリポジトリへアップロード

```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/<your-user>/<your-repo>.git
git push -u origin main
```

その後、Vercel でこのリポジトリをImportしてください。


## Vercelデプロイ時の注意

このリポジトリでは `package-lock.json` をコミットしない構成にしています。生成環境によっては lockfile の `resolved` URL が社内・一時的なnpmミラーを指してしまい、Vercel の `npm install` が失敗することがあるためです。

`vercel.json` で Vercel の install command を `npm install --package-lock=false --no-audit --no-fund --registry=https://registry.npmjs.org/` に固定しています。

既存リポジトリを修正する場合は、以下を実行してください。

```bash
git rm package-lock.json
git add package.json .npmrc vercel.json README.md
git commit -m "Fix Vercel npm install"
git push
```
