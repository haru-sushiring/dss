# サボらない開発支援サービス - 詳細技術仕様書

## 🎯 システム概要
土日や平日に「アプリ開発をサボらず続ける」ためのSlack連携型開発習慣支援サービス

---

## 📋 機能詳細仕様

### 1. ポイントシステム
#### ポイント獲得条件
- **Issue取組中**: 1ポイント (Issue作成・更新時)
- **PR作成**: +1ポイント (PR作成時)
- **Issue完了**: +1ポイント (Issueクローズ時)
- **コミット**: 1ポイント/コミット
- **PRレビュー・マージ**: 1ポイント/操作

#### ポイント集計
- **更新タイミング**: リアルタイム（GitHubのWebhook受信時）
- **週平均判定**: 毎週日曜日22:00に実行
- **基準**: 週平均1コミット（7日で7コミット）

#### 厳しさ判定
- **判定条件**: 1日3コミット以上必要な場合
- **通知タイミング**: 厳しい状況になった時点でSlack通知

### 2. レベル・ご褒美システム
#### ご褒美システム（累計ポイントベース）
```
レベル: 1, 3, 5, 8, 11, 15, 19, 24, 29, 35, 41, 48, 55, 63, 71, 80, 89, 99, 110, 120, 130...
差分: +2, +2, +3, +3, +4, +4, +5, +5, +6, +6, +7, +7, +8, +8, +9, +9, +10, +11, +10, +10...
```

#### レベルシステム（累計ポイントベース）
```
レベル: 10, 30, 50, 100, 150, 200, 250, 300, 350, 400, 450, 500...
差分: +20, +20, +50, +50, +50, +50, +50, +50, +50, +50, +50...
```

#### 通知メッセージ
- **ご褒美**: 「いいですね〜！」「順調です！」「素晴らしい進捗です！」
- **レベルアップ**: 「レベルアップ！よくやりました！」「すごいですね！新しいレベルに到達しました！」

### 3. Slack通知システム
#### 朝の通知（毎日7:00）
```
🌅 いい朝ですね！現在の残りIssueはこちらです。

📋 優先度高 Issues (5件):
1. [高] Issue Title 1 #123
2. [高] Issue Title 2 #124
3. [中] Issue Title 3 #125
4. [中] Issue Title 4 #126
5. [低] Issue Title 5 #127

📊 現在の進捗: 65% (13/20 Issues完了)
⭐ 累計ポイント: 45pt (レベル3)

[📝 稼働開始] ボタン
```

#### 催促通知（17:00）
- **条件**: 「稼働開始」ボタンが押されていない場合
```
⏰ 今日まだ何もしてませんが、大丈夫ですか？

[📝 今から稼働] ボタン
```

#### 成果チェック（21:00）
- **動作**: 自動でポイント集計、翌日の優先Issue選定
- **通知**: なし（バックグラウンド処理のみ）

#### 週次判定通知（日曜22:00）
```
📊 週次レポート
今週のコミット数: 5回
週平均目標: 7回 (達成率: 71%)

⚠️ 来週は少し頑張りましょう！
```

#### 厳しさ判定通知（リアルタイム）
```
🚨 スケジュール警告
残り3日で14コミット必要です (1日平均4.7コミット)
現実的に厳しいスケジュールです。

計画を見直すことをお勧めします。
```

#### プロジェクト完了チェック
```
🎊 残りIssueが1件になりました！
もうすぐプロジェクト完了ですね。

残りIssue: Issue Title #123

[✅ プロジェクト完了] [⏳ 継続] ボタン
```

### 4. GitHub連携
#### 対象リポジトリ
- **設定**: 1Bot = 1リポジトリ
- **対象**: プライベートリポジトリ対応
- **将来対応**: Organization配下のリポジトリ

#### Webhook対象イベント
- `push` (コミット検知)
- `issues` (Issue作成・更新・クローズ)
- `pull_request` (PR作成・マージ)
- `pull_request_review` (PRレビュー)

#### Issue優先度判定
- GitHubのラベルを使用
  - `priority:high` → 高
  - `priority:medium` → 中
  - `priority:low` → 低
  - ラベルなし → 低

---

## 🏗️ 技術アーキテクチャ

### システム構成
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Slack Bot     │    │   Web API       │    │   GitHub API    │
│   (通知・UI)     │◄──►│   (Node.js)     │◄──►│   (Webhook)     │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                │
                                ▼
                       ┌─────────────────┐
                       │   Supabase      │
                       │   (PostgreSQL)  │
                       └─────────────────┘
                                │
                                ▼
                       ┌─────────────────┐
                       │   Next.js       │
                       │   (Web UI)      │
                       └─────────────────┘
```

### 技術スタック
| 機能 | 技術 |
|------|------|
| Slack Bot | Slack Bolt.js |
| Web API | Node.js + TypeScript + Express |
| Database | Supabase (PostgreSQL) |
| Frontend | Next.js + TypeScript + Tailwind CSS |
| 定期実行 | node-cron |
| GitHub連携 | GitHub REST API + Webhooks |
| 認証 | Supabase Auth |

---

## 🗄️ データベース設計

### users テーブル
```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  slack_user_id VARCHAR(50) UNIQUE NOT NULL,
  email VARCHAR(255),
  github_username VARCHAR(100),
  github_token TEXT,
  target_repo VARCHAR(255),
  slack_channel_id VARCHAR(50),
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

### projects テーブル
```sql
CREATE TABLE projects (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES users(id),
  repo_name VARCHAR(255) NOT NULL,
  repo_url VARCHAR(500),
  is_active BOOLEAN DEFAULT true,
  total_issues INTEGER DEFAULT 0,
  completed_issues INTEGER DEFAULT 0,
  created_at TIMESTAMP DEFAULT NOW(),
  completed_at TIMESTAMP
);
```

### activities テーブル
```sql
CREATE TABLE activities (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES users(id),
  project_id UUID REFERENCES projects(id),
  activity_type VARCHAR(50) NOT NULL, -- 'commit', 'issue_create', 'issue_close', 'pr_create', 'pr_review'
  github_event_id VARCHAR(100),
  points INTEGER DEFAULT 1,
  description TEXT,
  activity_date DATE DEFAULT CURRENT_DATE,
  created_at TIMESTAMP DEFAULT NOW()
);
```

### daily_summaries テーブル
```sql
CREATE TABLE daily_summaries (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES users(id),
  project_id UUID REFERENCES projects(id),
  date DATE NOT NULL,
  total_points INTEGER DEFAULT 0,
  commits_count INTEGER DEFAULT 0,
  work_declared BOOLEAN DEFAULT false,
  created_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(user_id, project_id, date)
);
```

### user_levels テーブル
```sql
CREATE TABLE user_levels (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES users(id),
  current_level INTEGER DEFAULT 1,
  total_points INTEGER DEFAULT 0,
  last_reward_at INTEGER DEFAULT 0,
  last_level_up_at INTEGER DEFAULT 0,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

### priority_issues テーブル
```sql
CREATE TABLE priority_issues (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES users(id),
  project_id UUID REFERENCES projects(id),
  issue_number INTEGER NOT NULL,
  issue_title VARCHAR(500),
  priority VARCHAR(20) DEFAULT 'low',
  selected_for_tomorrow BOOLEAN DEFAULT false,
  selection_date DATE,
  created_at TIMESTAMP DEFAULT NOW()
);
```

---

## 🔄 処理フロー

### 1. 日次処理フロー
```
21:00 - 成果チェック処理
├── GitHub APIから全Issue取得
├── 優先度順にソート
├── 上位5件を翌日用に保存
└── daily_summariesテーブル更新

07:00 - 朝の通知
├── priority_issuesから5件取得
├── 進捗率計算 (completed_issues / total_issues)
├── 現在レベル・ポイント取得
└── Slack通知送信

17:00 - 催促通知
├── work_declared = false をチェック
└── 条件に合致すればSlack通知
```

### 2. GitHub Webhook処理フロー
```
Webhook受信
├── イベント種別判定
├── activitiesテーブルに記録
├── ポイント計算・付与
├── user_levelsテーブル更新
├── レベルアップ・ご褒美判定
└── 必要に応じてSlack通知
```

### 3. 週次処理フロー（日曜22:00）
```
週次判定処理
├── 過去7日のコミット数集計
├── 平均コミット数計算
├── 達成率算出
├── 厳しさ判定（将来の予測）
└── 週次レポートSlack通知
```

---

## 🚀 開発フェーズ

### フェーズ1: MVP（最小機能）
- [ ] Slack Bot基本設定
- [ ] 朝7:00の定期通知
- [ ] データベース基本設計
- [ ] GitHub API接続
- [ ] 基本的なポイント計算

### フェーズ2: 基本機能完成
- [ ] GitHub Webhook連携
- [ ] 17:00催促通知
- [ ] 稼働開始ボタン
- [ ] レベル・ご褒美システム
- [ ] 週次判定機能

### フェーズ3: UI・UX向上
- [ ] Next.js Web UI
- [ ] 進捗カレンダー表示
- [ ] 設定画面
- [ ] プロジェクト切り替え機能

### フェーズ4: 拡張機能
- [ ] 複数ユーザー対応
- [ ] Organization配下リポジトリ対応
- [ ] AI連携（要件定義支援）
- [ ] 詳細レポート機能

---

## ⚠️ エラー処理・例外対応

### GitHub API制限
- **制限**: 5000リクエスト/時間
- **対応**: レート制限監視、必要に応じて処理遅延
- **エラー時**: 管理者メール通知

### Slack API制限
- **制限**: 1メッセージ/秒
- **対応**: キュー機能実装
- **エラー時**: 管理者メール通知

### 通知失敗時の代替手段
- **Slack通知失敗**: 登録メールアドレスに通知
- **GitHub API失敗**: エラーログ保存、管理者通知
- **DB接続失敗**: ヘルスチェック、自動復旧試行

---

## 🔐 セキュリティ・認証

### GitHub認証
- **方式**: Personal Access Token
- **権限**: repo (プライベートリポジトリアクセス用)
- **保存**: Supabase暗号化保存

### Slack認証
- **方式**: OAuth 2.0
- **権限**: channels:read, chat:write, commands
- **保存**: Supabase暗号化保存

### Webhook検証
- **GitHub**: X-Hub-Signature-256検証
- **Slack**: 署名検証

---

## 📊 監視・ログ

### 監視項目
- API応答時間
- エラー発生率
- 通知送信成功率
- データベース接続状況

### ログ出力
- API呼び出しログ
- エラーログ
- ユーザーアクティビティログ
- システムヘルスログ

---

## 🎯 成功指標

### 定量指標
- 週平均コミット数の維持
- 連続開発日数
- Issue完了率
- システム稼働率 (99%以上)

### 定性指標
- 開発習慣の継続
- モチベーション維持
- サボり防止効果
- ユーザー満足度
