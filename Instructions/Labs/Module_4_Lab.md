---
lab:
    title: '4: Azure AD の認証と承認の管理'
    module: 'モジュール 4: 認証と承認の設計'
---

# ラボ: Azure AD の認証と承認の管理
# 受講生用ラボ マニュアル

## ラボ シナリオ

Azure への移行の一環として、Adatum Corporation は ID 戦略を定義する必要があります。Adatum には、adatum.com という名前の単一ドメインの Active Directory フォレストがあり、対応するパブリックに登録された DNS ドメインを所有しています。Adatum Enterprise Architecture チームは、一部のオンプレミス ワークロードを Azure に移行するオプションを模索しているため、Active Directory Domain Services (AD DS) 環境と、ターゲットの Azure サブスクリプションに関連付けられた Azure Active Directory (Azure AD) テナントとの統合を、長期的な認証および承認モデルのコア コンポーネントとして評価する予定です。

新しいモデルは、Azure AD の多要素認証機能を活用するアプリケーションごとのステップアップ認証とともに、シングル サインオンを容易にする必要があります。シングル サインオンを実装するために、Architecture チームは Azure AD Connect をデプロイし、パスワード ハッシュ同期用に構成することを計画しています。その結果、両方の ID ストアでユーザー オブジェクトが一致します。最適な認証方法を選択することは、クラウドへの移行を望む組織にとって最初の懸念事項です。Azure AD のパスワード ハッシュ同期は、Azure AD 統合リソースにアクセスするときに、オンプレミス ユーザーのシングル サインオン認証を実装する最も簡単な方法です。この方法は、Identity Protection などの一部のプレミアム Azure AD 機能でも必要です。

ステップアップ認証を実装するために、Adatum Enterprise Architecture チームは、Azure AD 条件付きアクセス ポリシーを利用する予定です。条件付きアクセス ポリシーは、アクセスされるアプリケーションまたはリソースのタイプに応じて、多要素認証の実施をサポートします。条件付きアクセスポリシーは、第一要素認証が完了した後に適用されます。条件付きアクセスは、次のような幅広い範囲に基づいています。

- ユーザーまたはグループのメンバーシップ。ポリシーは特定のユーザーおよびグループを対象とすることができ、管理者はアクセスをきめ細かく制御できます。
- IP ロケーション情報。組織は、ポリシーを決定するときに使用できる信頼できる IP アドレス範囲を作成できます。管理者は、国または地域全体の IP 範囲を指定して、トラフィックをブロックまたは許可できます。
- デバイス。特定のプラットフォームのデバイスを使用するユーザー、または特定の状態でマークされたユーザーは、条件付きアクセス ポリシーを適用するときに使用できます。
- アプリケーション。特定のアプリケーションにアクセスしようとするユーザーは、さまざまな条件付きアクセス ポリシーをトリガーできます。
- リアルタイムおよび計算されたリスク検出。シグナルと Azure AD Identity Protection の統合により、条件付きアクセス ポリシーで危険なサインイン動作を特定できます。ポリシーにより、ユーザーにパスワードの変更や多要素認証を強制的に実行させて、リスク レベルを下げたり、管理者が手動で操作するまでアクセスをブロックしたりできます。
- Microsoft Cloud App Security (MCAS)。ユーザー アプリケーションのアクセスとセッションをリアルタイムで監視および制御できるため、クラウド環境内で実行されるアクセスとアクティビティに対する可視性と制御が向上します。

これらの目標を達成するために、Adatum Enterprise Architecture チームは、Active Directory Domain Services (AD DS) フォレストと Azure Active Directory (Azure AD) テナントの統合をテストし、パイロット ユーザーの条件付きアクセス機能を評価する予定です。

## 目標
  
このラボを終了すると、下記ができるようになります。

 - AD DS ドメイン コントローラーをホストする Azure VM をデプロイする

 - Azure AD ユーザーを作成して構成する

 - AD DS フォレストを Azure AD テナントに統合する


## ラボ環境
  
Windows Server 管理者の認証資格情報

-  ユーザー名: **Student**

-  パスワード: **Pa55w.rd1234**

予想時間: 120 分


## ラボ ファイル

-  \\\\AZ303\\AllFiles\\Labs\\10\\azuredeploy30310suba.json


## 指示

### 演習 0: ラボ環境の準備

この演習の主なタスクは次のとおりです。

1. Azure VM デプロイメントで使用可能な DNS 名を特定する

1. Azure Resource Manager QuickStart テンプレートを使用して、AD DS ドメイン管理者を実行する Azure VM をデプロイする


#### タスク 1: Azure VM デプロイメントで使用可能な DNS 名を特定する

1. ラボのコンピューターから Web ブラウザーを起動し、 [Azure portal](https://portal.azure.com) に移動し、このラボで使用するサブスクリプションの所有者の役割を持つユーザーアカウントの認証情報を提供してサインインします。

1. Azure で、検索テキストボックスのすぐ右にあるツールバーアイコンを選択して、**Cloud Shell** ペインを表示します。

1. **Bash** や **PowerShell** のどちらかを選択するためのプロンプトが表示されたら、**PowerShell** を選択します。 

    >**注**: **Cloud Shell** を初めて起動し、「**ストレージがマウントされていません**」というメッセージが表示された場合は、このラボで使用しているサブスクリプションを選択し、「**ストレージの作成**」を選択します。 

1. Cloud Shell ペインで次のコマンドを実行して、次のタスクで提供する必要がある使用可能な DNS 名を特定します (プレースホルダー `<custom-label>` を、グローバルに一意である可能性が高い任意の有効な DNS ホスト名に置き換え、プレースホルダー `<Azure region>` を、Active Directory ドメイン コントローラーをホストする Azure VM をデプロイする Azure リージョンの名前に置き換えます)。

    ```powershell
    Test-AzDnsAvailability -DomainNameLabel <custom-label> -Location '<location>'
    ```
      >**注意**: Azure VM をプロビジョニングできる Azure リージョンを特定するには、 [https://azure.microsoft.com/ja-jp/regions/offers/](https://azure.microsoft.com/ja-jp/regions/offers/) を参照してください。
      ```powershell
      Get-AzLocation | FT
      ```

1. コマンドが **True** へ戻したことを確認します。そうでない場合は、コマンドが **True**へ戻すまで、`<custom-label>`の異なる値で同じコマンドを再実行します。

1. 成功した結果をもたらした `<custom-label>` の値を記録します。これは、次のタスクで必要になります。


#### タスク 2: Azure Resource Manager QuickStart テンプレートを使用して、AD DS ドメイン管理者を実行する Azure VM をデプロイする

1. Azure portal の Cloud Shell ペインのツールバーで、「**ファイルのアップロード/ダウンロード**」 アイコンを選択し、ドロップダウン メニューで 「**アップロード**」 を選択し、ファイル **\\\\AZ303\\AllFiles\Labs\\10\\azuredeploy30310suba.json** を Cloud Shell ホーム ディレクトリにアップロードします。

1. Cloud Shell ペインから、以下のコマンドを実行してリソース グループを作成します ('<Azure リージョン> プレースホルダーを、以前のタスクで指定した  Azure リージョンの名前に置き換えます）。

   ```powershell
   $location = '<Azure region>'
   ```
   ```powershell
   New-AzSubscriptionDeployment `
     -Location $location `
     -Name az30410subaDeployment `
     -TemplateFile $HOME/azuredeploy30410suba.json `
     -rgLocation $location `
     -rgName 'az30410a-labRG'
   ```

1. Azure portal で、**Cloud Shell** ウィンドウを閉じます。

1. ラボ コンピューターから別のブラウザータブを開き、[https://github.com/Azure/azure-quickstart-templates/tree/master/active-directory-new-domain](https://github.com/Azure/azure-quickstart-templates/tree/master/active-directory-new-domain) に移動します。 

1. **Create a new Windows VM and create a new AD Forest, Domain and DC(新しい Windows VM を作成し、新しい AD フォレスト、ドメイン、DC を作成する)** ページで、**Deploy to Azure(Azure に配置する)** を選択します。これにより、ブラウザーが自動的に、Azure portal 内の **Create an Azure VM with a new AD Forest(新しい AD フォレストで Azure VM を作成する)** ブレードにリダイレクトされます。

1. **Create an Azure VM with a new AD Forest(新しい AD フォレストで Azure VM を作成する)** ブレードで、「**パラメーターの編集**」 を選択します。

1. 「**パラメーターの編集**」 ブレードで、「**ファイルの読み込み**」 を選択し、「**開く**」 ダイアログ ボックスで、「**\\\\AZ303\\AllFiles\Labs\\10\\azuredeploy30310rga.parameters.json**」 を選択し、「**開く**」 を選択してから、「**保存**」 を選択します。 

1. **Create an Azure VM with a new AD Forest(新しい AD フォレストで Azure VM を作成する)** ブレードで、次の設定を指定します (他の設定は既存値のままにします)。

    | 設定 | 値 | 
    | --- | --- |
    | サブスクリプション | このラボのために使用する Azure サブスクリプションの名前 |
    | リソース グループ | **az30410a-labRG** |
    | DNS プレフィックス | 前のタスクで識別した DNS ホスト名| 

1. **Create an Azure VM with a new AD Forest(新しい AD フォレストで Azure VM を作成する)** ブレードで、「**確認および作成**」 をクリックし、さらに「**作成**」 をクリックします。

    > **注**: デプロイが完了するのを待たず、代わりに次の演習に進みます。デプロイには約 15 分間かかる場合があります。このラボの 3 番目の演習では、このタスクでデプロイされた仮想マシンを使用します。


### 演習 1: Azure AD ユーザーを作成して構成する
  
この演習の主なタスクは次のとおりです。

1. Azure AD テナントの作成

1. Azure AD ユーザーの作成および構成

1. Azure AD Premium P2 ライセンスをアクティブ化して割り当てる


#### タスク 1: Azure AD テナントの作成

1. Azure portal で、**Azure Active Directory** を検索して選択し、「Azure Active Directory」 ブレードで、**テナントを作成する** を選択します。

1. 「**テナントの作成**」 ブレードの 「**基本**」 タブで、「**Azure Active Directory**」 オプションを選択し、**次:構成 >** を選択します。

1. 「**テナントの作成**」 ブレードの 「**構成**」 タブで、次の設定を指定します (他の設定は既定値のままにします)。

    | 設定 | 値 |
    | --- | --- |
    | 組織名 | **Adatum Lab** |
    | 初期ドメイン名 | 小文字と数字で構成され、文字で始まる有効な DNS 名 | 
    | 国/地域 | **米国** |

   > **注**: **初期ドメイン名**テキスト ボックスの緑色のチェック マークは、入力したドメイン名が有効で一意であることを示します。

1. 「**次へ**」 を選択します。「**確認および作成**」 をクリックし、「**作成**」 を選択します。

1. Azure portal を表示するブラウザーのページを更新し、**Azure Active Directory** を検索して選択し、「Azure Active Directory」 ブレードで 「**テナントの切り替え**」 を選択します。

1. **Adatum Lab** カード上の「ディレクトリ + サブスクリプション」ブレードで、「**切り替え**」を選択します。


#### タスク 2: Azure AD ユーザーの作成および構成

1. **Adatum Lab** で | 「**概要**」 Azure Active Directory ブレードの、「**管理**」 セクションで、「**ユーザー**」 を選択します | 「**すべてのユーザー**」 ブレードで表示されているユーザー アカウントを選択して、「**プロファイル**」 設定を表示します。 

1. ユーザー アカウントのプロファイル ブレードで、「**編集**」 を選択し、「**設定**」 セクションで、「**使用場所**」 を 「**United States**」 に設定し、変更を **保存** します。

    >**注**: これは、このラボの後半で Azure AD Premium P2 ライセンスをユーザー アカウントに割り当てるために必要です。

1. 「**ユーザー - すべてのユーザー**」 ブレードに戻り、「**新しいユーザー**」 を選択します。

1. 「**新しいユーザー**」 ブレードで、次の設定を指定します (他の設定は既定値のままにします)。

    | 設定 | 値 |
    | --- | --- |
    | ユーザー名 | **az30410-aaduser1** |
    | 名前 | **az30410-aaduser1** |
    | 自動生成されたパスワード | 有効 |
    | パスワードの表示 | 有効 |
    | ロール | **グローバル管理者** |
    | 使用場所 | **United States** |

    >**注**: ユーザー名全体 (ドメイン名を含む) と自動生成されたパスワードを記録します。このタスクの後半で必要になります。

1. 「**新しいユーザー**」 ブレードで 「**作成**」 を選択します

1. ラボ コンピューターで **InPrivate** ブラウザーの画面を開き、新しく作成した **az30410-aaduser1** ユーザー アカウントを使用して [Azure portal](https://portal.azure.com) にログインします。パスワードの更新を求められたら、パスワードを **Pa55w.rd1234** に変更します。 

1. Azure portal から **az30410-aaduser1** ユーザーとしてサイン アウトし、InPrivate ブラウザーの画面を閉じます。


#### タスク 3: Azure AD Premium P2 ライセンスをアクティブ化して割り当てる

1. Azure portal を表示しているブラウザーの画面に戻り、**Adatum Lab** Azure AD テナントの**概要**ブレードに移動し、「**管理**」 セクションで、「**ライセンス**」 を選択します。

1. **ライセンス**で | 「**概要**」ブレード、「**すべての製品**」 を選択し、「**+ 試用/購入**」 を選択します。

1. 「**アクティブ化**」 ブレードの 「**Azure AD Premium P2**」 セクションで、「**無料試用版**」 を選択してから、「**アクティブ化**」 を選択します。 

1. 「**ライセンス**」 を表示しているブラウザーの画面を更新すし、 **すべての製品** ブレードでアクティブ化が成功したことを確認します。 

1. 「**ライセンス - すべての製品**」 ブレードで、「**Azure Active Directory Premium P2**」  エントリを選択します。 

1. 「**Azure Active Directory Premium P2**」 で **ライセンスされているユーザー** ブレード、**+ 割り当て** を選択してください。 

1. **ライセンスの割り当て** ブレードで、**ユーザー** ブレードを選択し、自身のアカウントと **az30410-aaduser1** ユーザー アカウントを選択します。

1. **ライセンスを割り当てる** ブレードに戻り、**割り当てオプション** を選択します。**ライセンス オプション** ブレードに記載されているオプションを確認し、**OK** を選択します。

1. **ライセンスを割り当てる** ブレードで **割り当て** を選択します。 


### 演習 2: AD DS フォレストを Azure AD テナントに統合する
  
この演習の主なタスクは次のとおりです。

1. Azure AD テナントにカスタム ドメイン名を割り当てる

1. Azure VM で AD DS を構成する

1. Azure AD Connect のインストール

1. 同期されたユーザー アカウントのプロパティを構成する


#### タスク 1: Azure AD テナントにカスタム ドメイン名を割り当てる

1. Azure portal で **Azure Active Directory Adatum Lab** に移動します。**| 概要** ブレード。

1. **Adatum Lab** で **| 概要**ブレード、**カスタム ドメイン名** を選択します。

1. **Adatum Lab** で **| カスタム ドメイン名** ブレードで、Azure AD テナントに関連付けられているプライマリ デフォルト DNS ドメイン名を確認します。 

    >**注**: Azure AD テナントのプライマリ DNS 名の値を記録します。これは、次のタスクで必要になります。

1. **Adatum Lab** で **| カスタム ドメイン名** ブレード、**カスタム ドメインの追加** を選択します。

1. **カスタム ドメイン名** ブレードの、**カスタム ドメイン名** テキスト ボックスで、**adatum.com** と入力し、**ドメインを追加** を選択します。 

1. **adatum.com** ブレードで、Azure ADドメイン名の検証を実行するために必要な情報を確認し、ブレードを閉じます。

    > **注**: **adatum.com** DNS ドメイン名を所有していないため、検証プロセスを完了できません。これにより **adatum.com** Azure AD テナントのある Active Directoryドメインを同期*しない*ようにします。この目的で、このタスクの前半で特定した、Azure AD テナントのデフォルトのプライマリ DNS 名 (名前の最後が **onmicrosoft.com**) を使用します。ただし、その結果、Active Directory ドメインの DNS ドメイン名と Azure AD テナントの DNS 名が異なることに注意してください。つまり、Adatum ユーザーは、Active Directory ドメインにサインインするときと、Azure AD テナントにサインインするときに、異なる名前を使用する必要があります。


#### タスク 2: Azure VM で AD DS を構成する

> **注**: この演習を開始する前に、ラボの初めに開始した Azure VM のデプロイが完了していることを確認してください。

1. Azure portalで、**仮想マシン** を検索して選択し、 **「Virtual Machines」** ブレードで、**az30410a-vm1** を選択します 。

1. **az30410a-vm1** ブレードで、**接続** を接続し、ドロップダウン メニューで **RDP** を選択し、**IP アドレス** ドロップダウン リストで **ロードバランサーのパブリック IP アドレス** エントリを選択し、次に **RDP ファイルをダウンロード** を選択します。

1. プロンプトが表示されたら、次の認証情報を入力します。

    | 設定 | 値 | 
    | --- | --- |
    | ユーザー名 | **Student** |
    | パスワード | **Pa55w.rd1234** |

1. **az30410a-vm1** へのリモート デスクトップ セッション内にある 「サーバー マネージャー」 ウィンドウで、「**ローカル サーバー**」 を選択し、「**IE 強化されたセキュリティ構成**」 ラベルの横にある 「**オン**」 のリンクを選択し、「**IE 強化されたセキュリティ構成**」 ダイアログ ボックスで、両方の 「**オフ**」 オプションを選択します。

1. **az30410a-vm1** へのリモート デスクトップ セッション内にある 「サーバー マネージャー」 ウィンドウで、「**ツール**」 を選択し、ドロップダウン メニューで、「**Active Directory 管理センター**」 を選択します。

1. 「**Active Directory Administrative Center**」 で、「**adatum (Local)**」 を選択し、右側の「**Tasks**」  ペインで、「**New**」 を選択し、カスケード メニューで 「**Organizational Unit(組織単位)**」 を選択します。

1. 「**Create Organizational Unit(組織単位を作成)**」 ウィンドウの 「**Name(名前)**」 テキストボックスに、「**ToSync**」と入力し、「**OK**」 を選択します。

1. 新しく作成された **ToSync** 組織単位をダブルクリックして開きます。

1. 「**Tasks**」  ペインの 「**ToSync**」 セクション内で、「**New**」 を選択し、カスケード メニューで、「**User**」 を選択します。

1. 「**ユーザーを作成**」 ウィンドウで、次の設定を使用して新しいユーザーアカウントを作成し (他の設定は既存値のままにします)、「**OK**」 を選択します。

    | 設定 | 値 | 
    | --- | --- |
    | Full Name | **aduser1** |
    | User UPN logon | **aduser1** |
    | User SamAccountName logon | **aduser1** |
    | パスワード | **Pa55w.rd1234** | 
    | その他のパスワード オプション | **Password never expires(パスワードは期限切れになりません)** |


#### タスク 3: Azure AD Connect のインストール

1. **az30410a-vm1** へのリモートデスクトップセッション内 で、Internet Explorer を起動し、[Azure portal](https://portal.azure.com) に移動し、前の演習で作成した **az30410-aaduser1** ユーザー アカウントを使用してサインインします。プロンプトが表示されたら、記録した完全なユーザー名とパスワード **Pa55w.rd1234** を指定します。

1. Azure portal で、**Azure Active Directory**を検索して選択し、**Adatum Lab** で **Overview** ブレード、**Azure AD Connect** を選択します。

1. **Adatum Lab** の **Azure AD Connect** ブレード、**Download Azure AD Connect** リンクを選択します。**Microsoft Azure Active Directory Connect** のダウンロード ページにリダイレクトされます。

1. **Microsoft Azure Active Directory Connect** のダウンロード ページで、**Download**を選択します。

1. プロンプトが表示されたら、**Run**を選択して**Microsoft Azure Active Directory Connect** ウィザードを起動します。

1. **Microsoft Azure Active Directory Connect** ウィザードの **Azure AD Connect へようこそ**ページで、チェックボックス**ライセンス条項とプライバシーに関する通知に同意します**を選択し、**続行**を選択します。

1. **Microsoft Azure Active Directory Connect** ウィザードの **Express Settings(簡易設定)** のページで、**Customize(カスタマイズ)** オプションを選択します。

1. **必要なコンポーネントをインストールする**ページで、オプションの構成オプションをすべて選択解除したままにして、**インストール**を選択します。

1. **ユーザーのサインイン**ページで、**Password Hash Synchlonization(パスワード ハッシュの同期)** のみ有効にして、**次へ** を選択します。

1. **Azure ADに接続する**ページで、前の演習で作成した**az30410-aaduser1**ユーザー アカウントの認証情報を使用して認証し、 **次へ**を選択します。 

1. **ディレクトリに接続する**ページで、 **adatum.com** フォレスト エントリの右側にある **Add Directory(ディレクトリの追加)** ボタンを選択します。

1. **ADフォレスト アカウント**ウィンドウで、**Create new AD account(新しい AD アカウントを作成する)** オプションが選択されていることを確認し、次の資格情報を指定して、**OK** を選択します。

    | 設定 | 値 | 
    | --- | --- |
    | ユーザー名 | **ADATUM\Student** |
    | パスワード | **Pa55w.rd1234** |

1. **ディレクトリを接続する**ページに戻り、**adatum.com** エントリが構成済みディレクトリとして表示されていることを確認し、**次へ**を選択します

1. **Azure AD サインイン情報の構成**ページで、**Users will be able to sign-in to Azure AD with on-premises credentials if the UPN suffix does not match a verified domain.(UPN サフィックスが確認済みのドメイン名と一致しない場合、ユーザーはオンプレミスの資格情報で Azure AD にサインインできません)** という警告に注意して、チェックボックス **Continue without matching all UPN suffixed to verified domains(すべての UPN サフィックスを確認済みドメインに一致させずに続行)** を有効にし、**次へ** を選択します。

    > **注**: 前に説明したように、カスタム Azure AD DNS ドメイン **adatum.com** を確認できなかったため、これは予想されたものです。

1. **ドメインと OU フィルタリング**ページで、オプション**選択したドメインと OU を同期する**を選択し、すべてのチェックボックスをクリアにし、**ToSync** OU の横にあるチェックボックスのみを選択して、**次へ**を選択します。

1. **ユーザーを一意に識別する**ページで、既定値の設定を承諾し、**次へ**を選択します。

1. **ユーザーとデバイスをフィルタリングする** ページで、既定値の設定を承諾し、**次へ**を選択します。

1. **オプション機能** ページで、既定の設定値を承諾してから、**次へ** を選択します。

1. **構成する準備ができました** ページで、**Start the synchronization process when configuration completes(構成が完了したら同期プロセスを開始します)** チェックボックスが選択されていることを確認し、**インストール** を選択します。

    > **注**: インストールにはおよそ 2 分かかります。

1. **構成が完了しました** ページの情報を確認し、**終了** を選択し、**Microsoft Azure Active Directory Connect** ウィンドウを閉じます。


#### タスク 4: 同期されたユーザー アカウントのプロパティを構成する

1. リモート デスクトップ セッション内から Azure portal を表示している Internet Explorer ウィンドウで、Adatum Lab Azure AD テナントの **ユーザー - すべてのユーザー**ブレードへ移動します。

1. 「**すべてのユーザー**」 ブレードで、ユーザー オブジェクトのリストには **aduser1** アカウントが含まれており、**同期されたディレクトリ** が **はい** 列に表示されていることに注意してください。

    > **注**: **aduser1**ユーザー アカウントを表示するには、数分待ってから 「**更新**」 をクリックする必要がある場合があります。

1. **ユーザー**の「**すべてのユーザー**」 ブレードで、「**aduser1**」 エントリを選択します。

1. 「**aduser1**」 の「**プロフィール**」 ブレードで、ユーザー アカウントの Name(フルネーム) をメモしておきます。

    > **注**: 完全なユーザー名を記録します。これは、次の演習で必要になります。

1. 「**aduser1**」 の | 「**プロフィール**」 ブレードの 「**Job Info(職務情報)**」 セクションに、「**Department(部署)**」 属性が設定されていないことに注意してください。

1. **az30410a-vm1** へのリモート デスクトップ セッション内で、 **Active Directory 管理センター** に切り替えて、**ToSync** OU のオブジェクトのリストで 「**aduser1**」 エントリを選択し、「**Tasks(タスク)**」 ペインの 「**ToSync**」 セクションで、「**Properties(プロパティ)**」 を選択します。

1. **aduser1** ウィンドウの、 「**Organization(組織)**」 セクションで、 「**Department(部署)**」 テキスト ボックスに、「**Sales**」と入力し、 「**OK**」 をクリックします。

1. **az30410a-vm1** へのリモート デスクトップ セッション内で、**Windows PowerShell** を管理者で起動します。

1. 次のコマンドを実行して、Azure AD Connect の差分同期を起動します。

   ```powershell
   Import-Module -Name 'C:\Program Files\Microsoft Azure AD Sync\Bin\ADSync\ADSync.psd1'

   Start-ADSyncSyncCycle -PolicyType Delta
   ```

1. **aduser1** を表示している Internet Explorerウィンドウに切り替えます。 | 「**プロファイル**」 ブレードで、ページを更新して、「**Department(部署)**」 プロパティは 「**Sales**」と設定されていることを確認します。

    > **注**: 「**部署**」 属性が設定されていないままの場合、さらに 1 分待ってからページを再度更新する必要があります。

1. 「**aduser1**」 の | 「**プロファイル**」 ブレードで、「**編集**」 を選択します。

1. 「**aduser1**」 の | 「**プロファイル**」 ブレードの、「**Settings(設定)**」 セクションで、「**Usage location(使用場所)**」 のドロップダウン リストから、「**United States**」 を選択し、 次に 「**保存**」 をクリックします。

1. 「**aduser1**」 の | 「**プロフィール**」 ブレードで、「**ライセンス**」を選択します。

1. 「**aduser1**」 の | 「**ライセンス**」 ブレードで、「**+ 割り当て**」 を選択します。

1. 「**ライセンスの割り当ての更新**」 ブレードで、「**Azure Active Directory Premium P2**」 チェックボックスを選択して 「**保存**」をクリックします。


### 演習 3: Azure AD 条件付きアクセスの導入
  
この演習の主なタスクは次のとおりです。

1. Azure AD セキュリティのデフォルトを無効にします。

1. Azure AD 条件付きアクセス ポリシーを作成する

1. Azure AD の条件付きアクセスを確認する

1. ラボにデプロイした Azure リソースを削除する


#### タスク 1: Azure AD セキュリティのデフォルトを無効にします。

1. **az30410a-vm1** へのリモート デスクトップ セッション内で 、Azure portal が表示されている Internet Explorer ウィンドウで、 **Adatum Lab** テナントの 「**概要**」 ブレードに移動します。

1. **Adatum Lab** で「**概要**」 ブレードの 「**管理**」 セクションで、「**プロパティ**」 を選択します。

1. **Adatum Lab** で | 「**プロパティ**」 ブレードで、ページの下部にある 「**Manage Security defaults(セキュリティの既定値群の管理)**」 リンクを選択します。

1. 「**セキュリティの既定値群の有効化**」 ブレードで、「**セキュリティの既定値群を有効にする**」 の設定を 「**No**」 に替えて、チェックボックスの 「**My organization is using Conditional Access(私の組織は条件付きアクセスを使用しています)**」 を選択して、「**保存**」 をクリックします。 


#### タスク 2: Azure AD 条件付きアクセス ポリシーを作成する

1. **Adatum Lab** の「**管理**」セクションで、 「**セキュリティ**」 を選択します。

1. 「**セキュリティ | 開始**」 ブレードで、「**Conditional Access(条件付きアクセス)**」 を選択します。

1. 「**条件付きアクセス**」 の「**ポリシー**」 ブレードで、「**+ 新しいポリシー**」 を選択します。

1. 「**新規**」 ブレードの 「**名前**」 テキストボックスに、「**Azure portal MFA 実施**」と入力します。 

1. 「**新規**」 ブレードの 「**Assignment(割り当て)**」 セクションで、「**ユーザーとグループ**」 を選択し、「**Include(含む)**」 タブで、「**Select users and groups(ユーザーとグループを選択)**」 を選択し、「**Users and groups(ユーザーとグループ)**」 チェックボックスをオンにし、「**選択**」 ブレード で、「**aduser1**」 を選択し、「**選択**」 をクリックして選択項目を確定します。

1. 「**新規**」 ブレードに戻り、「**割り当て**」 セクションで、「**Cloud apps and actions(クラウド アプリまたはアクション)**」 を選択し、「**Include(含む)**」 タブで、「**Select app(アプリを選択)**」 を選択し、「**選択**」 ブレードで、「**Microsoft Azure Management**」 チェックボックスをオンにし、「**選択**」 をクリックして選択項目を確定します。

1. 「**新規**」 ブレードに戻り、「**Access controls(アクセス制御)**」 セクションで、「**Grant access(許可)**」 を選択し、「**Require multi-factor authentication(多要素認証が必要)**」 を選択し、「**選択**」 をクリックして選択項目を確定します。

1. 「**新規**」 ブレードに戻り、「**Enable policy(ポリシーを有効にする)**」 スイッチを 「**オン**」 に設定し、「**作成**」 を選択します。


#### タスク 3: Azure AD の条件付きアクセスを確認する

1. **az30410a-vm1** へのリモート デスクトップ セッション内で、新しい **InPrivate Browsing** Internet Explorer ウィンドウを起動し、アクセス パネル アプリケーション ポータル [https://account.activedirectory.windowsazure.com](https://account.activedirectory.windowsazure.com) に移動します。

1. プロンプトが表示されたら、同期された Azure AD アカウント **aduser1** で、前の演習で記録した完全なユーザー名と、**Pa55w.rd1234** のパスワードを使用してサインインします。 

1. アクセス パネル アプリケーション ポータルに正常にサインインできることを確認します。 

1. 同じブラウザーの画面で、[Azure portal](https://portal.azure.com) に移動します。

1. 今回は、「**More Information Required(追加情報が必要)**」のメッセージが表示されていることを確認してください。メッセージを表示するページ内で、「**次へ**」 を選択します。 

1. その時点で、「**追加のセキュリティ検証**」 ページ にリダイレクトされます。このページでは、多要素認証の構成をステップごとに説明します。

    > **注**: 多要素認証構成の完了はオプションです。続行する場合は、モバイル デバイスを認証電話として指定するか、モバイル デバイスを使用してモバイル アプリを実行する必要があります。


#### タスク 4: ラボにデプロイした Azure リソースを削除する

1. **az30410a-vm1** へのリモート デスクトップ セッション内で、Internet Explorer を起動し、IT プロフェッショナル RTW 向けの Microsoft Online Services サインイン アシスタント [https://go.microsoft.com/fwlink/p/?LinkId=286152](https://go.microsoft.com/fwlink/p/?LinkId=286152) に移動します。 

1. IT プロフェッショナル RTW 向け Microsoft Online Services サインイン アシスタントのダウンロード ページで、「**ダウンロード**」 を選択し、「**必要なダウンロードを選択してください**」 ページで、「**en\msoidcli_64.msi**」 を選択し、「**次へ**」 を選択します。 

1. プロンプトが表示されたら、既定のオプションを使用して、**Microsoft Online Services サインイン アシスタントのセットアップ**を実行します。

1. セットアップが完了したら、**az30410a-vm1** へのリモート デスクトップ セッション内で、「**Windows PowerShell**」 コンソールを起動します。

1. **管理者**で「**Windows PowerShell**」 ウィンドウで、次のコマンドを実行して、必要な PowerShell モジュールをインストールします。

   ```powershell
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   Install-PackageProvider -Name NuGet -MinimumVersion 2.8.5.201 -Force
   Install-Module MSOnline -Force
   ```
1. **管理者**で「**Windows PowerShell**」 ウィンドウで、次のコマンドを実行して、**Adatum Lab** Azure AD テナントに対して認証します。

   ```powershell
   Connect-MsolService
   ```

1. 認証を求められたら、**az30410-aaduser1** ユーザー アカウントの認証情報を提供します。

1. **管理者**で「**Windows PowerShell**」 ウィンドウで、次のコマンドを実行して、Azure AD Connect の同期を無効にします。

   ```powershell
   Set-MsolDirSyncEnabled -EnableDirSync $false -Force
   ```

1. ラボ コンピューターから、Azure portal が表示されているブラウザーの画面で、「**Azure Active Directory Premium P2 - ライセンス ユーザー**」 ブレードに移動し、このラボでライセンスを割り当てたユーザー アカウントを選択し、「**ライセンスを削除**」 を選択し、確認を求められたら、「**OK**」 を選択します。

1. Azure portal で、 「**ユーザー - すべてのユーザー**」 ブレードに移動し、このラボで作成したすべてのユーザー アカウントには、「**ソース**」 列に **Azure Active Directory** があることを確認します。 

1. 「**ユーザー - すべてのユーザー**」 ブレードで、このラボで作成した各ユーザー アカウントを選択し、ツールバーで 「**削除**」 を選択します。 

1. Adatum Lab Azure AD テナントの 「**Adatum Lab - 概要**」 ブレードに移動し、「**テナントを削除**」 を選択し、「**ディレクトリ 'Adatum Lab' を削除する**」 ブレードで、「**Azure リソースを削除する権限を取得する**」 リンクを選択し、Azure Active Directory の 「**プロパティ**」 ブレードで、「**Azure リソースのアクセス管理**」 を 「**はい**」 に設定し、「**保存**」 を選択します。

1. Azure portal からサインアウトし、再度サインインします。 

1. 「**ディレクトリ 'Adatum Lab' を削除する**」 ブレードに戻り、「**削除**」 を選択します。

1. **az30410a-vm1** へのリモート デスクトップ セッション内の Azure portal を表示しているブラウザーの画面で、Cloud Shell ペイン内で PowerShell セッションを起動します。

1. 「Cloud Shell」 ウィンドウから次のコマンドを実行して、この演習で作成したリソース グループを一覧表示します。

   ```powershell
   Get-AzResourceGroup -Name 'az30410*'
   ```

    > **注**: このラボで作成したリソース グループのみが出力に含まれていることを確認します。このグループは、このタスクで削除されます。

1. 「Cloud Shell」 ウィンドウから次を実行して、このラボで作成したリソース グループを削除します

   ```powershell
   Get-AzResourceGroup -Name 'az30410*' | Remove-AzResourceGroup -Force -AsJob
   ```

1. 「Cloud Shell」 ウィンドウを閉じます。
