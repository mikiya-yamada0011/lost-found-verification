# 落とし物システム（SharePoint・SPFx版）検証結果と大学側への依頼

# 1. 現在までに確認できたこと

学生向け画面と職員向け画面を作成し、基本動作を確認した。Power Apps・Power Automateの検証環境からSharePointへ接続できることも確認している。

学生向け画面はReactで作成し、SPFxを使ってSharePointのページ上に配置する構成とした。SharePoint上で動作するため、サインイン中のMicrosoft 365アカウントをそのまま利用でき、このシステム専用のログインは必要ない。現在は、対象SharePointサイトの開発用検証画面でローカルのReactコードを読み込んで動作を確認しており、学生向け画面の本配置はまだ行っていない。

大学の学生アカウントでは、Microsoft 365へサインインできても対象SharePointサイトを利用する権限がないため、学生アカウントでの動作確認はまだできていない。

* SPFxで実装した学生向け画面の動作確認動画

<video controls width="100%">
  <source src="./レコーディング 2026-09-11 061453.mp4" type="video/mp4">
  この環境では動画を再生できません。
</video>

<!--
動画挿入位置：学生の検索・登録から、職員の確認・返却までのデモ
例：
<video controls width="100%">
  <source src="./学生・職員デモ.mp4" type="video/mp4">
  この環境では動画を再生できません。
</video>
-->

# 2. 現在止まっていること

現在の権限では、次の作業を実施できない。

- SharePointの各情報について、学生用・職員用・申告した本人用の閲覧範囲を設定すること
- SPFxパッケージを対象サイトのアプリカタログへ登録し、学生向け画面をSharePointページに常設すること
- 実際の学生・職員アカウントを使って、検索・申告から返却・メール送信までを確認すること

このため、以下の2点について大学側の対応をお願いしたい。

# 3. 大学側へお願いしたいこと

| お願いしたいこと                                                 | これによってできるようになること                                                                                   |
| ---------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| 検証用アカウントを対象サイトのサイトコレクション管理者に設定する | 学生用・職員用グループを作成し、各SharePointリストについて学生・職員・申告した本人の閲覧範囲を設定できるようになる |
| 対象サイトのサイトコレクションアプリカタログを有効化する         | 作成済みの学生向け画面を対象サイトへインストールし、SharePointのページに追加できるようになる                       |

## 3.1 検証用アカウントをサイトコレクション管理者に設定する

大学のSharePoint管理者が、SharePoint管理センターで次の操作を行う。

1. [SharePoint管理センター](https://cloudkobeu-admin.sharepoint.com)を開く
2. 「アクティブなサイト」を開く
3. [対象サイト](https://cloudkobeu.sharepoint.com/teams/t-dxsuisin-tesvd2)を選択する
4. 「メンバーシップ」を開く
5. 「サイト管理者」または「追加の管理者」に検証用アカウントを追加する
6. 保存する

画面表記は環境によって「アクセス許可」から「サイト管理者の管理」を開く場合もある。詳細は[Microsoft公式手順](https://learn.microsoft.com/en-us/sharepoint/manage-site-collection-administrators)を参照する。

## 3.2 サイトコレクションアプリカタログを有効化する

通常の管理画面には有効化ボタンがないため、大学のSharePoint管理者がSharePoint Online Management Shellで次のコマンドを実行する。

```powershell
Connect-SPOService -Url https://cloudkobeu-admin.sharepoint.com
Add-SPOSiteCollectionAppCatalog -Site https://cloudkobeu.sharepoint.com/teams/t-dxsuisin-tesvd2
```

実行後、対象サイトの「サイトコンテンツ」に「Apps for SharePoint」ライブラリが作成され、検証用アカウントがSPFxパッケージ（`.sppkg`）を登録できるようになる。詳細は[サイトコレクションアプリカタログの公式手順](https://learn.microsoft.com/en-us/sharepoint/dev/general-development/site-collection-app-catalog)および[`Connect-SPOService`の公式リファレンス](https://learn.microsoft.com/en-us/powershell/module/microsoft.online.sharepoint.powershell/connect-sposervice?view=sharepoint-ps)を参照する。

以上の設定後、SPFxパッケージの登録、SharePointページへの追加、以後の更新はこちらで行う。
