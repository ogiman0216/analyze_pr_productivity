# analyze_pr_productivity

## 概要

このスクリプトは、GitHubリポジトリ内のプルリクエスト（PR）に関する生産性統計を分析するためのツールです。指定された期間（デフォルトでは前月）におけるマージ済みPRの数、変更されたファイル数、追加・削除された行数などの統計情報を収集します。

Dependabotによって作成されたPRやライブラリアップデート関連のPRは自動的に除外され、純粋な開発活動による変更のみを分析対象とします。

## 必要条件

- Node.js (v12以上推奨)
- GitHubパーソナルアクセストークン（リポジトリの読み取り権限が必要）
- 分析対象のGitHubリポジトリへのアクセス権

## インストール方法

1. リポジトリをクローンまたはダウンロードします
2. 依存関係をインストールします:

```bash
npm install
```

   または、個別にインストールする場合:

```bash
npm install @octokit/graphql axios dayjs
```

### 必要な依存関係

- `@octokit/graphql`: GitHubのGraphQL APIを使用するため
- `axios`: HTTP通信（Slack通知など）のため
- `dayjs`: 日付操作のため

## 使用方法

### 環境変数の設定

以下の環境変数を設定してください:

- `GITHUB_TOKEN`: GitHubのパーソナルアクセストークン

```bash
export GITHUB_TOKEN=your_github_token_here
```

### 設定ファイルの作成

ルートディレクトリに`date_range_config.json`ファイルを作成し、以下の内容を設定します:

```json
{
  "mode": "lastMonth",                      // "lastMonth"または"custom"
  "startDate": "2025-06-01",               // modeが"custom"の場合のみ使用
  "endDate": "2025-06-30",                 // modeが"custom"の場合のみ使用
  "githubOwner": "組織名またはユーザー名",   // 必須
  "specificRepositories": [                // 分析対象リポジトリ（必須）
    "repo1",
    "repo2"
  ],
  "baseRef": "master",                     // 対象ブランチ（省略可、デフォルト: "master"）
  "filterKeyword": null                    // 除外キーワード（省略可）
}
```

### スクリプトの実行

```bash
node analyze_pr_productivity
```

## 実行結果の例

スクリプトを実行すると、以下のような出力が表示されます：

```

=== 全期間集計結果 ===
分析期間: 2024-03-01 から 2024-03-31
全体合計: 280件のPR, 2061件のコミット, 2070ファイル変更, +114816 / -45849 行

=== 月ごとの集計サマリ ===
2024年03月: 280 PRs, 2061 commits, 2070 ファイル変更, +114816/-45849 行

分析完了

::set-output name=total_prs::280
::set-output name=total_commits::2061
::set-output name=total_files_changed::2070
::set-output name=total_lines_added::114816
::set-output name=total_lines_deleted::45849
```

### 出力の説明

- **月次集計結果**: 各月ごとのPR数、コミット数、ファイル変更数、行数変更の統計
- **全期間集計結果**: 分析期間全体の合計統計
- **月ごとの集計サマリ**: 各月の結果を1行でまとめた概要
- **GitHub Actions出力**: CI/CDパイプラインで使用可能な出力変数（`::set-output`形式）

## 設定ファイルの詳細説明

- `mode`: 分析対象期間の設定モード
  - `lastMonth`: 前月の1日から末日までを自動設定
  - `custom`: 任意の期間を指定（`startDate`と`endDate`が必要）
- `startDate`/`endDate`: カスタム期間の開始日・終了日（YYYY-MM-DD形式）
- `githubOwner`: 分析対象リポジトリのオーナー（組織名またはユーザー名）
- `specificRepositories`: 分析対象のリポジトリ名のリスト
- `baseRef`: プルリクエストのベースブランチ（デフォルト: "master"）
- `filterKeyword`: このキーワードを含むPRを除外（オプション）

## 出力情報

スクリプトは以下の統計情報を出力します:

- 分析対象期間
- 合計PR数
- 変更されたファイル数（追加/削除/修正別）
- 変更された行数（追加/削除別）

GitHubActionsで使用する場合、以下の出力が生成されます:
- `total_prs`: 合計PR数
- `total_files_changed`: 変更されたファイル数の合計
- `total_lines_added`: 追加された行数の合計
- `total_lines_deleted`: 削除された行数の合計

## 注意事項

- 大量のリポジトリやPRを処理する場合、GitHubのAPI制限に注意してください
- スクリプトには5分のタイムアウトが設定されています
- Slackへの通知機能はコメントアウトされています（必要に応じて有効化してください）

## トラブルシューティング

エラーが発生した場合は、以下を確認してください:
- GitHubトークンが正しく設定されているか
- 設定ファイルの書式が正しいか
- 指定したリポジトリにアクセス権があるか
