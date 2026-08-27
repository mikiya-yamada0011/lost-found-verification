k

# 落とし物システム　技術検証結果と本番構成の相談

## 今回の目的

技術検証の結果を共有し、学生向けWeb画面からSharePointへ接続する方式と、データの保存先についてを確認する。

# 1. 動作確認できたこと

## 1.1 学生向けWebアプリとSharePointの接続

検証環境（tsSTU2-istc）では、次の構成が動作した。

```text
学生
  ↑
React＋Viteの一覧画面にリストを表示
  ↑
Power Automate：HTTP要求の受信時 → 複数の項目の取得 → 応答
  ↑
SharePoint：リスト（FoundItems）
```

## 1.2 Power AppsとSharePointの接続

検証環境（tsSTU2-istc）では、Power AppsからSharePointの`FoundItems`リストへ直接接続し、SharePointに登録されている拾得物をPower Appsの一覧画面に表示できることを確認した。

```text
職員
  ↓
Power Apps：拾得物の一覧を表示
  ↓
SharePoint：リスト（FoundItems）
```

これにより、職員向け画面はPower Apps、データの保存先はSharePointとする構成が技術的に接続できることを確認した。

### 動作確認動画

[apps.powerapps.com/play/e/4673518f-05bc-e242-9104-50370b6e23b6/a/e117c001-4c34-45dc-bc89-ef8cdd6daed9?tenantId=20ee4c80-87bd-422c-a506-3a2b0aca0615&amp;hint=8941a2d8-d6b7-4fea-a29d-5d71a6492576&amp;sourcetime=1787857549450](https://apps.powerapps.com/play/e/4673518f-05bc-e242-9104-50370b6e23b6/a/e117c001-4c34-45dc-bc89-ef8cdd6daed9?tenantId=20ee4c80-87bd-422c-a506-3a2b0aca0615&hint=8941a2d8-d6b7-4fea-a29d-5d71a6492576&sourcetime=1787857549450)

<video controls width="100%">
  <source src="./レコーディング 2026-08-24 003442.mp4" type="video/mp4">
  この環境では動画を再生できません。
</video>

## 1.3 Entra ID管理画面へのアクセス状況

検証用アカウントで、次のEntra ID管理画面へアクセスを試したが、現在もアクセスできない。

- [Microsoft Entra管理センター](https://entra.microsoft.com/)

![1787503074717](image/verification/1787503074717.png)

今回追加された「環境作成者」と「システムカスタマイザー」は、Power Platform環境内でアプリやフロー、Dataverse等を扱うための権限である。Entra ID管理画面へのアクセスやアプリ登録を行う権限とは別のため、これらの追加後もEntra IDへのアクセス状況は変わっていない。

学生向け画面の大学アカウント認証や、独自APIからMicrosoft Graphを経由してSharePointへ接続する検証には、大学側でのEntra IDアプリ登録またはEntra ID側の権限設定が別途必要になる。

# 2. 学生向け画面（React）からSharePointの拾得物情報を取得する方法

## 2.1 Power Automateを経由して取得する

現在のHTTP要求トリガーはプレミアム機能であり、指定環境ではProcessライセンスを契約していない。多数の学生が利用する本番構成でこの方式を採用する場合、Processライセンスを1つ新規購入する案が現実的であり、公開価格は**年間約27万円（税別）**となる。

検証環境でHTTP要求トリガーは動作したが、契約がないまま動作する理由は確認できていない。動作したことと本番で利用できることは別のため、現在の構成は検証用とする。

[Microsoft Learn：サイトスクリプトからのPower Automateの呼び出し](https://learn.microsoft.com/ja-jp/sharepoint/dev/declarative-customization/site-design-trigger-flow-tutorial#%E3%83%95%E3%83%AD%E3%83%BC%E3%82%92%E4%BD%9C%E6%88%90%E3%81%99%E3%82%8B)

上記のurl内に「**注** : **要求**トリガーは**プレミアム**になったので、追加のライセンスが必要になります。」と記載がある。

[Microsoft Learn：どのPower Automateライセンスが必要ですか](https://learn.microsoft.com/ja-jp/power-platform/admin/power-automate-licensing/faqs#%E3%81%A9%E3%81%AE-power-automate-%E3%83%A9%E3%82%A4%E3%82%BB%E3%83%B3%E3%82%B9%E3%81%8C%E5%BF%85%E8%A6%81%E3%81%A7%E3%81%99%E3%81%8B)

上記のurl内に「プレミアムフローは複数のユーザーが呼び出します。 この場合、全員にプレミアムライセンスが必要か、フローにプロセスライセンスが必要です。」と記載がある。

[Microsoft公式のPower Automate価格ページ](https://www.microsoft.com/ja-jp/power-platform/products/power-automate/pricing#tabs-pill-bar-ocd242_tab0)

上記のurlでProcessプランが年額約27万(税別)かかることが確認できる。

この3つのurlの情報からHTTP要求トリガーには最低でも年額約27万かかることがわかる。

## 2.2 独自APIとMicrosoft Graphを経由して取得する

年間約27万円かかるPower Automate Process方式以外では、独自APIからMicrosoft Graphへ接続し、SharePointから情報を取得する方式が考えられる。

Microsoft Graphの通常のSharePointリスト操作には追加のAPI料金はかからないが、独自APIのコードを書く必要がある。

また、学生向けアプリの利用者認証とは別に、SharePointへ接続するためのEntra IDアプリ登録が必要になる。

# 3. データの保存先

現時点の提案として、Dataverseではなく、保存先の第一候補をSharePointとする。

理由は次のとおり。

- 職員がPower AppsからDataverseへ直接接続する構成では、原則として、利用する職員ごとにPower Apps Premium等のDataverse利用権が必要になり、職員数に応じて費用が増える。
- ProcessライセンスではDataverse接続の利用権を利用者ではなくフローに持たせられるが、職員・学生の全操作をそのフロー経由にする必要があり、SharePoint案よりフロー数と保守範囲が増える。

そのため、現時点ではDataverseの利点よりライセンスと構成の負担が大きいと判断する。

# 4. 今回確認したいこと

1. Processライセンスを新規購入するか、独自API方式で進めるか
2. 独自API方式の場合、認証とは別に外部からsharepointへアクセスを許可する認可が必要になるが、大学側にEntra IDのアプリ登録を依頼できるか
