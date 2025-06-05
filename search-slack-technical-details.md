# Search Slack - 技術詳細

## アーキテクチャ詳細

### システム構成図

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Slack User    │    │   Slack Bot     │    │   Claude AI     │
│                 │    │                 │    │   (Bedrock)     │
└─────────┬───────┘    └─────────┬───────┘    └─────────┬───────┘
          │                      │                      │
          │ 1. リアクション(+1)    │                      │
          ├─────────────────────►│                      │
          │                      │ 2. 質問クエリ生成      │
          │                      ├─────────────────────►│
          │                      │                      │
          │                      │ 3. 生成されたクエリ    │
          │                      │◄─────────────────────┤
          │                      │                      │
          │                      │                      │
          │ 5. 回答メッセージ       │                      │
          │◄─────────────────────┤                      │
          │                      │ 4. 回答生成           │
          │                      ├─────────────────────►│
          │                      │                      │
          │                      │ 回答テキスト          │
          │                      │◄─────────────────────┤
```

### コンポーネント詳細

#### 1. search_slack.py - コア検索機能

**主要クラス:**
- `SlackMessage`: Slackメッセージのデータ構造
- `SlackThread`: Slackスレッドのデータ構造

**主要関数:**
- `slack_search_and_threads(search_query: str)`: Slack検索とスレッド取得
- `get_value_or_default(dictionary, key, default_value)`: 安全な辞書アクセス

**処理フロー:**
1. Slack Web APIを使用してメッセージ検索
2. 検索結果から各メッセージのスレッドを取得
3. メッセージとスレッドをデータクラスに変換
4. 辞書形式で結果を返却

#### 2. generate_search_query.py - AI検索クエリ生成

**機能:**
- ユーザーの自然言語質問をSlack検索クエリに変換
- Claude AIを使用した意図理解と最適化
- 複数の検索クエリ候補を生成

**処理例:**
```python
# 入力: "クラウドアカウント接続方法"
# 出力: ["クラウドアカウント 接続方法", "Organization連携 確認"]
```

#### 3. categorize_message.py - メッセージ分類

**分類カテゴリ:**
- 質問: 情報を求める内容
- 相談: アドバイスを求める内容
- 依頼: 作業や対応を求める内容
- 同意: 賛成や承認を示す内容
- 共感: 理解や共感を示す内容
- 確認: 事実や状況の確認
- 感謝: お礼や感謝の表現
- 失望: 不満や失望の表現
- その他: 上記に該当しない内容

#### 4. generate_response.py - 回答生成

**機能:**
- 検索結果の要約と整理
- 質問に対する適切な回答の生成
- 関連情報の提示

#### 5. bolt_listener.py - Slack Bot

**イベント処理:**
- `@app.message("bolt")`: "bolt"を含むメッセージの処理
- `@app.event("reaction_added")`: リアクション追加イベントの処理

**リアクション処理フロー:**
1. +1リアクションの検出
2. 対象メッセージのスレッド取得
3. メッセージ内容の分析と処理
4. 必要に応じて回答生成

## API仕様

### Slack API使用箇所

#### 1. メッセージ検索
```python
response = client.search_messages(query=search_query, count=5)
```

#### 2. スレッド取得
```python
thread = client.conversations_replies(channel=msg.channel_id, ts=msg.ts)
```

### Claude AI API使用箇所

#### 1. 検索クエリ生成
- エンドポイント: AWS Bedrock Claude API
- モデル: Claude-3 (Sonnet/Haiku)
- 用途: 自然言語からSlack検索クエリへの変換

#### 2. 回答生成
- エンドポイント: AWS Bedrock Claude API
- モデル: Claude-3 (Sonnet)
- 用途: 検索結果からの回答生成

## 設定とデプロイ

### 環境変数詳細

```bash
# AWS設定（Claude AI用）
export AWS_ACCESS_KEY_ID="your-access-key"
export AWS_SECRET_ACCESS_KEY="your-secret-key"
export AWS_SESSION_TOKEN="your-session-token"  # 一時的な認証の場合

# Slack設定
export SLACK_BOT_TOKEN="xoxb-your-bot-token"
export SLACK_APP_TOKEN="xapp-1-your-app-token"

# 将来的な拡張用
export OPENAI_API_KEY="sk-your-openai-key"
```

### Slack App設定

#### 必要な権限（OAuth Scopes）:
- `channels:history`: チャンネル履歴の読み取り
- `groups:history`: プライベートチャンネル履歴の読み取り
- `im:history`: ダイレクトメッセージ履歴の読み取り
- `mpim:history`: グループダイレクトメッセージ履歴の読み取り
- `search:read`: 検索機能の使用
- `reactions:read`: リアクションの読み取り
- `chat:write`: メッセージの送信

#### イベント購読:
- `reaction_added`: リアクション追加イベント
- `message.channels`: チャンネルメッセージ
- `message.groups`: プライベートチャンネルメッセージ
- `message.im`: ダイレクトメッセージ
- `message.mpim`: グループダイレクトメッセージ

### デプロイ方法

#### 1. ローカル開発
```bash
# 依存関係インストール
pip install -e .

# Bot起動
python -m search_slack.bolt_listener
```

#### 2. Docker デプロイ
```dockerfile
FROM python:3.12-slim

WORKDIR /app
COPY . .
RUN pip install -e .

CMD ["python", "-m", "search_slack.bolt_listener"]
```

#### 3. AWS Lambda デプロイ
- Slack Bolt for Pythonのlambda_handlerを使用
- 環境変数をLambda環境に設定
- API Gatewayとの連携設定

## パフォーマンス考慮事項

### 1. レート制限
- Slack API: 1分間に50リクエスト
- Claude AI: 使用量に応じた制限

### 2. レスポンス時間最適化
- 検索結果の件数制限（デフォルト5件）
- 並列処理の活用
- キャッシュ機能の実装（将来的な改善）

### 3. エラーハンドリング
- Slack API エラーの適切な処理
- Claude AI API エラーの処理
- ネットワークエラーの再試行機能

## セキュリティ考慮事項

### 1. 認証情報の管理
- 環境変数での機密情報管理
- AWS IAMロールの最小権限原則
- Slackトークンの適切な管理

### 2. データプライバシー
- Slackメッセージの一時的な処理
- ログ出力時の機密情報マスキング
- Claude AIへの送信データの最小化

### 3. アクセス制御
- Slack Appの適切な権限設定
- チャンネルアクセス権限の尊重
- ユーザー認証の確認

## 監視とログ

### 1. ログ出力
- 検索クエリと結果のログ
- エラー発生時の詳細ログ
- パフォーマンスメトリクス

### 2. 監視項目
- API呼び出し回数
- レスポンス時間
- エラー率
- ユーザー利用状況

## 今後の拡張予定

### 1. 機能拡張
- 検索結果のキャッシュ機能
- より高度な意図理解
- 多言語対応

### 2. UI/UX改善
- Slack Slash Commandsの対応
- インタラクティブなボタン機能
- 検索結果のフォーマット改善

### 3. 分析機能
- 利用統計の収集
- 検索パターンの分析
- 回答品質の評価機能