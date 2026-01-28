# CI ワークフロー API 仕様書

ファイル: `.github/workflows/ci.yml`

## 概要
このワークフローは、Gradle を使用した Java プロジェクトのビルドとテストを自動で実行するための GitHub Actions ワークフローです。JDK 17（Temurin）をセットアップし、Gradle Wrapper を使って `./gradlew build` を実行します。ビルド結果はワークフローの成功/失敗で判断します。

---

## トリガー（Triggers）
- push: すべてのブランチ（branches: - "**"）への push によりトリガーされます。
- pull_request: `main` ブランチへのプルリクエスト作成／更新時にトリガーされます。

---

## Jobs
- ジョブ名: `build`
  - 実行環境: `ubuntu-latest`
  - 概要: リポジトリをチェックアウトし、JDK 17 をセットアップした上で Gradle Wrapper に実行権限を付与し、ビルド（およびテスト）を実行します。

---

## Inputs / Parameters
なし

---

## Steps
各ステップの処理内容は以下の通りです。

1. Checkout repository
   - name: `Checkout repository`
   - uses: `actions/checkout@v4`
   - 説明: リポジトリのソースコードをワークフロー実行環境に取得します。

2. Set up JDK 17
   - name: `Set up JDK 17`
   - uses: `actions/setup-java@v4`
   - with:
     - distribution: `temurin`
     - java-version: `17`
   - 説明: Temurin ディストリビューションの Java 17 をインストールし、環境変数（例: JAVA_HOME）を設定します。

3. Grant execute permission for gradlew
   - name: `Grant execute permission for gradlew`
   - run: `chmod +x gradlew`
   - 説明: Linux ランナー上で `gradlew` に実行権限を与えます（Windows では不要／別途対応が必要）。

4. Build with Gradle
   - name: `Build with Gradle`
   - run: `./gradlew build`
   - 説明: Gradle Wrapper を用いてプロジェクトのビル��とテストを実行します（`build` タスク）。

---

## 環境変数 / シークレット
なし

---

## Output
なし

---

## 権限（Permissions）
なし
