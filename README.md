# 指示理解・実行力診断テスト / Instruction Comprehension & Execution Check

特定技能介護士向けの診断テストです。試験に合格して来日したのに、現場で指示を実行できない場合に、どこでつまずいているかを4つの側面から確認します。

A screening test for Specified Skilled Workers (Kaigo). It checks where a worker who passed the exam but struggles to carry out instructions on the job is having difficulty, across four areas.

Yoshida EdTech Works / EdTech Gabay — 共通開発プロンプト準拠（試験系テーマ）

---

## 目的と使う人 / Purpose and users

送り出し機関、介護士本人、受け入れ施設の三者が、同じテストと同じ結果画面を使います。開始時に「実施する人」を選ぶと、結果画面の「役割別の次の一歩」で該当する役割が先頭に開きます。

Sending organizations, caregivers and receiving facilities all use the same test and result screen. The role chosen at the start decides which "Next steps by role" section opens first.

**診断ではなく目安です。採用・評価・解雇の判断には使わないでください。**
**This is a screening guide, not a diagnosis. Do not use it for hiring, evaluation or dismissal decisions.**

## 構成 / Structure

全20問（4部 × 5問）、所要約15分。

| 部 / Part | 測るもの / What it checks | 形式 / Format |
| --- | --- | --- |
| 第1部 | 現場語彙・聞き取り（擬音語、指示語、専門用語） | 4択 |
| 第2部 | 連続指示の保持（3〜4動作、うち1問は環境ノイズ付き） | 並べ替え |
| 第3部 | 時間・条件・優先順位（「〜前に」「もし〜たら」「〜は後でいいから」） | 4択 |
| 第4部 | 別作業（10秒の暗算）を挟んだあとの記憶 | 割り込み＋4択 |

音声は端末の日本語読み上げ（Web Speech API）で再生します。

## モード / Modes

| モード | 内容 |
| --- | --- |
| 本番 / Test | 音声は1問につき1回のみ |
| 練習 / Practice | 何回でも聞ける |
| 文字表示 / Text | 音声の代わりに文字で表示。聞き取りが原因かの切り分け用 |

音声の速さは「現場の速さ 1.15x」「標準 0.95x」「ゆっくり 0.8x」から選べます。モードと速さは結果に記録されます。テスト中には変更できません。

**原因の切り分け方 / Isolating the cause:** 音声モードで要観察・要支援の項目があれば、同じ人に文字表示モードで再実施します。文字だと正解が増えるなら聞き取りが主因、増えないなら記憶・論理・語彙が主因の可能性があります。0.8xとの比較も有効です。

## 判定 / Scoring

部ごとの正答率で判定します。

| 正答率 | 判定 |
| --- | --- |
| 80%以上（5問中4〜5問） | 良好 / Good |
| 60%以上（5問中3問） | 要観察 / Watch |
| 60%未満（5問中2問以下） | 要支援 / Needs support |

第2部は部分点（正しい位置に置けた数）も記録します。判定は完全一致のみで行います。

## 結果の書き出し / Export

結果画面から次の3つを出せます。

- **CSV**（Excel対応のUTF-8 BOM付き）: 実施情報、部ごとの結果、全問の詳細
- **JSON**: 他システムへの取り込み用
- **印刷 / PDF**: ブラウザの印刷機能を使用

ファイル名は `shiji-test_YYYYMMDD_ID.csv` の形式です。

### JSONの項目 / JSON fields

```json
{
  "version": 1,
  "id": "受験者ID",
  "role": "sending | caregiver | facility",
  "date": "ISO 8601",
  "mode": "strict | practice | text",
  "speed": 1.15,
  "parts": [{ "part": 1, "correct": 4, "items": 5, "level": "Good" }],
  "items": [{
    "no": 1, "part": 1, "correct": true, "stepsInPlace": null,
    "answer": "回答テキスト", "plays": 1, "ms": 3200,
    "sums": null
  }]
}
```

- `stepsInPlace`: 第2部のみ。正しい位置に置けた数
- `ms`: 音声再生後から回答までの時間（ミリ秒）
- `sums`: 第4部のみ。割り込み計算の `{ok, n}`（正解数／回答数）

## 公開方法 / Deploy

ビルド不要の単一HTMLです。

1. このリポジトリに `index.html` を置いてコミットします。
2. Vercelでリポジトリをインポートします（Framework Preset: Other、Build Command は空欄）。
3. 公開後、スマホで音声再生と書き出しを確認します。

ローカルで試す場合は `index.html` をブラウザで開くだけです。

## 問題の編集 / Editing questions

問題は `index.html` 内の `Q` 配列にあります。1問は次の形のオブジェクトです。

| キー | 意味 |
| --- | --- |
| `s` | 部の番号（0〜3） |
| `ty` | `'c'` 4択、`'o'` 並べ替え、`'d'` 割り込み＋4択 |
| `c` | 場面 `[日本語, English]` |
| `h` | 問い（任意） |
| `a` | 音声の台本 `[日本語, English]`。読み上げは日本語のみ |
| `o` | 選択肢 `[アイコン, 日本語, English]` を4つ（`c`・`d`） |
| `k` | 正解の番号（`o` の0始まりの位置） |
| `st` | 並べ替えのステップ `[日本語, English]` を正しい順に（`o`） |
| `dq` | 割り込み後の質問 `[日本語, English]`（`d`） |
| `nz` | `1` で環境ノイズ付き |
| `x` | 解説 `[日本語, English]` |

部ごとの問題数を変えても、判定は正答率で計算されるため動作します。各部の問題数は揃えることをおすすめします。

## デザインとセキュリティ / Design and security

- 共通開発プロンプトのトークン（クリーム・墨色、試験系アクセント `#2E8B76` / `#1F6355`）、Shippori Mincho と Zen Kaku Gothic New を使用
- 提出時の遷移音は437Hz
- タップ領域44px、キーボードフォーカス表示
- 入力は絞り込みとHTMLエスケープ、CSV出力時は数式インジェクション対策（`=` `+` `-` `@` 始まりを無害化）
- CSP（メタタグ）で外部通信を制限。外部読み込みはGoogleフォントのみ
- 回答は端末内だけで処理し、サーバー送信・保存は一切しません

## 制限事項 / Limitations

- 日本語音声がない端末では、文字表示モードを使うか別の端末で実施してください
- 合成音声と環境ノイズは現場の再現として近似であり、実際の声・施設音とは異なります
- 各部5問のため、個人の診断としては粗い目安です
- 37.5度などの基準は施設ごとに異なります。解説は施設の決まりに従う前提です
- PWAとしてのインストール（manifest / Service Worker）には未対応です
- Googleフォントは読み込めない場合、端末標準のフォントで表示されます

## 免責 / Disclaimer

本ツールは指導計画の参考にするための目安であり、医学的・心理学的な診断ではありません。結果は本人の同意のもとで共有し、不利益な扱いに使わないでください。

This tool is a guide for planning support, not a medical or psychological diagnosis. Share results only with the worker's consent and never use them to the worker's disadvantage. shiji-rikai-test
