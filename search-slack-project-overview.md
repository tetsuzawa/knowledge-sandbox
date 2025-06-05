# Search Slack プロジェクト概要

## プロジェクト概要

**Search Slack** は、Slack上の会話から関連情報を検索し、質問に対して最適な回答を生成するAI駆動のツールです。Claude AIを活用して、ユーザーの質問に対してSlackの会話履歴から最も適切な検索クエリを生成し、関連する情報を抽出して回答を提供します。

## 主な機能

### 1. Slack検索機能
- ユーザーの質問から最適な検索クエリを自動生成
- Slack APIを使用して関連するスレッドを検索
- 検索結果をスレッド単位で整理して取得

### 2. メッセージ分類機能
メッセージの意図を以下のカテゴリに自動分類：
- 質問
- 相談
- 依頼
- 同意
- 共感
- 確認
- 感謝
- 失望
- その他

### 3. 回答生成機能
- 検索結果と質問内容から最適な回答を生成
- Claude AIを使用した自然な日本語での回答作成
- 検索結果の要約と関連情報の提供

### 4. Slack Bot連携
- Slack Botとして動作
- 特定のリアクション（+1）が付いたメッセージに対して自動処理
- リアルタイムでの質問応答システム

## 技術構成

### 開発環境
- **Python**: 3.12以上
- **パッケージ管理**: uv（高速なPythonパッケージマネージャー）

### 主要ライブラリ
- `slack-sdk>=3.31.0`: Slack API操作
- `anthropic[bedrock]>=0.34.1`: Claude AI（AWS Bedrock経由）
- `slack-bolt>=1.20.1`: Slack Botフレームワーク

### アーキテクチャ
```
src/search_slack/
├── __init__.py
├── search_slack.py          # Slack検索のコア機能
├── generate_search_query.py # Claude AIを使用した検索クエリ生成
├── categorize_message.py    # メッセージの意図分類
├── generate_response.py     # 検索結果から回答を生成
└── bolt_listener.py         # Slack Botリスナー機能
```

## セットアップ手順

### 1. リポジトリのクローン
```bash
git clone https://github.com/tetsuzawa/search-slack.git
cd search-slack
```

### 2. 依存関係のインストール
```bash
pip install -e .
```

### 3. 環境変数の設定
`.envrc.sample`ファイルを`.envrc`としてコピーし、必要なAPIキーを設定：

```bash
cp .envrc.sample .envrc
# .envrc を編集してAPIキーを設定
```

#### 必要な環境変数
- `AWS_ACCESS_KEY_ID`: AWSアクセスキーID
- `AWS_SECRET_ACCESS_KEY`: AWSシークレットアクセスキー
- `AWS_SESSION_TOKEN`: AWSセッショントークン（必要な場合）
- `OPENAI_API_KEY`: OpenAI APIキー（将来的な拡張用）
- `SLACK_BOT_TOKEN`: Slack Botトークン
- `SLACK_APP_TOKEN`: Slack Appトークン

## 使用方法

### 基本的なSlack検索
```python
from search_slack.search_slack import slack_search_and_threads

# 検索クエリを指定してSlackを検索
threads = slack_search_and_threads("検索クエリ")

# 結果を表示
for thread_ts, thread in threads.items():
    print(f"Thread: {thread.thread_ts}")
    print(f"Permalink: {thread.permalink}")
    for msg in thread.messages:
        print(f"- {msg.user}: {msg.text}")
```

### 検索クエリ生成
```python
from search_slack.generate_search_query import generate_search_queries

# 質問とスレッド情報から検索クエリを生成
queries = generate_search_queries("質問内容", "スレッド情報のJSON")

# 生成されたクエリを表示
print(queries)
```

### Slack Botとして実行
```bash
python -m search_slack.bolt_listener
```

## データ構造

### SlackMessage
```python
@dataclass
class SlackMessage:
    text: str
    user: str
    bot_id: str
    ts: str
    channel_id: str
    permalink: str = None
    attachments: list = None
```

### SlackThread
```python
@dataclass
class SlackThread:
    thread_ts: str
    permalink: str
    messages: list[SlackMessage]
```

## 特徴と利点

### AI駆動の検索
- Claude AIを使用してユーザーの質問に対して最適な検索クエリを自動生成
- 自然言語での質問を効果的なSlack検索クエリに変換

### 自動分類システム
- メッセージの意図を自動的に分類
- 適切な処理フローを選択して効率的な応答を実現

### Slack統合
- Slack Botとして動作し、リアクションベースでの自動処理
- 既存のSlackワークフローに自然に統合

### 柔軟な検索機能
- 複数の検索クエリを生成して包括的な情報収集
- スレッド単位での情報整理と提供

## 活用シーン

1. **社内Q&A**: 過去の会話から類似の質問と回答を検索
2. **ナレッジ共有**: 散在する情報を効率的に収集・整理
3. **問題解決支援**: 関連する議論や解決策を自動で提示
4. **オンボーディング**: 新メンバーの質問に対する迅速な情報提供

## ライセンス

このプロジェクトはMITライセンスの下で公開されています。

## 貢献

バグ報告や機能リクエストはGitHubのIssueで受け付けています。プルリクエストも歓迎します。

---

**参考リンク**
- [Search Slackリポジトリ](https://github.com/tetsuzawa/search-slack)
- [Slack API Documentation](https://api.slack.com/)
- [Claude AI Documentation](https://docs.anthropic.com/)