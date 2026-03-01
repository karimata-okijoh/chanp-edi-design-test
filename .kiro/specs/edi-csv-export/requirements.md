# 要件定義書（Requirements Document）

## はじめに（Introduction）

本文書は、既存のMS-Accessアプリケーションに共通EDI準拠のCSV出力機能を追加する新機能の要件を定義します。この機能により、発注データを標準化されたEDIフォーマットでCSV出力し、サービスプロバイダーを介してサプライヤーと電子的にデータ交換することが可能になります。

## 用語集（Glossary）

- **EDI_Export_System**: MS-Accessアプリケーションに追加される共通EDI準拠のCSV出力機能
- **Order_Data**: MS-Accessデータベースに格納されている発注情報
- **EDI_CSV**: 共通EDI標準に準拠したフォーマットのCSVファイル
- **Service_Provider**: EDIデータの送受信を仲介するサービスプロバイダー
- **Supplier**: 発注データを受け取る取引先企業
- **Access_Database**: 既存のMS-Accessアプリケーションのデータベース
- **Export_Configuration**: CSV出力時の設定情報（文字コード、区切り文字、ヘッダー有無など）

## 要件（Requirements）

### 要件1: 発注データの抽出

**ユーザーストーリー:** 発注担当者として、MS-Accessデータベースから発注データを抽出したい。これにより、EDI形式でのデータ送信が可能になる。

#### 受入基準（Acceptance Criteria）

1. WHEN ユーザーがCSV出力を実行する、THE EDI_Export_System SHALL Access_Databaseから未送信の発注データを抽出する
2. THE EDI_Export_System SHALL 抽出した発注データに発注番号、発注日、品目コード、数量、単価、納期を含める
3. WHEN 抽出対象の発注データが存在しない、THE EDI_Export_System SHALL ユーザーに対象データなしのメッセージを表示する
4. THE EDI_Export_System SHALL 抽出した発注データの件数をユーザーに表示する

### 要件2: 共通EDIフォーマットへの変換

**ユーザーストーリー:** システム管理者として、発注データを共通EDI標準に準拠したフォーマットに変換したい。これにより、異なるシステム間でのデータ互換性が確保される。

#### 受入基準（Acceptance Criteria）

1. THE EDI_Export_System SHALL 抽出した発注データを共通EDIフォーマットの項目配置に変換する
2. THE EDI_Export_System SHALL 日付データをYYYYMMDD形式に変換する
3. THE EDI_Export_System SHALL 数値データを右詰めゼロ埋め形式に変換する
4. THE EDI_Export_System SHALL 文字データを左詰めスペース埋め形式に変換する
5. WHEN 必須項目が欠落している、THE EDI_Export_System SHALL エラーメッセージを表示し出力を中止する
6. THE EDI_Export_System SHALL 各項目の文字数制限を共通EDI仕様に従って適用する

### 要件3: CSV形式での出力

**ユーザーストーリー:** 発注担当者として、変換されたデータをCSVファイルとして出力したい。これにより、サービスプロバイダーへのアップロードが可能になる。

#### 受入基準（Acceptance Criteria）

1. THE EDI_Export_System SHALL 変換されたデータをCSV形式でファイルに出力する
2. THE EDI_Export_System SHALL CSVファイルの文字コードをUTF-8で出力する
3. THE EDI_Export_System SHALL CSVファイルの区切り文字としてカンマを使用する
4. THE EDI_Export_System SHALL CSVファイルの1行目にヘッダー行を出力する
5. THE EDI_Export_System SHALL 出力ファイル名を「PO_YYYYMMDD_HHMMSS.csv」形式で生成する
6. WHEN ファイル出力に失敗した、THE EDI_Export_System SHALL エラーメッセージを表示しエラーログに記録する
7. THE EDI_Export_System SHALL 出力先フォルダをExport_Configurationから取得する

### 要件4: データ検証

**ユーザーストーリー:** 品質管理者として、出力前にデータの妥当性を検証したい。これにより、不正なデータの送信を防止できる。

#### 受入基準（Acceptance Criteria）

1. WHEN CSV出力を実行する前、THE EDI_Export_System SHALL 発注データの必須項目の存在を検証する
2. THE EDI_Export_System SHALL 数値項目が数値として妥当であることを検証する
3. THE EDI_Export_System SHALL 日付項目が有効な日付であることを検証する
4. THE EDI_Export_System SHALL 品目コードがマスタに存在することを検証する
5. WHEN 検証エラーが発生した、THE EDI_Export_System SHALL エラー内容と該当レコードの情報を表示する
6. THE EDI_Export_System SHALL 検証エラーがある場合は出力を中止する

### 要件5: 出力履歴の記録

**ユーザーストーリー:** システム管理者として、CSV出力の履歴を記録したい。これにより、いつ誰がどのデータを出力したかを追跡できる。

#### 受入基準（Acceptance Criteria）

1. WHEN CSV出力が成功した、THE EDI_Export_System SHALL 出力日時、出力ユーザー、出力ファイル名、出力件数をAccess_Databaseに記録する
2. THE EDI_Export_System SHALL 出力された発注データに出力済みフラグを設定する
3. THE EDI_Export_System SHALL 出力履歴テーブルに一意の出力IDを生成して記録する
4. WHEN 履歴記録に失敗した、THE EDI_Export_System SHALL エラーメッセージを表示する

### 要件6: エラーハンドリングとログ記録

**ユーザーストーリー:** システム管理者として、エラー発生時の詳細情報を記録したい。これにより、問題の原因究明と対応が迅速に行える。

#### 受入基準（Acceptance Criteria）

1. WHEN データ抽出エラーが発生した、THE EDI_Export_System SHALL エラー内容をログファイルに記録する
2. WHEN データ変換エラーが発生した、THE EDI_Export_System SHALL エラー内容と該当データをログファイルに記録する
3. WHEN ファイル出力エラーが発生した、THE EDI_Export_System SHALL エラー内容とファイルパスをログファイルに記録する
4. THE EDI_Export_System SHALL ログファイルに日時、エラーレベル、エラーメッセージを含める
5. THE EDI_Export_System SHALL ログファイルを日付別に生成する
6. WHEN ログファイルへの書き込みに失敗した、THE EDI_Export_System SHALL ユーザーにエラーメッセージを表示する

### 要件7: ユーザーインターフェース

**ユーザーストーリー:** 発注担当者として、直感的な操作でCSV出力を実行したい。これにより、特別な技術知識なしに機能を利用できる。

#### 受入基準（Acceptance Criteria）

1. THE EDI_Export_System SHALL MS-Accessフォーム上にCSV出力ボタンを提供する
2. WHEN CSV出力を実行中、THE EDI_Export_System SHALL 進捗状況を表示する
3. WHEN CSV出力が完了した、THE EDI_Export_System SHALL 完了メッセージと出力件数を表示する
4. THE EDI_Export_System SHALL 出力先フォルダを開くボタンを提供する
5. THE EDI_Export_System SHALL 出力履歴を表示する機能を提供する
6. WHEN エラーが発生した、THE EDI_Export_System SHALL エラーメッセージをダイアログで表示する

### 要件8: 設定管理

**ユーザーストーリー:** システム管理者として、CSV出力の設定を管理したい。これにより、環境に応じた柔軟な運用が可能になる。

#### 受入基準（Acceptance Criteria）

1. THE EDI_Export_System SHALL Export_ConfigurationをAccess_Databaseのテーブルに保存する
2. THE EDI_Export_System SHALL 出力先フォルダパスをExport_Configurationから取得する
3. THE EDI_Export_System SHALL 文字コード設定（UTF-8）をExport_Configurationから取得する
4. THE EDI_Export_System SHALL ログファイル出力先をExport_Configurationから取得する
5. WHERE 管理者権限を持つユーザー、THE EDI_Export_System SHALL Export_Configurationを編集する機能を提供する
6. WHEN Export_Configurationが存在しない、THE EDI_Export_System SHALL デフォルト設定を使用する

### 要件9: サービスプロバイダー連携準備

**ユーザーストーリー:** システム管理者として、生成されたCSVファイルをサービスプロバイダーにアップロードする準備をしたい。これにより、サプライヤーへのデータ送信が可能になる。

#### 受入基準（Acceptance Criteria）

1. THE EDI_Export_System SHALL 出力されたCSVファイルを指定されたフォルダに配置する
2. THE EDI_Export_System SHALL ファイル名にタイムスタンプを含めることで一意性を確保する
3. THE EDI_Export_System SHALL 出力されたファイルが読み取り専用でないことを確認する
4. WHEN 同名のファイルが既に存在する、THE EDI_Export_System SHALL ファイル名に連番を付加する
5. THE EDI_Export_System SHALL 出力完了後にファイルパスをクリップボードにコピーする機能を提供する

### 要件10: データ整合性の保証

**ユーザーストーリー:** 品質管理者として、出力されたCSVデータが元のデータベースと整合していることを確認したい。これにより、データの正確性が保証される。

#### 受入基準（Acceptance Criteria）

1. THE EDI_Export_System SHALL 出力前のレコード件数と出力後のCSV行数が一致することを検証する
2. THE EDI_Export_System SHALL 出力されたCSVファイルの各行が正しい項目数を持つことを検証する
3. WHEN 整合性検証に失敗した、THE EDI_Export_System SHALL エラーメッセージを表示し出力を無効とする
4. THE EDI_Export_System SHALL 検証結果をログファイルに記録する
5. FOR ALL 出力された発注データ、出力前のデータを読み込み、CSV出力し、再度読み込んだ場合に同等のデータ構造が得られる（ラウンドトリップ特性）

