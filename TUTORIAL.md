# 実践チュートリアル: 従業員管理システム

<!--
📖 このファイルについて
========================
実務で必須となるGitHub・Java・PL/SQL・AIコーディングの基礎を実践的に学ぶチュートリアルです。
従業員管理システムを題材に、各技術を統合的に学習します。

💡 チュートリアルの進め方
========================
1. 各Phaseを順番に進めてください
2. 「AIへのプロンプト」をGemini CLIにコピペして実行
3. AIの回答を確認しながら、コマンドやコードを理解
4. 最後にスキルチェックリストで習得度を確認
-->

働くうえで必須となるGitHub・Java・PL/SQL・AIコーディングを**1つのプロジェクトで実践的に学ぶ**チュートリアルです。

> [!TIP]
> 学習用ファイルは `/home/ryo_onishi/work/java_pl_sql_test/` に作成してください。

<!--
💡 > [!TIP] とは？
=================
GitHub Flavored Markdownの「アラート」記法です。
TIP, NOTE, WARNING, CAUTION などの種類があり、
目立つボックスで情報を強調表示します。
-->

---

## 🎯 学習目標

| 技術 | 習得スキル |
|------|------------|
| **GitHub** | Issue管理、ブランチ、PR、gh CLI、リリース |
| **Java** | クラス設計、JDBC、例外処理、ストアドプロシージャ呼出 |
| **PL/SQL** | テーブル設計、プロシージャ、トリガー、カーソル |
| **AIコーディング** | 効果的なプロンプト、対話的開発、コードレビュー |

<!--
💡 各技術の関係性
=================
                  ┌─────────────┐
                  │   GitHub    │ ← コード管理・チーム協業
                  └─────────────┘
                         ↓
┌───────────────────────────────────────────┐
│               Javaアプリ                  │ ← ビジネスロジック
│  (JDBC経由でDBと接続)                    │
└───────────────────────────────────────────┘
                         ↓
┌───────────────────────────────────────────┐
│           Oracle Database                 │
│  (PL/SQL: プロシージャ、トリガー)        │ ← データ永続化
└───────────────────────────────────────────┘
-->

---

## 🚀 Phase 1: プロジェクトセットアップ

<!--
📌 Phase 1の目標
================
- GitHubリポジトリを作成する
- Issueでタスクを管理する
- ブランチ戦略を理解する
-->

### 1-1: GitHub認証とリポジトリ作成

```bash
# GitHub CLIの認証
gh auth login
# ↑ GitHubアカウントとCLIを連携させます
#   初回のみ必要。ブラウザが開いて認証画面が表示されます
#   
#   認証フロー:
#   1. HTTPS or SSH を選択（HTTPSがおすすめ）
#   2. ブラウザで認証（Login with a web browser を選択）
#   3. 表示されたコードを入力
#   4. GitHubでAuthorize

# リポジトリ作成
gh repo create employee-management-system --public --clone
# ↑ gh repo create: 新しいリポジトリを作成
#   employee-management-system: リポジトリ名
#   --public: 公開リポジトリとして作成（--private で非公開）
#   --clone: 作成後にローカルにクローン（ダウンロード）

cd employee-management-system
# ↑ 作成したリポジトリのディレクトリに移動
```

**AIへのプロンプト:**
```
employee-management-systemのREADME.mdと
Java用の.gitignoreを作成してください。
```

<!--
💡 .gitignoreとは？
==================
Gitで追跡しないファイルを指定するファイルです。
例: コンパイル済みファイル(.class)、設定ファイル、一時ファイルなど
これらをGitHubにアップロードしないようにします。
-->

### 1-2: Issue作成（タスク分割）

<!--
💡 Issueとは？
=============
GitHubでタスク、バグ、機能要望を管理する機能です。
- 各作業をIssueとして登録
- ラベルで分類（bug, enhancement, documentation など）
- 担当者をアサイン
- PRとリンク（Closes #1 でIssueを自動クローズ）
-->

**AIへのプロンプト:**
```
ghコマンドで以下のIssueを作成してください：
#1 「データベース設計」- ラベル: database
#2 「PL/SQLストアドプロシージャ」- ラベル: database
#3 「JavaモデルとDAO」- ラベル: java
#4 「メインアプリ作成」- ラベル: java
```

<!--
↑ AIが実行するコマンド例:
gh issue create --title "データベース設計" --label "database"
gh issue create --title "PL/SQLストアドプロシージャ" --label "database"
gh issue create --title "JavaモデルとDAO" --label "java"
gh issue create --title "メインアプリ作成" --label "java"
-->

### 1-3: ブランチ戦略

<!--
💡 ブランチ戦略とは？
====================
開発の流れを整理するためのルールです。
一般的なパターン:
  main ← 本番用（常に動作する状態）
    ↑
  develop ← 開発用（機能をマージする場所）
    ↑
  feature/xxx ← 機能開発用（各機能ごとに作成）
-->

```bash
# developブランチ作成
git switch -c develop
# ↑ git switch: ブランチを切り替えるコマンド
#   -c: 新しいブランチを作成して切り替え（create）
#   develop: 新しいブランチ名
#
# 従来のコマンド git checkout -b develop と同じ意味
# git switch は新しい推奨コマンドです

git push -u origin develop
# ↑ git push: ローカルの変更をリモートに送信
#   -u: upstream（追跡ブランチ）を設定
#       次回から git push だけでOKになる
#   origin: リモートリポジトリの名前（通常はorigin）
#   develop: プッシュするブランチ名
```

---

## 🗄️ Phase 2: データベース設計（PL/SQL）

<!--
📌 Phase 2の目標
================
- Oracleテーブルを設計・作成する
- ストアドプロシージャを作成する
- トリガーで自動処理を実装する
-->

**ブランチ:** `git switch -c feature/database`
<!-- ↑ 機能開発用のブランチを作成して切り替え -->

### 2-1: テーブル作成

**AIへのプロンプト:**
```
従業員管理用に以下のOracleテーブルを設計してください：
- DEPARTMENTS（部署）: dept_id, dept_name
- EMPLOYEES（社員）: emp_id, emp_name, dept_id, salary, hire_date
- SALARY_HISTORY（給与履歴）: history_id, emp_id, old_salary, new_salary, changed_date
- AUDIT_LOG（監査）: log_id, table_name, action, action_date, user_name

ER図とCREATE TABLE文をお願いします。
```

<!--
💡 テーブル設計のポイント
========================
- 主キー (Primary Key): 各行を一意に識別（emp_id など）
- 外部キー (Foreign Key): 他テーブルとの関連（dept_id など）
- NOT NULL: 必須項目の指定
- DEFAULT: デフォルト値の設定
-->

**SQL基礎（Python比較）:**

<!--
💡 PythonからSQLへの思考の切り替え
==================================
Pythonは「手続き的」（順番に処理を書く）
SQLは「宣言的」（欲しい結果を書く）
-->

| Python (pandas) | SQL | 説明 |
|-----------------|-----|------|
| `df[df['age'] > 20]` | `SELECT * FROM t WHERE age > 20` | 条件で行を絞り込む |
| `df.groupby('dept')` | `GROUP BY dept` | グループ化して集計 |
| `pd.merge(df1, df2)` | `JOIN` | 複数テーブルを結合 |

### 2-2: ストアドプロシージャ

<!--
💡 ストアドプロシージャとは？
============================
データベース内に保存された処理プログラムです。
メリット:
- パフォーマンス向上（事前コンパイル）
- セキュリティ向上（SQLを直接公開しない）
- 再利用性（複数のアプリから呼び出せる）
-->

**AIへのプロンプト:**
```
以下のPL/SQLプロシージャを作成してください：

1. add_employee(p_name, p_dept_id, p_salary)
   - EMPLOYEESに追加し、追加したemp_idを返す

2. update_salary(p_emp_id, p_new_salary)  
   - 給与更新し、SALARY_HISTORYに履歴を自動記録

3. get_dept_employees(p_dept_id) 
   - カーソルで部署の社員一覧を返す

呼び出し例もつけてください。
```

**PL/SQL基本構造:**
```sql
CREATE OR REPLACE PROCEDURE proc_name (params) AS
-- ↑ CREATE OR REPLACE: 作成または上書き
--   PROCEDURE: プロシージャを定義
--   proc_name: プロシージャ名
--   (params): パラメータ（引数）

    -- 変数宣言セクション
    v_count NUMBER;  -- 変数名 データ型;
BEGIN
    -- 処理セクション（実際の処理を書く）
    SELECT COUNT(*) INTO v_count FROM employees;
    
EXCEPTION
    -- 例外処理セクション（エラー時の処理）
    WHEN OTHERS THEN
        -- OTHERS: すべての例外をキャッチ
        DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM);
END;
/
-- ↑ / は SQL*Plus でプロシージャを実行するための終端記号
```

### 2-3: トリガー

<!--
💡 トリガーとは？
================
テーブルへの操作（INSERT/UPDATE/DELETE）時に
自動的に実行される処理です。
例: 変更履歴の自動記録、データの自動検証など
-->

**AIへのプロンプト:**
```
EMPLOYEESテーブルへのINSERT/UPDATE/DELETE時に
AUDIT_LOGへ自動記録するトリガーを作成してください。
記録内容: テーブル名、操作種別、日時、ユーザー名
```

<!--
💡 トリガーの実行タイミング
===========================
BEFORE: 操作の「前」に実行（値の検証・変更に使う）
AFTER: 操作の「後」に実行（ログ記録に使う）
INSTEAD OF: ビューに対する操作を置き換え
-->

### 2-4: PRを作成

```bash
git add .
# ↑ すべての変更をステージング（コミット対象に追加）
#   . はカレントディレクトリ以下のすべてのファイル

git commit -m "feat: add database schema and procedures"
# ↑ 変更をコミット（保存）
#   -m: コミットメッセージを指定
#   "feat: ..." は Conventional Commits 形式
#     feat: 新機能
#     fix: バグ修正
#     docs: ドキュメント
#     refactor: リファクタリング

git push -u origin feature/database
# ↑ ブランチをリモートにプッシュ

gh pr create --base develop --title "Database設計" --body "Closes #1, #2"
# ↑ Pull Requestを作成
#   --base develop: マージ先ブランチ
#   --title: PRのタイトル
#   --body: PRの説明文
#   "Closes #1, #2": PRがマージされるとIssue #1, #2 が自動クローズ
```

---

## ☕ Phase 3: Javaアプリケーション

<!--
📌 Phase 3の目標
================
- Mavenプロジェクトを作成する
- モデルクラス（DTO）を設計する
- JDBC でデータベースに接続する
- ストアドプロシージャをJavaから呼び出す
-->

**ブランチ:** `git switch -c feature/java-app`

### 3-1: プロジェクト構造

<!--
💡 Javaプロジェクトの構造
========================
一般的なMavenプロジェクトの構造:
project/
├── pom.xml          ← 依存関係・ビルド設定
├── src/
│   ├── main/
│   │   ├── java/    ← ソースコード
│   │   └── resources/ ← 設定ファイル
│   └── test/
│       └── java/    ← テストコード
-->

**AIへのプロンプト:**
```
以下の構造でMavenプロジェクトを作成してください：
src/main/java/com/example/employee/
  ├── model/Employee.java, Department.java
  ├── dao/EmployeeDAO.java
  ├── service/EmployeeService.java
  └── Main.java

pom.xmlにOracle JDBCドライバを追加してください。
```

<!--
💡 各レイヤーの役割
==================
model/: データを保持するクラス（Entity, DTO）
dao/: データベースアクセス（Data Access Object）
service/: ビジネスロジック
Main.java: アプリケーションのエントリーポイント
-->

### 3-2: モデルクラス

**AIへのプロンプト:**
```
Employee.javaを作成してください：
- フィールド: id, name, departmentId, salary, hireDate
- コンストラクタ（全引数、デフォルト）
- getter/setter全て
- toString()

Pythonとの比較も教えてください。
```

**Java vs Python 型比較:**

<!--
💡 静的型付け vs 動的型付け
===========================
Python（動的型付け）: 実行時に型が決まる
Java（静的型付け）: コンパイル時に型が決まる

静的型付けのメリット:
- コンパイル時にエラーを検出できる
- IDEの補完が効きやすい
- コードの意図が明確になる
-->

| Python | Java | 説明 |
|--------|------|------|
| `x = 10` | `int x = 10;` | 変数宣言（Javaは型を明示） |
| `name = "太郎"` | `String name = "太郎";` | 文字列 |
| `flag = True` | `boolean flag = true;` | 真偽値（大文字小文字に注意） |
| 動的型付け | 静的型付け | 型の決定タイミング |

### 3-3: JDBC接続とDAO

<!--
💡 JDBCとは？（Java Database Connectivity）
==========================================
JavaからデータベースにアクセスするためのAPIです。
標準化されているため、DB製品を問わず同じコードで接続できます。
-->

**AIへのプロンプト:**
```
EmployeeDAO.javaを作成してください：
- DB接続情報はプロパティファイルから読込
- メソッド:
  - findAll(): List<Employee>
  - findById(int id): Optional<Employee>
  - save(Employee): ストアドプロシージャadd_employee呼出
  - updateSalary(int id, BigDecimal salary): update_salary呼出

try-with-resourcesで接続管理、PreparedStatement使用。
```

**JDBC基本パターン:**
```java
// try-with-resources: リソースを自動でクローズする構文
try (Connection conn = DriverManager.getConnection(url, user, pass);
     // ↑ データベースへの接続を取得
     //   url: "jdbc:oracle:thin:@localhost:1521:XE" のような接続文字列
     
     PreparedStatement ps = conn.prepareStatement(sql)) {
     // ↑ SQLを事前にコンパイル（パフォーマンス向上、SQLインジェクション対策）
    
    ps.setInt(1, id);
    // ↑ プレースホルダー（?）に値をセット
    //   1: 1番目の ? に、id の値をセット
    
    ResultSet rs = ps.executeQuery();
    // ↑ SELECT文を実行し、結果を取得
    //   INSERT/UPDATE/DELETEの場合は executeUpdate() を使用
    
    // 結果処理
    while (rs.next()) {
        // rs.getString("name"), rs.getInt("id") などで値を取得
    }
    
} catch (SQLException e) {
    // エラー処理
    e.printStackTrace();
}
// ← try-with-resources のおかげで
//   Connection, PreparedStatement は自動的にクローズされる
```

### 3-4: メインアプリ

**AIへのプロンプト:**
```
コンソールメニュー形式のMain.javaを作成してください：
1. 社員一覧表示
2. 社員追加
3. 給与更新
4. 部署別検索
5. 終了

Scanner使用、入力バリデーション付きで。
```

<!--
💡 Scannerとは？
===============
ユーザー入力を読み取るためのJavaクラスです。
Scanner sc = new Scanner(System.in);
String input = sc.nextLine();  // 1行読み取り
int num = sc.nextInt();        // 整数を読み取り
-->

### 3-5: PRを作成

```bash
git add .
git commit -m "feat: add Java application"
git push -u origin feature/java-app
gh pr create --base develop --title "Javaアプリ" --body "Closes #3, #4"
```

---

## 🔄 Phase 4: 結合・テスト・リリース

<!--
📌 Phase 4の目標
================
- 各ブランチをマージする
- 統合テストを実施する
- タグ付けしてリリースする
-->

### 4-1: マージ

```bash
# developにマージ（GitHub上でPRをマージするか、以下）
git switch develop
# ↑ developブランチに切り替え

git merge feature/database
# ↑ feature/database ブランチの変更を取り込む
#   マージ: 2つのブランチを統合すること

git merge feature/java-app
# ↑ feature/java-app ブランチの変更も取り込む

git push
# ↑ マージ結果をリモートにプッシュ
```

### 4-2: 統合テスト

<!--
💡 統合テストとは？
==================
各コンポーネント（DB, Java, API など）を
結合した状態でテストすることです。
単体テスト（個別の関数テスト）の後に行います。
-->

**AIへのプロンプト:**
```
以下のテストシナリオを実行してください：
1. 部署を2つ追加（営業部、開発部）
2. 社員を3人追加（各部署に配属）
3. 1人の給与を更新
4. SALARY_HISTORYに履歴があることを確認
5. AUDIT_LOGに操作ログがあることを確認

SQLとJavaの両方から確認してください。
```

### 4-3: リリース

```bash
# mainにマージ
git switch main
git merge develop
git push

# タグ付けとリリース
git tag v1.0.0
# ↑ タグを作成
#   タグ: 特定のコミットに名前をつける（バージョン番号など）
#   v1.0.0 はセマンティックバージョニング
#     メジャー.マイナー.パッチ
#     1: メジャーバージョン（破壊的変更時に上げる）
#     0: マイナーバージョン（機能追加時に上げる）
#     0: パッチバージョン（バグ修正時に上げる）

git push origin v1.0.0
# ↑ タグをリモートにプッシュ

gh release create v1.0.0 --title "v1.0.0" --notes "初回リリース"
# ↑ GitHubリリースを作成
#   リリース: ダウンロード可能な成果物を公開する機能
#   --title: リリースタイトル
#   --notes: リリースノート（変更内容の説明）
```

---

## 📋 Phase 5: 仕様駆動開発（オプション）

<!--
📌 Phase 5の目標
================
- 仕様駆動開発（SDD）の概念を理解する
- cc-sddを使って仕様書を生成する
- GitHub Projectと連携してタスク管理する
-->

> [!TIP]
> 仕様駆動開発とは、コードの前に仕様書（ドキュメント）を生成し、それに基づいて開発する手法です。AIコーディングと相性が良く、メンテナンス性が向上します。

### 5-1: cc-sddセットアップ

```bash
# プロジェクトディレクトリで実行
npx cc-sdd@latest --gemini --lang ja
# ↑ cc-sddをGemini CLI向けにセットアップ
#   --gemini: Gemini CLI用の設定
#   --lang ja: 日本語でドキュメント生成
```

### 5-2: 仕様書の生成

**Gemini CLIで実行:**
```
/kiro:steering
```
<!-- ↑ 既存ファイルを読み取って初期ドキュメントを生成 -->

**新機能を開発する場合:**
```
/kiro:spec-init 従業員の勤怠管理機能
```
<!-- ↑ 新機能の仕様書を対話的に生成 -->

### 5-3: GitHub Projectと連携

```bash
# GitHub Projectを操作する権限を追加
gh auth refresh -s project
# ↑ ghコマンドでProjectを操作するための権限を付与
```

**Gemini CLIで実行:**
```
@tasks.md をGitHubのissueにghコマンドで登録してください
```

```
すべてのissuesにStart dateとTarget dateを設定してください
1ヶ月で終わるように、タスクの難易度に応じてスケジュールを設定してください
```

### 5-4: タスク実装

**Gemini CLIで実行:**
```
/kiro:spec-impl task.mdをもとに順に実装してください。
タスクはGitHub issueとも対応しているので、そちらも合わせて対処してください
```

> 生成されたドキュメントは `.kiro/steering/` に保存されます。  
> `.kiro/settings/` はcc-sdd設定なので `.gitignore` に追加推奨。

---

## 📝 AIプロンプトのコツ

### ✅ 良いプロンプト

```
EmployeeDAOのfindByIdメソッドを作成してください。

要件:
- Oracle JDBC使用
- PreparedStatement使用  
- 見つからない場合はOptional.empty()を返す
- SQLExceptionは呼び出し元に投げる

現在のEmployeeクラス:
[コード貼り付け]
```

<!--
💡 良いプロンプトの構成要素
===========================
1. 何を作るか（findByIdメソッド）
2. 技術的制約（Oracle JDBC, PreparedStatement）
3. 期待する振る舞い（Optional.empty()を返す）
4. エラー処理方針（SQLExceptionを投げる）
5. コンテキスト（Employeeクラスのコード）
-->

### ❌ 避けるプロンプト

- 「DAOクラス作って」（曖昧）
- 「完璧なコードで」（基準不明）
- 「エラーが出る」（詳細なし）

<!--
💡 なぜダメ？
============
- 曖昧: AIが何を求められているかわからない
- 基準不明: 「完璧」の定義が人それぞれ違う
- 詳細なし: エラーメッセージやスタックトレースがないと調査できない
-->

### レビュー依頼

```
以下のコードをレビューしてください。
改善点、パフォーマンス、セキュリティの観点で。

[コード貼り付け]
```

---

## 📊 スキルチェックリスト

<!--
💡 チェックリストの使い方
========================
各項目を実際に手を動かして確認してください。
チェック記号:
- [ ] 未完了
- [x] 完了
-->

### GitHub
- [ ] リポジトリ作成（gh repo create）
- [ ] Issue作成・ラベル付け
- [ ] ブランチ作成・切替（git switch -c）
- [ ] PR作成・マージ（gh pr create）
- [ ] タグ・リリース作成

### PL/SQL
- [ ] CREATE TABLE（主キー・外部キー）
- [ ] ストアドプロシージャ（IN/OUTパラメータ）
- [ ] トリガー（BEFORE/AFTER）
- [ ] カーソル（FETCH、FORループ）
- [ ] 例外処理（EXCEPTION WHEN）

### Java
- [ ] クラス設計（フィールド、コンストラクタ、getter/setter）
- [ ] JDBC接続（DriverManager）
- [ ] PreparedStatement/CallableStatement
- [ ] try-with-resources
- [ ] Optional使用

### AIコーディング
- [ ] 具体的な要件記述
- [ ] コンテキスト（既存コード）の提供
- [ ] 段階的な指示
- [ ] コードレビュー依頼

---

## 🔗 参考リンク

| カテゴリ | リンク | 説明 |
|----------|--------|------|
| GitHub | [GitHub Docs](https://docs.github.com/ja) | 公式ドキュメント（日本語） |
| GitHub | [Pro Git Book](https://git-scm.com/book/ja/v2) | Git入門書（無料） |
| Java | [Oracle Java Tutorial](https://docs.oracle.com/javase/tutorial/) | 公式チュートリアル |
| PL/SQL | [Oracle PL/SQL Docs](https://docs.oracle.com/en/database/oracle/oracle-database/19/lnpls/) | 公式リファレンス |
| SQL | [W3Schools SQL](https://www.w3schools.com/sql/) | SQL入門（インタラクティブ） |

---

## 📅 推奨スケジュール

| Phase | 内容 | 目安時間 | 難易度 |
|-------|------|----------|--------|
| 1 | セットアップ | 1-2時間 | ⭐ |
| 2 | PL/SQL | 3-4時間 | ⭐⭐ |
| 3 | Java | 4-6時間 | ⭐⭐⭐ |
| 4 | 結合・リリース | 1-2時間 | ⭐⭐ |
| **合計** | | **10-14時間** | |

<!--
💡 効率的に進めるコツ
====================
- わからないことはすぐにAIに質問
- エラーが出たらエラーメッセージをそのままAIに貼り付け
- 1つのPhaseを終えたら必ずコミット
- 疲れたら休憩（集中力が落ちるとミスが増える）
-->
