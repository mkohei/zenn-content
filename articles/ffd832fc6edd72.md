---
title: "Terraform で Google Identity Platform マルチテナントのユーザーサインアップと削除を制御する方法"
emoji: "🧱"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["googlecloud", "identityplatform", "terraform"]
published: true
---

Google Identity Platform では、プロジェクト単位でユーザーのサインアップやアカウント削除を制限できます。しかし、マルチテナント環境で複数のテナントを運用する場合、プロジェクトレベルの設定がテナントには反映されず、テナント側でユーザー登録や削除が許可されてしまうという問題があります。さらに、Google Cloud Console や Terraform の `google_identity_platform_tenant` には、テナント単位でサインアップやアカウント削除を制御する設定項目がありません。

本記事では、この課題と回避策、そして将来に期待される機能追加について解説します。

## 課題

1. **プロジェクト単位の制限は可能**
   - プロジェクト単位でユーザーのサインアップやアカウント削除を禁止・許可する機能は提供されています。

2. **マルチテナント運用時の問題**
   - プロジェクト単位で設定した制限がテナントに反映されず、テナントでは依然としてユーザー登録や削除が行えてしまいます。

3. **テナント単位の設定項目が未実装**
   - Google Cloud Console 上でも、Terraform の `google_identity_platform_tenant` リソースでも、テナント単位でのサインアップやアカウント削除を制御するパラメータが存在しないのが現状です。

## 回避策: Google Identity Platform API の直接呼び出し

API ではテナント単位の制御が可能です。以下のエンドポイントを呼び出してテナントごとのユーザー登録や削除の可否を設定できます。

- [projects.tenants.get](https://cloud.google.com/identity-platform/docs/reference/rest/v2/projects.tenants/get)  
- [projects.tenants.patch](https://cloud.google.com/identity-platform/docs/reference/rest/v2/projects.tenants/patch)

これらのエンドポイントでは、「Try this method」というボタンからその場で認証済みのリクエストをテストできます。運用で繰り返し適用する場合は、gcloud CLI やサービスアカウントを使ったスクリプトなどで自動化すると効率的です。

## 今後の展望

Terraform でテナントごとの設定を完結させるには、公式プロバイダーへの機能追加が必要です。すでにテナント単位の設定項目追加を求める Issue が作成しており、将来的に `google_identity_platform_tenant` リソースでサインアップやアカウント削除を制御できるようになる可能性があります。

- [Add support for user signup/deletion settings in google_identity_platform_tenant](https://github.com/hashicorp/terraform-provider-google/issues/20918)

## まとめ

- **マルチテナント環境の課題**  
  - プロジェクト単位で設定した禁止設定が、テナントに反映されない。  
  - テナント単位の設定項目がコンソールや Terraform に備わっていない。

- **回避策**  
  - Google Identity Platform API で `projects.tenants.patch` などを直接呼び出し、テナントごとのサインアップ／削除制限を適用する。

- **将来への期待**  
  - Terraform プロバイダーに機能が追加されれば、コンソールや Terraform リソースからテナント単位の設定を直接管理できるようになる。

マルチテナント環境でテナントごとに異なるユーザー管理ポリシーが必要なケースでは、当面は API の直接呼び出しという回避策が必要です。公式サポートが追加されれば、より一貫した方法で一元管理できるようになるでしょう。
