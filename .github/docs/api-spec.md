# CI ワークフロー API 仕様書

ファイル: `.github/workflows/ci.yml`  
コミット: `7659931edb4498e9339e9fd4f2e789c0f97ac79a`  
作成日（解析日時）: 2026-01-08

## 概要
このワークフローは Java/Gradle プロジェクトの継続的インテグレーション（CI）用です。リポジトリのチェックアウト、JDK 17 のセットアップ、Gradle ラッパーに実行権限を付与してから `./gradlew build` を実行するシンプルなビルドジョブを含みます。

---

## トリガー（Triggers）
- push
  - branches: `["**"]`  
    - 説明: すべてのブランチへの push イベントでトリガーされます（glob パターン）。
- pull_request
  - branches: `["main"]`  
    - 説明: `main` ブランチに対するプルリクエストが作成・更新されたときにトリガーされます。

注: `workflow_dispatch`（手動トリガー）や `schedule`（定期実行）は定義されていません。手動実行を必要とする場合は `workflow_dispatch` を追加してください。

---

## Jobs
このワークフローは単一のジョブ `build` を定義します。

- job id: `build`
  - runs-on: `ubuntu-latest`
  - 説明: Ubuntu 最新イメージ上でジョブを実行します。

ジョブの振る舞い:
- ジョブは上のトリガーで起動され、ステップを上から順に実行します。
- いずれかのステップが失敗（非ゼロ終了コード）するとジョブは失敗として停止します。

---

## Inputs / Parameters
このワークフロー定義には外部入力（`inputs`）はありません。
- まとめ: `workflow_dispatch` の `inputs` は未定義 → 手動トリガーおよびパラメータ化は不可。

---

## Steps（各ステップ詳細）
ワークフローの `build` ジョブのステップを順に列挙します。

1. Checkout repository
   - type: uses
   - action: `actions/checkout@v4`
   - 説明: リポジトリのコンテンツをチェックアウト（作業ディレクトリへ取得）。

2. Set up JDK 17
   - type: uses
   - action: `actions/setup-java@v4`
   - with:
     - `distribution`: `temurin`
     - `java-version`: `17`
   - 説明: Temurin ディストリビューションの JDK 17 をセットアップします（JAVA_HOME の設定を含む）。

3. Grant execute permission for gradlew
   - type: run
   - run: `chmod +x gradlew`
   - shell: デフォルトのシェル（Ubuntu runner の bash）
   - 説明: Gradle wrapper（`gradlew`）に実行権限を付与します。Windows 環境では不要ですが、Linux runner では必須。

4. Build with Gradle
   - type: run
   - run: `./gradlew build`
   - 説明: Gradle ビルドを実行します。ビルドとテストがここで実行されます（プロジェクトの `build.gradle`/`settings.gradle` に従う）。
   - 成功条件: `./gradlew build` が 0 を返すこと
   - 失敗条件: テスト失敗やコンパイルエラー等で非ゼロ終了コードが返ること → ジョブは失敗

---

## 環境変数 / シークレット
- ワークフロー内には環境変数（`env:`）やシークレット参照（`secrets.`）の定義はありません。
- 必要に応じて `with:`、`env:`、あるいは `secrets` の参照を追加してください（例: publishing 認証情報など）。

---

## 出力 / アーティファクト
- 明示的なワークフロー出力（`outputs:`）は定義されていません。
- ビルド成果物はランナー内に作成されますが、現状では保存（upload-artifact）やキャッシュ（actions/cache）処理は行われません。
- 推奨: 成果物を後続ジョブやリリースに使う場合は `actions/upload-artifact` を追加してください。

---

## 権限（Permissions）
- 明示的な `permissions:` ブロックは指定されていません。デフォルトのリポジトリ権限が適用されます。
- 必要に応じて最小限の権限を明示的に指定してください（例: contents: read/write など）。

---

## 期待されるログ / 実行フロー
1. checkout が成功 → リポジトリがワーカーへ取得される
2. setup-java が成功 → `JAVA_HOME` が設定され `java -version` は JDK 17 を示す
3. chmod が成功 → `gradlew` に実行権限が付与される
4. `./gradlew build` 実行 → Gradle のビルドログ、テストの結果が出力される
5. いずれかのステップで失敗した場合、ジョブが停止し GitHub Actions 上で失敗として表示される

---

## 拡張案（運用上の推奨）
- キャッシュ: `actions/cache` で Gradle キャッシュ（`~/.gradle`）を保持してビルド時間を短縮する。
- アーティファクト保存: `actions/upload-artifact` を追加してビルド成果物を保持する。
- マトリクス: 複数の JDK バージョンや OS でのテストが必要な場合は `strategy.matrix` を利用する。
- 手動トリガー: `workflow_dispatch` を追加して手動実行やパラメータ入力を可能にする。
- 依存するサービス: DB 等外部サービスが必要な場合は `services:` を定義する。

---

## YAML スニペット（元ファイル）
以下は元のワークフローの要点（抜粋）です。

```yaml
name: Java CI with Gradle

on:
  push:
    branches:
      - "**"
  pull_request:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'

      - name: Grant execute permission for gradlew
        run: chmod +x gradlew

      - name: Build with Gradle
        run: ./gradlew build
```
