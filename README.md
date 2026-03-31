# custom-defender-for-sql-resource-audit

Azure Policy カスタム定義 — 各 Azure SQL Server および Azure SQL Managed Instance に対してリソース単位の **Defender for SQL** が有効化されていることを監査します。

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fhisashin0728%2FCheck-DefenderForSQL--ResourceLevel%2Frefs%2Fheads%2Fmain%2Fazuredeploy.json)

---

## 概要

本リポジトリには **4 つのカスタムポリシー定義** が含まれます。

### ポリシー 1：Azure SQL Database 用

| 項目 | 値 |
|---|---|
| ポリシー名 | `custom-defender-for-sql-db-resource-audit` |
| 表示名 | `[Custom] Defender for SQL (Azure SQL Database) - Resource Level Audit` |
| カテゴリ | Security Center |
| バージョン | 1.0.0 |
| モード | All |
| Effect | AuditIfNotExists（デフォルト） / Disabled |
| 対象リソース | `Microsoft.Sql/servers`（`kind` に `analytics` を含まないもの） |
| チェック対象 | `Microsoft.Sql/servers/advancedThreatProtectionSettings` |

### ポリシー 2：Azure SQL Managed Instance 用

| 項目 | 値 |
|---|---|
| ポリシー名 | `custom-defender-for-sql-mi-resource-audit` |
| 表示名 | `[Custom] Defender for SQL (Azure SQL Managed Instance) - Resource Level Audit` |
| カテゴリ | Security Center |
| バージョン | 1.0.0 |
| モード | All |
| Effect | AuditIfNotExists（デフォルト） / Disabled |
| 対象リソース | `Microsoft.Sql/managedInstances` |
| チェック対象 | `Microsoft.Sql/managedInstances/advancedThreatProtectionSettings` |

### ポリシー 3：Synapse Dedicated SQL Pool 用

| 項目 | 値 |
|---|---|
| ポリシー名 | `custom-defender-for-synapse-ded-audit` |
| 表示名 | `[Custom] Defender for SQL (Synapse Dedicated SQL Pool) - Resource Level Audit` |
| カテゴリ | Security Center |
| バージョン | 1.0.0 |
| モード | All |
| Effect | AuditIfNotExists（デフォルト） / Disabled |
| 対象リソース | `Microsoft.Sql/servers`（`kind` に `analytics` を含むもの） |
| チェック対象 | `Microsoft.Sql/servers/advancedThreatProtectionSettings` |

### ポリシー 4：Synapse Workspace 用

| 項目 | 値 |
|---|---|
| ポリシー名 | `custom-defender-for-synapse-ws-audit` |
| 表示名 | `[Custom] Defender for SQL (Synapse Workspace) - Resource Level Audit` |
| カテゴリ | Security Center |
| バージョン | 1.0.0 |
| モード | All |
| Effect | AuditIfNotExists（デフォルト） / Disabled |
| 対象リソース | `Microsoft.Synapse/workspaces` |
| チェック対象 | `Microsoft.Synapse/workspaces/securityAlertPolicies` |

---

## 動作

```
【Azure SQL Database】
Microsoft.Sql/servers が存在する（Synapse Analytics を除く）
  └─ advancedThreatProtectionSettings/Default の state == Enabled か？
       ├─ YES → Compliant
       └─ NO  → NonCompliant（AuditIfNotExists）

【Azure SQL Managed Instance】
Microsoft.Sql/managedInstances が存在する
  └─ advancedThreatProtectionSettings/Default の state == Enabled か？
       ├─ YES → Compliant
       └─ NO  → NonCompliant（AuditIfNotExists）

【Synapse Dedicated SQL Pool】
Microsoft.Sql/servers（kind に analytics を含む）が存在する
  └─ advancedThreatProtectionSettings/Default の state == Enabled か？
       ├─ YES → Compliant
       └─ NO  → NonCompliant（AuditIfNotExists）

【Synapse Workspace】
Microsoft.Synapse/workspaces が存在する
  └─ securityAlertPolicies/Default の state == Enabled か？
       ├─ YES → Compliant
       └─ NO  → NonCompliant（AuditIfNotExists）
```

各リソースに対して、リソース単位の Defender for SQL（`state: Enabled`）が設定されているかを評価します。  
`evaluationDelay: AfterProvisioning` により、リソースプロビジョニング完了後に評価されます。

> **注意:** このポリシーはリソース単位の設定のみを評価します。サブスクリプションレベルの Defender for SQL プラン（`Microsoft.Security/pricings/SqlServers`）は評価対象外です。サブスクリプションレベルのチェックには [Azure Defender for SQL should be enabled for unprotected Azure SQL servers](#関連ポリシー)（ビルトイン）を併用してください。

---

## パラメーター

| パラメーター | 型 | 許容値 | デフォルト | 説明 |
|---|---|---|---|---|
| `effect` | String | `AuditIfNotExists`, `Disabled` | `AuditIfNotExists` | ポリシーの効果。`Disabled` にするとポリシーが無効化されます。 |

---

## デプロイ方法

### ARM テンプレートによるデプロイ（推奨）

#### Deploy to Azure ボタン

以下のリンクをブラウザで開くと、Azure Portal から直接デプロイできます。

```
https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fhisashin0728%2FCheck-DefenderForSQL--ResourceLevel%2Frefs%2Fheads%2Fmain%2Fazuredeploy.json
```

#### Azure CLI によるデプロイ

```powershell
$mgId = "<Tenant Root Group ID>"  # 例: 464ded72-1950-4150-8811-b47ceb718344

az deployment mg create `
  --name "deploy-defender-sql-policy" `
  --management-group-id $mgId `
  --location japaneast `
  --template-file azuredeploy.json `
  --parameters azuredeploy.parameters.json
```

#### パラメーター

| パラメーター | 型 | デフォルト | 説明 |
|---|---|---|---|
| `policyEffect` | string | `AuditIfNotExists` | ポリシーの Effect（`AuditIfNotExists` / `Disabled`）。4 ポリシー共通。 |
| `assignPolicy` | bool | `true` | `true` にすると、定義作成と同時に管理グループへの割り当ても実施 |

#### ファイル構成

```
azuredeploy.json              # ARM テンプレート本体（4 ポリシー定義 + 4 割り当て）
azuredeploy.parameters.json   # パラメーターファイル
```

---

### Azure CLI（JSON 直接指定）によるデプロイ

#### 前提条件

- Azure CLI がインストール済みであること
- ポリシー定義を作成する権限（`Microsoft.Authorization/policyDefinitions/write`）があること
- 管理グループ または サブスクリプションへのアクセス権があること

#### Tenant Root Group（推奨）

```powershell
$mgId = "<Tenant Root Group ID>"

# ─── Azure SQL Database 用 ───
$policy = Get-Content "combined-policy.json" -Raw | ConvertFrom-Json
$policy.properties.policyRule | ConvertTo-Json -Depth 10 | Out-File "$env:TEMP\policy-rule-db.json" -Encoding utf8
$policy.properties.parameters | ConvertTo-Json -Depth 10 | Out-File "$env:TEMP\policy-params-db.json" -Encoding utf8

az policy definition create `
  --name "custom-defender-for-sql-db-resource-audit" `
  --rules "$env:TEMP\policy-rule-db.json" `
  --params "$env:TEMP\policy-params-db.json" `
  --mode All `
  --display-name "[Custom] Defender for SQL (Azure SQL Database) - Resource Level Audit" `
  --management-group $mgId

az policy assignment create `
  --name "dfsql-db-resource-audit" `
  --display-name "[Custom] Defender for SQL (Azure SQL Database) - Resource Level Audit" `
  --policy "/providers/Microsoft.Management/managementGroups/$mgId/providers/Microsoft.Authorization/policyDefinitions/custom-defender-for-sql-db-resource-audit" `
  --scope "/providers/Microsoft.Management/managementGroups/$mgId" `
  --enforcement-mode Default

# ─── Azure SQL Managed Instance 用 ───
$policyMi = Get-Content "combined-policy-mi.json" -Raw | ConvertFrom-Json
$policyMi.properties.policyRule | ConvertTo-Json -Depth 10 | Out-File "$env:TEMP\policy-rule-mi.json" -Encoding utf8
$policyMi.properties.parameters | ConvertTo-Json -Depth 10 | Out-File "$env:TEMP\policy-params-mi.json" -Encoding utf8

az policy definition create `
  --name "custom-defender-for-sql-mi-resource-audit" `
  --rules "$env:TEMP\policy-rule-mi.json" `
  --params "$env:TEMP\policy-params-mi.json" `
  --mode All `
  --display-name "[Custom] Defender for SQL (Azure SQL Managed Instance) - Resource Level Audit" `
  --management-group $mgId

az policy assignment create `
  --name "dfsql-mi-resource-audit" `
  --display-name "[Custom] Defender for SQL (Azure SQL Managed Instance) - Resource Level Audit" `
  --policy "/providers/Microsoft.Management/managementGroups/$mgId/providers/Microsoft.Authorization/policyDefinitions/custom-defender-for-sql-mi-resource-audit" `
  --scope "/providers/Microsoft.Management/managementGroups/$mgId" `
  --enforcement-mode Default

# ─── Synapse Dedicated SQL Pool 用 ───
$policySynDed = Get-Content "combined-policy-synapse-dedicated.json" -Raw | ConvertFrom-Json
$policySynDed.properties.policyRule | ConvertTo-Json -Depth 10 | Out-File "$env:TEMP\policy-rule-syn-ded.json" -Encoding utf8
$policySynDed.properties.parameters | ConvertTo-Json -Depth 10 | Out-File "$env:TEMP\policy-params-syn-ded.json" -Encoding utf8

az policy definition create `
  --name "custom-defender-for-synapse-ded-audit" `
  --rules "$env:TEMP\policy-rule-syn-ded.json" `
  --params "$env:TEMP\policy-params-syn-ded.json" `
  --mode All `
  --display-name "[Custom] Defender for SQL (Synapse Dedicated SQL Pool) - Resource Level Audit" `
  --management-group $mgId

az policy assignment create `
  --name "dfsql-syn-ded-audit" `
  --display-name "[Custom] Defender for SQL (Synapse Dedicated SQL Pool) - Resource Level Audit" `
  --policy "/providers/Microsoft.Management/managementGroups/$mgId/providers/Microsoft.Authorization/policyDefinitions/custom-defender-for-synapse-ded-audit" `
  --scope "/providers/Microsoft.Management/managementGroups/$mgId" `
  --enforcement-mode Default

# ─── Synapse Workspace 用 ───
$policySynWs = Get-Content "combined-policy-synapse-workspace.json" -Raw | ConvertFrom-Json
$policySynWs.properties.policyRule | ConvertTo-Json -Depth 10 | Out-File "$env:TEMP\policy-rule-syn-ws.json" -Encoding utf8
$policySynWs.properties.parameters | ConvertTo-Json -Depth 10 | Out-File "$env:TEMP\policy-params-syn-ws.json" -Encoding utf8

az policy definition create `
  --name "custom-defender-for-synapse-ws-audit" `
  --rules "$env:TEMP\policy-rule-syn-ws.json" `
  --params "$env:TEMP\policy-params-syn-ws.json" `
  --mode All `
  --display-name "[Custom] Defender for SQL (Synapse Workspace) - Resource Level Audit" `
  --management-group $mgId

az policy assignment create `
  --name "dfsql-syn-ws-audit" `
  --display-name "[Custom] Defender for SQL (Synapse Workspace) - Resource Level Audit" `
  --policy "/providers/Microsoft.Management/managementGroups/$mgId/providers/Microsoft.Authorization/policyDefinitions/custom-defender-for-synapse-ws-audit" `
  --scope "/providers/Microsoft.Management/managementGroups/$mgId" `
  --enforcement-mode Default
```

---

## コンプライアンス評価

### オンデマンドスキャン（即時評価）

通常の評価サイクルは約 24 時間ですが、以下のコマンドで即時評価をトリガーできます。

```powershell
# スキャントリガー（同期：完了まで待機）
az policy state trigger-scan --subscription <Subscription ID>

# SQL Database の結果確認
az policy state list `
  --subscription <Subscription ID> `
  --filter "policyAssignmentName eq 'dfsql-db-resource-audit'" `
  --query "[].{resource:resourceId, compliant:complianceState}" `
  -o table

# SQL Managed Instance の結果確認
az policy state list `
  --subscription <Subscription ID> `
  --filter "policyAssignmentName eq 'dfsql-mi-resource-audit'" `
  --query "[].{resource:resourceId, compliant:complianceState}" `
  -o table

# Synapse Dedicated SQL Pool の結果確認
az policy state list `
  --subscription <Subscription ID> `
  --filter "policyAssignmentName eq 'dfsql-syn-ded-audit'" `
  --query "[].{resource:resourceId, compliant:complianceState}" `
  -o table

# Synapse Workspace の結果確認
az policy state list `
  --subscription <Subscription ID> `
  --filter "policyAssignmentName eq 'dfsql-syn-ws-audit'" `
  --query "[].{resource:resourceId, compliant:complianceState}" `
  -o table
```

### NonCompliant リソースへの対応

NonCompliant と表示されたリソースは、以下のいずれかで対応します。

**Azure Portal の場合:**  
対象 SQL Server または SQL Managed Instance → [Microsoft Defender for Cloud] → [Microsoft Defender for SQL] を有効化

**Azure REST API の場合（SQL Database）:**
```powershell
az rest --method put `
  --url "https://management.azure.com/subscriptions/<Sub>/resourceGroups/<RG>/providers/Microsoft.Sql/servers/<ServerName>/advancedThreatProtectionSettings/Default?api-version=2023-08-01" `
  --body '{"properties":{"state":"Enabled"}}'
```

**Azure REST API の場合（SQL Managed Instance）:**
```powershell
az rest --method put `
  --url "https://management.azure.com/subscriptions/<Sub>/resourceGroups/<RG>/providers/Microsoft.Sql/managedInstances/<MiName>/advancedThreatProtectionSettings/Default?api-version=2023-08-01" `
  --body '{"properties":{"state":"Enabled"}}'
```

---

## サブスクリプション単位の Defender for SQL と組み合わせる場合

Defender for SQL はサブスクリプションレベルで一括有効化（`Microsoft.Security/pricings/SqlServers` の `pricingTier: Standard`）することもできます。  
その場合、リソース単位では `advancedThreatProtectionSettings/state` が `New`（未設定）のまま実質的に保護されている SQL Server が存在します。

> **本ポリシー単独ではそのリソースも NonCompliant と判定されます**（リソース単位の `state: Enabled` のみを評価するため）。

### 推奨アプローチ：Initiative（イニシアティブ）による組み合わせ

「サブスクリプション単位 **OR** リソース単位のどちらかで有効であれば Compliant」とみなしたい場合は、以下の 3 つのポリシーを **Policy Initiative（イニシアティブ）** にまとめることを推奨します。

| # | 種別 | ポリシー | チェック内容 |
|---|---|---|---|
| 1 | ビルトイン | `Azure Defender for SQL should be enabled for unprotected Azure SQL servers`<br>（ID: `abfb4388-5bf4-4ad7-ba82-2cd2f41ceae9`） | サブスクリプション単位（`securityAlertPolicies/state == Enabled`） |
| 2 | カスタム | `custom-defender-for-sql-db-resource-audit` | リソース単位 SQL DB（`advancedThreatProtectionSettings/state == Enabled`） |
| 3 | カスタム | `custom-defender-for-sql-mi-resource-audit` | リソース単位 SQL MI（`advancedThreatProtectionSettings/state == Enabled`） |
| 4 | カスタム | `custom-defender-for-synapse-ded-audit` | リソース単位 Synapse Dedicated SQL Pool（`advancedThreatProtectionSettings/state == Enabled`） |
| 5 | カスタム | `custom-defender-for-synapse-ws-audit` | リソース単位 Synapse Workspace（`securityAlertPolicies/state == Enabled`） |

#### コンプライアンス状態の解釈

```
サブスク単位（#1）   SQL DB（#2）    SQL MI（#3）    Syn Ded（#4）   Syn WS（#5）    解釈
──────────────────────────────────────────────────────────────────────────────────────────
Compliant           Compliant       Compliant       Compliant       Compliant       ✅ 全レベルで有効
Compliant           NonCompliant    -               NonCompliant    -               ✅ サブスク有効（実質保護中）
NonCompliant        Compliant       Compliant       Compliant       Compliant       ✅ リソース単位で個別有効化済み
NonCompliant        NonCompliant    NonCompliant    NonCompliant    NonCompliant    ❌ どのレベルも未設定 → 要対応
```

> **補足:** Azure Policy は「イニシアティブ内の複数ポリシーの AND / OR」を直接表現できません。上記の解釈は各ポリシーの結果を**手動で突き合わせる**運用を前提としています。

---

## 関連ポリシー

| ポリシー名 | チェック内容 | スコープ |
|---|---|---|
| `custom-defender-for-sql-db-resource-audit`（本ポリシー） | リソース単位 `state: Enabled` | 各 Azure SQL Server |
| `custom-defender-for-sql-mi-resource-audit`（本ポリシー） | リソース単位 `state: Enabled` | 各 Azure SQL Managed Instance |
| `custom-defender-for-synapse-ded-audit`（本ポリシー） | リソース単位 `state: Enabled` | Synapse Dedicated SQL Pool |
| `custom-defender-for-synapse-ws-audit`（本ポリシー） | リソース単位 `state: Enabled` | Synapse Workspace |
| `Azure Defender for SQL should be enabled for unprotected Azure SQL servers`（ビルトイン） | サブスクリプション単位の Defender for SQL 有効化 | サブスクリプション |
| `Azure Defender for SQL should be enabled for unprotected SQL Managed Instances`（ビルトイン） | サブスクリプション単位の Defender for SQL MI 有効化 | サブスクリプション |
| `Microsoft Defender for SQL should be enabled for unprotected Synapse workspaces`（ビルトイン, ID: `d31e5c31-63b2-4f12-887b-e49456834fa1`） | サブスクリプション単位の Synapse Workspace 有効化 | サブスクリプション |

---

## ファイル構成

```
.
├── azuredeploy.json                          # ARM テンプレート（4 ポリシー定義 + 4 割り当て）
├── azuredeploy.parameters.json               # ARM テンプレート パラメーターファイル
├── combined-policy.json                      # Azure SQL Database 用ポリシー定義（CLI デプロイ用）
├── combined-policy-mi.json                   # Azure SQL Managed Instance 用ポリシー定義（CLI デプロイ用）
├── combined-policy-synapse-dedicated.json    # Synapse Dedicated SQL Pool 用ポリシー定義（CLI デプロイ用）
└── combined-policy-synapse-workspace.json    # Synapse Workspace 用ポリシー定義（CLI デプロイ用）
```

---

## 参考リンク

- [Microsoft Defender for SQL とは](https://learn.microsoft.com/ja-jp/azure/azure-sql/database/azure-defender-for-sql)
- [Microsoft.Sql/servers/advancedThreatProtectionSettings - ARM テンプレートリファレンス](https://learn.microsoft.com/ja-jp/azure/templates/microsoft.sql/servers/advancedthreatprotectionsettings)
- [Microsoft.Sql/managedInstances/advancedThreatProtectionSettings - ARM テンプレートリファレンス](https://learn.microsoft.com/ja-jp/azure/templates/microsoft.sql/managedinstances/advancedthreatprotectionsettings)
- [Azure Policy の定義の構造](https://learn.microsoft.com/ja-jp/azure/governance/policy/concepts/definition-structure)
- [AuditIfNotExists の効果](https://learn.microsoft.com/ja-jp/azure/governance/policy/concepts/effects#auditifnotexists)
