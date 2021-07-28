---
lab:
    title: '3: Azure Migrate を使用して Hyper-V VM を Azure に移行する'
    module: 'モジュール 3: 移行を設計する'
---

# ラボ: Azure Migrate を使用して Hyper-V VM を Azure に移行する
# 受講生用ラボ マニュアル

## ラボ シナリオ

Azure への移行の一部としてワークロードを最新化するという野心にもかかわらず、Adatum Enterprise Architecture チームは、積極的なタイムラインのため、多くの場合、リフトアンドシフトのアプローチに従う必要があることを認識しています。このタスクを簡略化するために、Adatum Enterprise Architecture チームは Azure Migrate の機能の調査を開始しました。Azure Migrate は、Azure オンプレミス サーバー、インフラストラクチャ、アプリケーション、およびデータを評価して移行するための集中ハブとして機能します。

Azure Migrate には次の機能が用意されています。

- 統合された移行プラットフォーム。Azure への移行を開始、実行、追跡するための単一のポータル。
- ツールの範囲: 評価と移行のためのさまざまなツール。ツールには、Azure Migrate: Server Assessment、Azure Migrate: Server Migration に関するエラーのトラブルシューティングに役立つ情報を提供しています。Azure Migrate は、他の Azure サービスや、他のツールおよび独立系ソフトウェア ベンダー (ISV) オファリングと統合されています。
- 評価と移行: Azure Migrate ハブでは、以下の評価と移行を行うことができます。
- サーバー: オンプレミスのサーバーを評価し、Azure の仮想マシンに移行します。
- データベース: オンプレミスのデータベースを評価し、Azure SQL Database または SQL Managed Instance に移行します。
- Web アプリケーション: Azure App Service Migration Assistant を使用して、オンプレミスの Web アプリケーションを評価し、Azure App Service に移行します。
- 仮想デスクトップ: オンプレミスの仮想デスクトップ インフラストラクチャ (VDI) を評価し、Azure の Windows Virtual Desktop に移行します。
- Data: Azure Data Box 製品を使用して、大量のデータを迅速かつコスト効果よく Azure に移行します。

データベース、Web アプリ、仮想デスクトップは移行イニシアチブの次の段階の範囲にありますが、Adatum Enterprise Architecture チームは、オンプレミスの Hyper-V 仮想マシンを Azure VM に移行するための Azure Migrate の使用を評価することから始めたいと考えています。

## 目標
  
このラボを終了すると、次のことができるようになります。

-  Azure Migrate を使用して、評価と移行のために Hyper-V を準備する

-  Azure Migrate を使用して移行のために Hyper-V を評価する

-  Azure Migrate を使用して Hyper-V VM を移行する


## ラボ環境
  
Windows サーバー管理者の資格情報

-  ユーザー名: **Student**

-  パスワード: **Pa55w.rd1234**

推定時間: 120 分


## ラボ ファイル

-  \\\\AZ304\\AllFiles\\Labs\\08\\azuredeploy30308suba.json


### 演習 0: ラボ環境の準備

この演習の主なタスクは次のとおりです。

1. Azure Resource Manager クイックスタート テンプレートを使用して Azure VM をデプロイする

1. Azure VM でネストされた仮想化を構成する


#### タスク 1: Azure Resource Manager クイックスタート テンプレートを使用して Azure VM をデプロイする

1. ラボのコンピューターから Web ブラウザーを起動し、[Azure portal](https://portal.azure.com) - `portal.azure.com` に移動して、このラボで使用するサブスクリプションの所有者の役割を持つユーザー アカウントの認証情報を提供してサインインします。

1. Azure portal で、検索テキストボックスのすぐ右にあるツールバー アイコンを選択して 「**Cloud Shell**」 ペインを開きます。

1. **Bash** や **PowerShell** のどちらかを選択するためのプロンプトが表示されたら、**PowerShell** を選択します。 

   >**注**: **Cloud Shell** を初めて起動し、「**ストレージがマウントされていません**」というメッセージが表示された場合は、このラボで使用しているサブスクリプションを選択し、「**ストレージの作成**」 を選択します。 

1. Cloud Shell ペインのツールバーで、 **ファイルのアップロード/ダウンロード** アイコンを選択し、ドロップダウン メニューで **アップロード**を選択して、ファイル **\\\\\AZ303\\AllFiles\Labs\\08\\azuredeploy30308suba.json** を Cloud Shell ホーム ディレクトリにアップロードします。

1. Cloud Shell から次のコマンドを実行して最寄りの Azure リージョンのある場所の名前が付いた変数を設定します (`<Azure region>` プレースホルダーを、サブスクリプションでの Azure VM のデプロイに使用でき、ラボのコンピューターの場所に最も近い Azure リージョンの名前に置き換えます。例: 'eastus'):

   ```powershell
   $location = '<Azure region>'
   ```

      > **注**: Azure VM をプロビジョニングできる Azureリージョンを特定するには、次を参照してください。[**https://azure.microsoft.com/ja-jp/regions/offers/**](https://azure.microsoft.com/ja-jp/regions/offers/)
      
1. Cloud Shell ペインで次の操作を実行してリソース グループを作成します:

   ```powershell
   New-AzSubscriptionDeployment `
     -Location $location `
     -Name az30308subaDeployment `
     -TemplateFile $HOME/azuredeploy30308suba.json `
     -rgLocation $location `
     -rgName 'az30308a-labRG'
   ```

1. Azure portal で、**Cloud Shell** ウィンドウを閉じます。

1. ラボ コンピューターから別のブラウザー タブを開き、[301-nested-vms-in-virtual-network Azure QuickStart template](https://github.com/Azure/azure-quickstart-templates/tree/master/demos/nested-vms-in-virtual-network) に移動して、「**Deploy to Azure**」 を選択します。これにより、Azure portal の 「**Hyper-V Host Virtual Machine with nested VMs.**」 ブレードにブラウザーが自動的にリダイレクトされます。

    ``` url
    https://github.com/Azure/azure-quickstart-templates/tree/master/demos/nested-vms-in-virtual-network
    ```

1. Azure portal の 「**VM がネストされた Hyper-V ホスト仮想マシン**」 ブレードで、次の設定を指定します (他は既定値のままにします)。

    | 設定 | 値 | 
    | --- | --- |
    | サブスクリプション | このラボで使用する Azure サブスクリプションの名前 |
    | リソース グループ | **az30308a-labRG** |
    | ホスト パブリック IP アドレス名 | **az30308a-hv-vm-pip** |
    | 仮想ネットワーク名 | **az30308a-hv-vnet** |
    | ホスト ネットワークのインターフェイス 1 の名前 | **az30308a-hv-vm-nic1** |
    | ホスト ネットワークのインターフェイス 2 の名前 | **az30308a-hv-vm-nic2** |
    | ホスト仮想マシン名 | **az30308a-hv-vm** |
    | ホスト管理者ユーザー名 | **Student** |
    | ホスト管理者パスワード | **Pa55w.rd1234** |

1. 「**Hyper-V Host Virtual Machine with nested VMs　(入れ子になった VM による仮想マシンでの Hyper-V)**」ブレードで、「**レビュー + 作成**」を選択し、「**作成**」を選択します。

    > **注**: デプロイが完了するのを待ちます。デプロイには約 10 分間かかります。

#### タスク 2: Azure VM でネストされた仮想化を構成する

1. Azure portal で、「**仮想マシン**」 を検索して選択し、「**仮想マシン**」 ブレードで、「**az30308a-hv-vm**」 を選択します。

1. 「**az30308a-hv-vm**」 ブレードで、「**ネットワーク**」 を選択します。 

1. 「**az30308a-hv-vm \| ネットワーク**」 ブレードで、「**az30308a-hv-vm-nic1**」 タブを選択し、次に 「**受信ポート規則を追加する**」 を選択します。

    >**注**: 必ず、パブリック IP アドレスが割り当てられている **az30308a-hv-vm-nic1** の設定を変更してください。

1. 「**受信セキュリティ規則の追加**」 ブレードで、次の設定を指定し (他は既定値のままにします)、「**追加**」 を選択します。

    | 設定 | 値 | 
    | --- | --- |
    | 宛先ポート範囲 | **3389** |
    | プロトコル | **Any** |
    | 名前 | **AllowRDPInBound** |

1. 「**az30308a-hv-vm**」 ブレードで、「**概要**」 を選択します。 

1. 「**az30308a-hv-vm**」 ブレードで、「**接続**」 を選択し、ドロップダウン メニューで、「**RDP**」 を選択して、「**RDP ファイルのダウンロード**」 をクリックします。

1. プロンプトが表示されたら、「**接続**」 をクリックし、以下の認証情報を使用してサインインします。

    | 設定 | 値 | 
    | --- | --- |
    | ユーザー名 | **Student** |
    | パスワード | **Pa55w.rd1234** |

1. **az30308a-hv-vm** へのリモート デスクトップ セッション内で、「Server Manager」 ウィンドウの 「**Local Server**」 をクリックし、「**IE Enhanced Security Configuration**」 ラベルの横にある 「**On**」 リンクをクリックし、「**IE Enhanced Security Configuration**」 ダイアログ ボックスで両方の 「**Off**」 オプションを選択した後、「**OK**」 をクリックします 。

1. リモート デスクトップ セッション内で、エクスプローラーを開き、**F:** に移動し、**VHDs** という名前のフォルダーを作成します。

1. **az30308a-hv-vm** へのリモート デスクトップ セッション内で、Internet Explorer を起動し、[Microsoft Edge](https://www.microsoft.com/ja-jp/edge/business/download) のダウンロード ページに移動し、Microsoft Edge のインストーラーをダウンロードし、インストールを実行します。 

1. **az30308a-hv-vm** へのリモート デスクトップ セッション内で、Microsoft Edge を起動し、[Windows サーバーの評価](https://www.microsoft.com/ja-jp/evalcenter/evaluate-windows-server-2019) を参照して、Windows Server 2019 **VHD** ファイルを **F:\VHDs** フォルダーにダウンロードします。 
    
    ```url
    https://www.microsoft.com/ja-jp/evalcenter/evaluate-windows-server-2019
    ```
    
    > **注**: 評価ページでは、ダウンロードを完了するために個人情報を尋ねられます。「英国」 または他の国を選択すると、通知をオプトアウトできます。

1. **az30308a-hv-vm** へのリモート デスクトップ セッション内で 「スタート」 をクリックし、「**Windows Administrative Tools**」 をクリックして **Hyper-V Manager**を起動します。 

1. **HHyper-V Manager** コンソールで、「**az30308a-hv-vm**」 ノードを選択します。 

1. 「**New**」 をクリックし、カスケード メニューで 「**Virtual Machine**」 をクリックします。これにより、「**New Virtual Machine Wizard**」 が起動します。 

1. 「**New Virtual Machine Wizard**」 の 「**Before You Begin**」 ページで、「**Next**」 をクリックします。

1. 「**New Virtual Machine Wizard**」 の 「**Specify Name and Location**」 ページで、次の設定を指定して 「**Next**」 をクリックします。

    | 設定 | 値 | 
    | --- | --- |
    | 名前 | **az30308a-vm1** | 
    | Store the virtual machine in a different location | 選択済み | 
    | 場所 | **F:\VMs** |

    >**注**: 必ず **F:\VMs** フォルダーを作成してください。

1. 「**New Virtual Machine Wizard**」 の 「**Specify Generation**」 ページで、「**Generation 1**」 オプションが選択されていることを確認して、「**Next**」 をクリックします。

1. 「**New Virtual Machine Wizard**」 の 「**Assign Memory**」 ページで、「**Startup memory**」 に「**2048**」を設定し、「**Next**」 をクリックします。

1. 「**New Virtual Machine Wizard**」 の 「**Configure Networking**」 ページで、「**Connection**」 ドロップダウン リストから 「**NestedSwitch**」 を選択し、「**Next**」 をクリックします。

1. 「**New Virtual Machine Wizard**」 の 「**Connect Virtual Hard Disk**」 ページで、「**Use an existing virtual hard disk**」 オプションを選択し、**F:\VHDs** フォルダーにダウンロードした VHD ファイルの場所を設定して、「**Next >**」 を選択します。

1. 「**New Virtual Machine Wizard**」 の 「**Completing the New Virtual Machine Wizard**」 ページで、「**Finish**」 を選択します。

1. 「**Hyper-V Manager**」 コンソールで、新しく作成した仮想マシンを選択し、「**Start**」 を選択します。 

1. 「**Hyper-V Manager**」 コンソールで、仮想マシンが実行されていることを確認し、「**Connect**」 を選択します。 

1. 「**az30308a-vm1**」 への仮想マシン接続ウィンドウの 「**Hi there**」 ページで、「**Next**」 を選択します。 

1. 「**az30308a-vm1**」 への仮想マシン接続ウィンドウの 「**License terms**」 ページで、「**Accept**」 を選択します。 

1. 「**az30308a-vm1**」 への仮想マシン接続ウィンドウの 「**Customize settings**」 ページで、ビルトイン管理者アカウントのパスワードを「**Pa55w.rd1234**」に設定し、「**Finish**」 を選択します。 

1. 「**az30308a-vm1**」 への仮想マシン接続ウィンドウで、新しく設定したパスワードを使用してサインインします。

1. 「**az30308a-vm1**」 への仮想マシン接続ウィンドウで、Windows PowerShell を起動し、「**Administrator: Windows PowerShell**」 ウィンドウで次のコマンドを実行して、コンピューター名を設定します。 

   ```powershell
   Rename-Computer -NewName 'az30308a-vm1' -Restart
   ```

### 演習 1: Azure Migrate を使用した評価と移行の準備
  
この演習の主なタスクは次のとおりです。

1. Hyper-V 環境を構成する

1. Azure Migrate プロジェクトを作成する

1. ターゲット Azure 環境を実装する


#### タスク 1: Hyper-V 環境を構成する

1. **az30308a-hv-vm** へのリモート デスクトップ セッション内で Microsoft Edge を起動し、[[Microsoft ダウンロード センター](https://aka.ms/migrate/script/hyperv)] に移動して、構成 PowerShell スクリプトを **F:** にダウンロードします。

    ```url
    https://aka.ms/migrate/script/hyperv
    ```

    > **注**: このスクリプトは、次を実行します。

    - サポートされている PowerShell バージョンでスクリプトが実行されていることを確認します。

    - Hyper-V ホストに対する管理者権限があることを確認します。

    - Azure Migrate サービスが Hyper-V ホストとの通信に使用するローカル ユーザー アカウントを作成できます。このユーザー アカウントは、Hyper-V ホストの リモート管理ユーザー、 Hyper-V Administrators、および パフォーマンス モニターユーザー グループに追加されます。

    - サポートされているバージョンの Hyper-V がホストで実行されていることを確認し、Hyper-V ロールを確認します。

    - WinRM サービスを有効にし、ホストのポート 5985（HTTP）および 5986（HTTPS）を開きます。これはメタデータ収集に必要です。

    - ホストでの PowerShell リモート処理を有効にします。

    - ホストによって管理されているすべての VM で Hyper-V 統合サービスが有効になっていることを確認します。

    - 必要に応じて、ホスト上で CredSSP を有効にします。

1. **az30308a-hv-vm** へのリモートデスクトップセッション内で **Windows PowerShell ISE** を起動します。 

1. 「**Administrator: Windows PowerShell ISE** ウィンドウのコンソール ペインで次のコマンドを実行して、Zone.Identifier 代替データ ストリームを削除します。この場合、ファイルがインターネットからダウンロードされたことを示しています。

   ```powershell
   Unblock-File -Path F:\MicrosoftAzureMigrate-Hyper-V.ps1
   ```

1. 「**Administrator: Windows PowerShell ISE** ウィンドウで、**F:** フォルダーにある **MicrosoftAzureMigrate-Hyper-V.ps1** スクリプトを開いて実行します。確認を求められたら、**Y** キーを押してから**Enter** キーを押します。ただし、次のプロンプトは例外であり、その場合は **N** をタイプし、 **Enter** キーを押します。

- Do you use SMB share(s) to store the VHDs?(SMB 共有を使用して VHD を保存していますか？)

- Do you want to create non-administrator local user for Azure Migrate and Hyper-V Host communication?(Azure Migrate と Hyper-V ホスト通信用の非管理者ローカルユーザーを作成しますか?)


#### タスク 2: Azure Migrate プロジェクトを作成する

1. **az30308a-hv-vm** へのリモートデスクトップセッション内で、Microsoft Edge を起動し、[Azure portal](https://portal.azure.com) に移動し、このラボで使用するサブスクリプションの所有者の役割を持つユーザー アカウントの認証情報を提供してサインインします。

1. Azure portal で「**Azure Migrate**」を検索して選択し、「**Azure Migrate**」 ブレードの 「**移行の目標**」 セクションで、「**Windows、Linux、SQL サーバー**」 を選択してから 「**プロジェクトの作成**」 を選択します。

1. 「**Azure Migrate**」 ブレードで、次の設定を指定し (他は既定値のままにします)、「**作成**」 を選択します。

    | 設定 | 値 | 
    | --- | --- |
    | サブスクリプション | このラボで使用する Azure サブスクリプションの名前 |
    | リソース グループ | 新しいリソース グループ **az30308b-labRG** の名前 |
    | プロジェクトを移行する | **az30308b-migrate-project** |
    | 地理的な場所 | あなたの国または地域の名前 |


#### タスク 3: ターゲット Azure 環境を実装する

1. Azure portal で、「**仮想ネットワーク**」 を検索して選択し、「**仮想ネットワーク**」 ブレードで、コマンド バーで、「**+ 追加**」 (または **「+作成」** ) を選択します。

1. 「**仮想ネットワークの作成**」 ブレードの 「**基本**」 タブで次の設定を指定し (他の設定はデフォルト値のままにします)、「**次へ:**」 を選択します**IP アドレス**:

    | 設定 | 値 |
    | --- | --- |
    | サブスクリプション | このラボで使用する Azure サブスクリプションの名前 |
    | リソース グループ | 新しいリソース グループ **az30308c-labRG** の名前 |
    | 名前 | **az30308c-migration-vnet** |
    | リージョン | このラボの前半で仮想マシンをデプロイした Azure リージョンの名前 |

1. **仮想ネットワークの作成**ブレードの **IP アドレス**タブで、**IPv4 アドレス スペース**テキスト ボックスに **10.8.0.0/16** と入力し、**+サブネットを追加**を選択します。

1. **サブネットを追加** ブレードで次の設定を指定（他の設定はデフォルト値のままにします）し、 **追加**を選択します：

    | 設定 | 値 |
    | --- | --- |
    | サブネット名 | **subnet0** |
    | サブネット アドレス範囲 | **10.8.0.0/24** |

1. 「**仮想ネットワークの作成**」 ブレードの 「**IP アドレス**」タブに戻り、「**確認および作成**」 を選択します。

1. 「**仮想ネットワークの作成**」 ブレードの 「**確認および作成**」 タブで、「**作成**」 を選択します。

1. Azure portal で、「**仮想ネットワーク**」 を検索して選択し、「**仮想ネットワーク**」 ブレードで、コマンド バーで、「**+ 追加**」 (または **「+作成」** ) を選択します。

1. 「**仮想ネットワークの作成**」 ブレードの 「**基本**」 タブで次の設定を指定し (他の設定はデフォルト値のままにします)、「**次へ:**」 を選択します**IP アドレス**:

    | 設定 | 値 |
    | --- | --- |
    | サブスクリプション | このラボで使用する Azure サブスクリプションの名前 |
    | リソース グループ | **az30308c-labRG** |
    | 名前 | **az30308c-test-vnet** |
    | リージョン | このラボの前半で仮想マシンにデプロイした Azure リージョンの名前 |

1. **仮想ネットワークの作成**ブレードの **IP アドレス**タブで、**IPv4 アドレス スペース**テキスト ボックスに **10.8.0.0/16** と入力し、**+サブネットを追加**を選択します。

1. **サブネットを追加** ブレードで次の設定を指定（他の設定はデフォルト値のままにします）し、 **追加**を選択します：

    | 設定 | 値 |
    | --- | --- |
    | サブネット名 | **subnet0** |
    | サブネット アドレス範囲 | **10.8.0.0/24** |

1. 「**仮想ネットワークの作成**」 ブレードの 「**IP アドレス**」タブに戻り、「**確認および作成**」 を選択します。

1. 「**仮想ネットワークの作成**」 ブレードの 「**確認および作成**」 タブで、「**作成**」 を選択します。

1. Azure portal で、「**ストレージ アカウント**」 を検索して選択し、「**ストレージ アカウント**」 ブレードで、コマンド バーで、「**+ 追加**」 (または **「+作成」** ) を選択します。

1. 「**ストレージ アカウントの作成**」ブレードの「**基本**」タブで、次のように設定を行います (他の設定は既定値のままにします)。

    | 設定 | 値 | 
    | --- | --- |
    | サブスクリプション | このラボで使用する Azure サブスクリプションの名前 |
    | リソース グループ | **az30308c-labRG** |
    | ストレージ アカウント名 | 文字と数字で構成される、長さが 3 から 24 のグローバルに一意の名前 |
    | 場所 | このタスクの前半で仮想ネットワークを作成した Azure リージョンの名前 |
    | 業績 | **Standard** |
    | アカウントの種類 | **StorageV2 (汎用 v2)** |
    | 冗長性 | **ローカル冗長ストレージ (LRS)** |

1. 「**ストレージ アカウントの作成**」 ブレードの 「**基本**」 タブで、「**確認および作成**」 を選択します。

1. 「**ストレージ アカウントの作成**」 ブレードの 「**確認および作成**」 タブで、「**作成**」 を選択します。


### 演習 2: Azure Migrate を使用して移行のために Hyper-V を評価する
  
この演習の主なタスクは次のとおりです。

1. Azure Migrate アプライアンスをデプロイし、構成する

1. 評価の構成、実行、および表示


#### タスク 1: Azure Migrate アプライアンスをデプロイし、構成する

1. **az30308a-hv-vm** へのリモート デスクトップ セッション内で、Azure portal の Microsoft Edge ウィンドウで「**Azure Migrate**」を検索して選択します。

1. **Azure Migrate \| 「Windows、Linux、SQL サーバー」** ブレードの 「**Azure Migrate: サーバー評価**」 タイルで 「**検出**」 を選択します。 

1. 「**検出**」 ブレードの 「**サーバーは仮想化されていますか?**」 ドロップダウン リストで、「**はい、Hyper-V を使用します**」 を選択します。 

1. 「**検出**」 ブレードの 「**アプライアンスに名前を付ける**」 テキスト ボックスで「**az30308a-vma1**」と入力し、「**キーの生成**」 を選択します。

   >**注**: Azure Migrate プロジェクト キーの生成中にアクセス許可に関連するエラーが発生した場合は、Azure portal で 「**サブスクリプション**」 ブレードに移動し、お使いになっているサブスクリプションを選択します。そのサブスクリプション ブレードで 「**アクセス制御 (IAM)**」 を選択し、Azure AD ユーザー アカウントに**所有者**の役割を割り当てます。

1. リソースのプロビジョニングが完了するのを待ちます。**az30308a-hv-vm** へのリモート デスクトップ セッションでメモ帳を起動し、**Azure Migrate プロジェクト キー**をメモ帳にコピーします。 

1. 「**検出**」 ブレードの 「**Azure Migrate アプライアンスのダウンロード**」 テキスト ボックスで、「**VHD ファイル**」 オプションを選択し、「**ダウンロード**」 を選択します。プロンプトが表示されたら、ダウンロードの場所を **F:\VMs** フォルダーに設定します。

   >**注**: ダウンロードが完了するまで待ってください。5 分ほどかかる場合があります。

1. ダウンロードが完了したら、ダウンロードした .ZIP ファイルのコンテンツを **F:\VMs** フォルダーに抽出します。 

1. **az30308a-hv-vm** へのリモート デスクトップ セッション内で、**Hyper-V Manager** コンソールに切り替え、**az30308a-hv-vm** ノードを選択し、「**Import Virtual Machine**」 を選択します。これにより、**Import Virtual Machine** ウィザードが起動します。

1. **Import Virtual Machine** ウィザードの 「**Before You Begin**」 ページで、「**Next >**」 をクリックします。

1. 「**Import Virtual Machine**」 ウィザードの 「**Locate Folder**」 ページで、抽出した**Virtual Machinesフォルダー**の場所を指定し、「**Next >**」 を選択します。

1. **Import Virtual Machine** ウィザードの 「**Select Virtual Machine**」 ページで、「**Next >**」 を選択します:

1. **Import Virtual Machine** ウィザードの 「**Choose Import Type**」 ページで、「**Register the virtual machine in place (use the existing unique ID)**」 を選択し、「**Next >**」を選択します。

1. **Import Virtual Machine** ウィザードの**Configure Processor**ページで、**Number of virtual processors**を **4** に設定し、「**Next >**」 を選択します。

   > **注**: 仮想プロセッサーの数の変更に関連するエラー メッセージは無視してください。

1. **Import Virtual Machine** ウィザードの、**Connect Network**ページの**Connection**ドロップダウン リストで **NestedSwitch** を選択し、「**Next >**」 を選択します。

1. **Import Virtual Machine** ウィザードの**Completing Import Wizard**ページで、「**Finish**」 を選択します。

   >**注**: インポートが完了するまで待ちます。10 分ほどかかる場合があります。

1. **Hyper-V Manager** コンソールで、新しくインポートした仮想マシンを選択し、「**Rename**」 を選択して、名前を **az30308a-vma1** に設定します。

1. **Hyper-V Manager** コンソールの中で、新しくインポートした仮想マシンを選択し、 **Start**を選択します。 

1. 「**Hyper-V Manager**」 コンソールで、仮想マシンが実行されていることを確認し、「**Connect**」 を選択します。 

1. 仮想アプライアンスへの仮想マシン接続ウィンドウで、**License terms** ページで、**Accept**を選択します。 

1. 仮想アプライアンスへの仮想マシン接続ウィンドウで、**Customize settings** ページで、ビルトイン管理者アカウントのパスワードを **Pa55w.rd1234** に設定し、 **Finish**を選択します。 

1. 仮想アプライアンスへの 「仮想マシン接続」 ウィンドウで、新しく設定したパスワードを使用してサインインします。

1. 仮想アプライアンスへの仮想マシン接続ウィンドウ内で、Windows PowerShell を起動し、次のコマンドを実行して IP アドレスを特定します。

   ```powershell
   (Get-NetIPAddress).IPAddress
   ```

1. **az30308a-hv-vm** へのリモート デスクトップ セッション内で、Microsoft Edge ウィンドウを起動し、[https://`IPaddress`:44368](https://`IPaddress`:44368) に移動します。ここで、` Ipaddress` プレースホルダーは、前のステップで識別した IP アドレスを表します。

   >**注**: Web サイトのセキュリティ証明書に関する警告は無視してください。 

1. プロンプトが表示されたら、次の認証情報を入力します。

    | 設定 | 値 | 
    | --- | --- |
    | ユーザー名 | **administrator** |
    | パスワード | **Pa55w.rd1234** |

1. Microsoft Edge ウィンドウの 「**Appliance Configuration Manager**」 ページで 「**I agree**」 ボタンを選択し、前提条件が検証されるまで待ちます。 

1. Microsoft Edge ウィンドウの 「**Appliance Configuration Manager**」 ページの 「**Register with Azure Migrate**」 セクションの 「**Provide Azure Migrate project key**」 テキスト ボックスで、この演習の前半でメモ帳にコピーしたキーを貼り付け、「**Login**」 を選択します。表示される既定のコードを受け入れ、クリップボードにコピーし、「**Copy code and login**」 を選択します。ブラウザーの 「**Enter code**」 ペインにクリップボードにコピーしたコードを貼り付け、「**Next**」 を選択します。このラボで使用しているサブスクリプションの所有者ロールのユーザー アカウントの資格情報でサインインし、ブラウザーのページを閉じます。 

1. Microsoft Edge ウィンドウ内の 「**Appliance Configuration Manager**」 ページで、登録が成功したことを確認します。

1. Microsoft Edge ウィンドウの 「**Appliance Configuration Manager** 」 ページの 「**Manage credentials and discovery sources**」 セクションで、「**Add credentials**」 を選択します。「**Add credentials**」 ウィンドウで、次の設定を指定して 「**Save**」 を選択します。

    | 設定 | 値 | 
    | --- | --- |
    | フレンドリ名 | **az30308acreds** |    
    | ユーザー名 | **Student** |
    | パスワード | **Pa55w.rd1234** |

1. Microsoft Edge ウィンドウ内で、「**Appliance Configuration Manager**」 ページの 「**Provide Hyper-V host/cluster details**」 セクションで 「**Add discovery source**」 を選択します。「**Add discovery source**」 ペインで 「**Add single item**」 オプションを選択し、「**Discovery source**」 ドロップダウン リストが 「**Hyper-V Host/Cluster**」 に設定されていることを確認します。「**Friendly name**」 ドロップダウン リストで 「**az30308acreds**」 エントリを選択します。「**IP address /FQDN**」 テキスト ボックスで「**10.0.2.1**」と入力し、「**Save**」 を選択します。 

1. Microsoft Edge ウィンドウ内で、「**Appliance Configuration Manager**」 ページの 「**Provide Hyper-V host/cluster details**」 セクションで、「**Start discovery**」 を選択します。 

   >**注**: 一般に、検出されたサーバーのメタデータが Azure portal に表示されるまで、ホストごとに約 15 分かかる場合があります。


#### タスク 2: 評価の構成、実行、および表示

1. **az30308a-hv-vm** へのリモート デスクトップ セッション内で、Azure portal を表示している Microsoft Edge ウィンドウで、「**Azure Migrate \| Windows、Linux、SQLサーバー**」 ブレードに戻り、「**更新**」 を選択し、「**Azure Migrate: Discovery and assessment**」 タイルで、「**評価**」 ドロップダウンを開き、「**Azure VM**」 を選択します。

   >**注**: 場合によっては、もう一度ページを更新する必要があります。 

1. 「**評価プロパティ**」 ブレードで、「**編集**」 を選択し、次の設定を指定し (他は既定値のままにします)、「**保存**」 を選択します。

    | 設定 | 値 | 
    | --- | --- |
    | ターゲットの場所 | このラボのために使用する Azure リージョン名 |
    | ストレージの種類 | **自動** |
    | 予約インスタンス | **予約インスタンスはありません** |
    | サイズ変更の設定基準 | **オンプレミス** |
    | VM シリーズ | **Dsv3_series** |
    | 快適性係数 | **1** |
    | プラン | **従量課金制** |
    | 通貨 | 米ドル ($) | 
    | 割引 | **0** |
    | VM のアップタイム | 1 か月あたり **31** 日、1 日あたり **24** 時間| 

   >**注**: ラボ環境に固有の限られた時間を考慮すると、この場合の唯一の実行可能なオプションは、**オンプレミスとして** の評価です。 

1. 「**評価の作成**」 ブレードに戻り、「**次へ**」 を選択し、「**評価するサーバーの選択**」 タブに移動します。

1. 「**評価名**」 を 「**az30308a-assessment**」 に設定します。

1. 「**新規作成**」 オプションが選択されていることを確認し、グループ名を「**az30308a-assessment-group**」に設定します。グループに追加するマシンのリストで 「**az30308a-vm1**」 を選択します。

1. 「**次へ**」 をクリックしてから、「**評価の作成**」 をクリックします。 

1. 「**Azure Migrate \| Windows、Linux、SQLサーバー** ブレードで、「**更新**」 を選択し、「**Azure Migrate: サーバーの評価**」 タイルで、「**評価**」 行に「**1**」のエントリが含まれていることを確認してから、それを選択します。

1. 「**Azure Migrate: Server Assessment \| 評価**」 ブレードで、新しく作成した評価 **az30308a-assessment** を選択します。 

1. 「**az30308a-assessment**」 ブレードで、コンピューティングとストレージの両方に対する Azure 対応性と毎月のコストの見積もりを示す情報を確認します。 

   >**注**: 実際のシナリオでは、依存関係エージェントのインストールを検討して、評価段階でサーバーの依存関係を詳しく分析する必要があります。


### 演習 3: Azure Migrate を使用して Hyper-V VM を移行する
  
この演習の主なタスクは次のとおりです。

1. Hyper-V VM の移行の準備

1. Hyper-V VM のレプリケーションを構成する

1. Hyper-V VM の移行を実行する

1. ラボでデプロイされている Azure リソースを削除する


#### タスク 1: Hyper-V VM の移行の準備

1. **az30308a-hv-vm** へのリモート デスクトップ セッション内で、Azure portal を表示している Microsoft Edge ウィンドウで、「**Azure Migrate \| Windows、Linux、SQL サーバー**」 ブレードに戻ります。 

1. **Azure Migrate \| Windows、Linux、SQL サーバー** ブレードで、「**更新**」 を選択し、「**Azure Migrate: Server Migration**」 タイルで、「**検出**」 リンクを選択します。 

1. 「**検出**」 ブレードで、次の設定を指定し (他の設定は既定値のままにします)、「**リソースの作成**」 を選択します。

    | 設定 | 値 | 
    | --- | --- |
    | マシンは仮想化されていますか?  | **はい、Hyper-V を使用します** |
    | ターゲット リージョン | このラボのために使用する Azure リージョン名 | 
    | 移行先のターゲット リージョンを確認する | 選択済み |

    >**注**: この手順により、Azure Site Recovery コンテナーのプロビジョニングが自動的にトリガーされます。

1. 「**マシンの検出**」 ブレードの 「**1. Hyper-V ホスト サーバーを準備する**」 ステップで、最初の**ダウンロード** リンク (「ダウンロード」 ボタンではありません) を選択し、Hyper-V レプリケーション プロバイダー ソフトウェア インストーラーをダウンロードします。

1. プロンプトが表示されたら、「**AzureSiteRecoveryProvider.exe**」 を起動します。これによって **Azure Site Recovery Provider Setup（Hyper-V server）** ウィザードが起動します。

1. 「**Microsoft Update**」 ページで、「**Off**」 を選択して 「**Next**」 を選択します。

1. 「**Provider installation**」 ページで、「**Install**」 を選択します。

1. Azure portal に切り替え、「**マシンの検出**」 ブレードで、コンテナー登録キーをダウンロードするため、オンプレミスの Hyper-V ホストを準備するためのステップ 1 で**ダウンロード** ボタンを選択します。プロンプトが表示されたら、登録キーを 「**ダウンロード**」 フォルダーに保存します。

1. 「**Provider installation**」 ページに切り替えて、「**Register**」 を選択します。これによって、「**Microsoft Azure Site Recovery Registration Wizard**」 が起動します。

1. 「**Microsoft Azure Site Recovery Registration Wizard**」 の 「**Vault Settings**」 のページで、「**Browse**」 を選択して 「**Downloads**」 フォルダーを表示し、Vault 認証情報ファイルを選択して 「**Open**」 を選択します。

1. 「**Microsoft Azure Site Recovery Registration Wizard**」 の 「**Vault Settings**」 ページに戻り、「**Next**」 を選択します。

1. 「**Microsoft Azure Site Recovery Registration Wizard**」 の 「**Proxy Settings**」 ページで、既定値の設定を承諾して 「**Next**」 を選択します。

1. 「**Microsoft Azure Site RecoveryRegistration Wizard**」 の 「**Registration**」 ページで、 「**Finish**」 を選択します。

1. 登録プロセスが完了したら、Edgeの「**検出**」 ブレードで 「**登録の最終処理**」 を選択します。

   >**注**: 「**検出**」 ブレードを表示しているブラウザーのページを更新して、そこに戻る必要があるかもしれません。

   >**注**: 仮想マシンの検出が完了するまでに最大 15 分かかる場合があります。


#### タスク 2: Hyper-V VM のレプリケーションを構成する

1. 登録完了の確認を受信したら、「**Azure Migrate
\| Windows、Linux、SQL サーバー**」 ブレードで、「**更新**」 を選択し、「**Azure Migrate: Server Migration**」 タイルで、「**レプリケート**」 リンクを選択します。 

1. 「**レプリケート**」 ブレードの 「**ソース設定**」 ページの 「**マシンは仮想化されていますか?**」 ドロップダウン リストで、「**はい、 Hyper-V を使用します**」 を選択し、「**次へ**」をクリックします。  

1. 「**レプリケート**」 ブレードの 「**仮想マシン**」 ページで、次の設定を指定し (他は既定値のままにします)、 「**次へ**」 を選択します**Next: ターゲット設定**:

    | 設定 | 値 | 
    | --- | --- |
    | Azure Migrate 評価から移行設定をインポートする | **はい、Azure Migrate 評価から移行設定を適用** |
    | グループの選択 | **az30308a-assessment-group** |
    | 評価の選択 | **az30308a-assessment** |
    | 仮想マシン | **az30308a-vm1** |

1. 「**ターゲット設定**」 ページで、次の設定を指定し (他は既定値のままにします)、 「**次へ**」 を選択します:

    | 設定 | 値 | 
    | --- | --- |
    | サブスクリプション | このラボで使用する Azure サブスクリプションの名前 |
    | リソース グループ | **az30308c-labRG** |
    | レプリケーション ストレージ アカウント | このラボで前に作成したストレージ アカウントの名前 | 
    | 仮想ネットワーク | **az30308c-migration-vnet** |
    | サブネット | **subnet0** |

1. 「**レプリケート**」 ブレードの 「**コンピューティング**」 ページで、「**Azure VM サイズ**」 ドロップダウン リスト内で 「**Standard_D2s_v3**」 が選択されていることを確認します。「**OSの種類**」 ドロップダウン リストで 「**Windows**」 を選択し、「**次へ**」 を選択します。  

1. 「**レプリケート**」 ブレードの 「**ディスク**」 ページで、デフォルト設定を受け入れ、「**次へ**」 を選択します。  

1. 「**レプリケーション**」 ブレードの 「**レプリケーションの確認と開始**」　ページで、「**レプリケート**」 を選択します。  

1. レプリケーションのステータスを監視するには、「**Azure Migrate \| Windows、Linux、SQL サーバー**」 ブレードで、「**更新**」 を選択し、「**Azure Migrate: Server Migration**」 タイルで 「**サーバーをレプリケートしています**」 エントリを選択します。「**Azure Migrate: Server Migration \| マシンのレプリケート**」 で、レプリケート中のマシンのリストの 「**状態**」 列を調べます。

1. ステータスが 「**プロテクト**」 に変わるまで待ちます。これはおよそ 15 分間かかるかもしれません。


#### タスク 3: Hyper-V VM の移行を実行する

1. Azure portal の 「**Azure Migrate: Server Migration \| マシンのレプリケート**」 で、**az30308a-vm1** 仮想マシンを表すエントリを選択します。

1. **az30308a-vm1** 仮想マシンのレプリケート ブレードで、「**テスト移行**」 を選択します。

1. 「**移行テスト**」 ブレードの 「**仮想ネットワーク**」 ドロップダウン リストで、「**az30308c-test-vnet**」 および 「**テスト移行**」 を選択します。

   >**注**: テスト移行が完了するまで待ちます。5 分ほどかかる場合があります。

1. Azure portal で、**Virtual Machine** を検索して選択し、**「Virtual Machines」** ブレード、新しくプロビジョニングされた仮想マシン **az30308a-vm1-test** を表すエントリを確認します。

1. Azure portal で、「**Azure Migrate: Server Migration \| マシンのレプリケート**」 に戻り、画面を定期的にリフレッシュして、**az30308a-vm1** 仮想マシンが「**テストのフェールオーバーのクリーンアップが保留中**」状態で表示されることを確認します。

1. 「**Azure Migrate: Server Migration \| マシンのレプリケート**」 ブレードで、**az30308a-vm1** 仮想マシンを表すエントリを選択します。

1. 「**az30308a-vm1** マシンのレプリケート」 ブレードで、「**テスト移行をクリーンアップする**」 を選択します。

1. 「**テスト移行のクリーンアップ**」 ブレードで、「**テストが完了しました**」 チェックボックスを選択します。**テスト仮想マシンを削除**して、「**テストのクリーンアップ**」 を選択します。

1. テスト フェールオーバー クリーンアップ ジョブが完了したら、**az30308a-vm1** マシンのレプリケート ブレードを表示するブラウザー ページを更新して、ツールバーの**移行**アイコンが自動的に利用可能になったことを確認します。

1. **az30308a-vm1** マシンのレプリケート ブレードで、「**移行**」 リンクを選択します。 

1. 「**移行**」 ブレードの、「**データの損失を最小限に抑えるため、移行前にマシンをシャットダウンしますか?**」 ドロップダウン リストで、「**はい**」 を選択してから、「**az30308a-vm1**」 エントリの横にあるチェックボックスを選択した後、「**移行**」 を選択します。

1. 移行のステータスを監視するには、「**Azure Migrate \| サーバー**」 ブレードの 「**Azure Migrate: Server Migration**」 タイルで 「**サーバーのレプリケート**」 エントリを選択します。「**Azure Migrate: Server Migration \| マシンのレプリケート**」 で、レプリケート中のマシンのリストの 「**状態**」 列を調べます。ステータスが「**予定されていたフェールオーバーが終了しました**」になっていることを確認します。

   >**注**: 移行は不可逆的なアクションであると想定されています。完全な情報を確認する場合は、「Azure Migrate \| Windows、Linux、SQL サーバー」 ブレードに戻り、「Azure Migrate: Server Migration」 タイルの 「移行済みサーバー」 の値が 1 であることを確認します。


#### タスク 4: ラボでデプロイされている Azure リソースを削除する

1. **az30308a-hv-vm** へのリモート デスクトップ セッション内の Azure portal を表示しているブラウザーの画面で、Cloud Shell ペイン内で PowerShell セッションを起動します。

1. 「Cloud Shell」 ウィンドウで、次の手順を実行して、この演習で作成したリソース グループを一覧表示します。

   ```powershell
   Get-AzResourceGroup -Name 'az30308*'
   ```

   > **注**: このラボで作成したリソース グループのみが出力に含まれていることを確認します。このグループは、このタスクで削除されます。

1. 「Cloud Shell」 ウィンドウから次を実行して、このラボで作成したリソース グループを削除します

   ```powershell
   Get-AzResourceGroup -Name 'az30308*' | Remove-AzResourceGroup -Force -AsJob
   ```

1. 「Cloud Shell」 ペインを閉じます。
