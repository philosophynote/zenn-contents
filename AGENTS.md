# Repository Guidelines

## プロジェクト構成とモジュールの整理
- `articles/` は Zenn に公開する投稿の Markdown。ファイル名はスラッグと一致させ、記事ごとのフォルダで分ける場合もある。
- `books/` には Zenn Books 向けの章やカバー画像を収録。表紙画像やメタ情報は対応するディレクトリ内にまとめる。
- `images/` は記事で使う画像素材を入れる。スペースなしの説明的な名前（例: `articles/zenn-cli-overview.png`）で管理。
- ルートの `package.json`/`README.md`/`CLAUDE.md` は CLI 操作、投稿ルール、補助的なメモを残す場所。
- `node_modules/` は npm 管理対象なのでコミット不要。`package-lock.json` は依存固定のため保持。

## ビルド・テスト・開発コマンド
- `npm install` : `zenn-cli` を含む依存をインストール。初回と依存追加時に実行。
- `npm test` : 現在エコー処理のみ（`echo "Error: no test specified"`）。実装が増えたら `package.json` を更新。
- `npx zenn preview` : Markdown をローカルサーバーで表示。公開前にフォントや画像リンクを確認。
- `npx zenn new:article "タイトル"` / `npx zenn new:book "タイトル"` : 必要な front matter を持つテンプレートを自動生成。

## コーディングスタイルと命名規則
- Markdown は箇条書きでネストが必要な場合は半角2スペースでインデント。横幅は約100文字以内を目安。
- ファイル名は `kebab-case`（例: `zenn-cli-tips.md`）でスラグとして使える形に。
- Front matter のキーは小文字（`title`/`published`/`topics` など）、タグはカンマ区切り。
- 日本語と英語を投稿ごとに統一し、末尾の不要な空白や全角スペースを使わない。

## テスト指針
- 自動テストは未整備。追加する際は Jest 等で `npm test` から起動できるようにする。
- テストファイルは対象記事と同ディレクトリか近傍（例: `articles/sample.test.js`）に置き、対象名を含める。
- カバレッジはまだ必須ではないが、命名と配置を整備することで後続メンテナの移行を容易に。

## コミット/プルリクエスト方針
- コミットメッセージは Conventional Commits 式を推奨（`feat: add article draft`、`fix: correct front matter` など）。
- PR には「何を」「なぜ」行ったか一行要約と、主要な変更点（例: `Zenn CLI preview で確認済み`）を記載。
- 該当する issue があればリンクし、ビジュアルに変化があれば `npx zenn preview` のスクリーンショットを添付。

## 公開フロー
- 下書きは `published: false` の状態で `articles/` や `books/` に保存し、`npx zenn preview` で整合性を確認。
- 公開するときは `npx zenn publish article` または `npx zenn publish book` を実行し、生成された slug がファイル名と一致するかチェック。
- PR ではメタデータの変更（カテゴリ/タグ更新など）と一緒にレビュー対象を明示し、必要なら共同執筆者への言及を含める。
