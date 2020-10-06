---
lab:
    title: '6: Azure SQL Database ベースのアプリケーションの実装'
    module: 'モジュール 6: データベースのソリューションを設計する'
---

# ラボ: Azure SQL Database ベースのアプリケーション を実装する
# 受講生用ラボ マニュアル

## ラボ シナリオ

Adatum 社には、.NET Core ベースのフロントエンドと SQL Server ベースのバックエンドを備えた 2 層アプリケーションがあります。Adatum エンタープライズ アーキテクチャ チームは、Azure SQL データベースをデータ層として活用できるように、これらのアプリケーションを実装する可能性を模索しています。既存の SQL Server バックエンドの断続的で予測不可能な使用と、フロントエンド アプリに組み込まれた待機時間に対する比較的高い許容度を考慮して、Adatum は Azure SQL データベースのサーバーレス層を検討しています。 

サーバーレスは、個々の Azure SQL データベース インスタンスのコンピューティング層であり、ワークロードの需要に基づいてコンピューティングを自動的に拡大し、1 秒あたりのコンピューティング使用料金を請求します。サーバーレス コンピューティング層は、ストレージのみが請求される非アクティブ期間にはデータベースを自動的に一時停止し、アクティビティが戻ったときにデータベースを自動的に再開することもできます。

Adatum エンタープライズ アーキテクチャ チームは、サイト間 VPN または ExpressRoute を介したハイブリッド接続に依存することなく、オンプレミス ロケーションからアプリが接続できるシナリオとして、特定の範囲の IP アドレスから受信接続を確実に制限できるようにするために、Azure SQL データベースによって提供されるネットワーク レベルのセキュリティ評価にも関心があります。 
  
これらの目標を達成するために、Adatum アーキテクチャ チームは、次のような Azure SQL データベース アプリケーションをテストします。

-  Azure SQL データベースでのサーバーレス層の実装

-  データ ストアとして Azure SQL データベースを使用する .NET Core コンソール アプリの実装


## 目標
  
このラボを終了すると、下記ができるようになります。

-  Azure SQL Database のサーバーレス層を実装する

-  データ ストアとして Azure SQL Database を使用する .NET Core ベースのコンソール アプリを構成する


## ラボ環境
  
Windows Server 管理者の認証資格情報

-  ユーザー名: **Student**

-  パスワード: **Pa55w.rd1234**

予想時間: 60 分


## ラボ ファイル

-  なし


### 演習 1: Azure SQL Database のプロビジョニング
  
この演習の主なタスクは次のとおりです。

1. Azure SQL データベースの作成 

1. Azure SQL データベースに接続してクエリを実行する


#### タスク 1: Azure SQL データベースの作成 

1. ラボのコンピューターから Web ブラウザーを起動し、 [Azure portal](https://portal.azure.com) に移動し、このラボで使用するサブスクリプションの所有者の役割を持つユーザーアカウントの認証情報を提供してサインインします。

1. Azure portal で、**SQL Database**を検索して選択し、「**SQL データベース**」 ブレードで、「**+ 追加**」 を選択します。

1. 「**SQL Database の作成**」 ブレードの 「**基本**」 タブで、次の設定を指定します (他の設定は既定値のままにします)。

    | 設定 | 値 | 
    | --- | --- |
    | サブスクリプション | このラボで使用する Azure サブスクリプションの名前 |
    | リソース グループ | 新しいリソース グループ **az30303a-LabRG** の名前 |
    | データベース名 | **az30303a-db1** | 

1. 「**サーバ**」 ドロップダウン リストのすぐ下で、「**新規作成**」 を選択し、「**新しいサーバー**」 ブレードで、次の設定を指定して、「**OK**」 を選択します ( 他の設定は既定値のままにします)。

    | 設定 | 値 | 
    | --- | --- |
    | サーバー名 | グローバルにユニークな任意の名前 | 
    | サーバー管理者のログイン | **sqladmin** |
    | パスワード | **Pa55w.rd1234** |
    | 場所 | SQL データベースをプロビジョニングできる Azure リージョンの名前 |
    | Azure サービスにサーバーへのアクセスを許可する | **チェックボックスを選択する** |

1. 「**コンピューティング + ストレージ**」 ラベルの横で、「**データベースを構成する**」 リンクを選択します。

1. 「**構成**」 ブレードで、「**サーバーレス**」 を選択し、対応するハードウェア構成と自動一時停止遅延設定を確認し、「**自動一時停止を有効にする**」 チェックボックスをオンにし、「**適用する**」 を選択します。

1. 「**SQL Database の作成**」 ブレードの 「**基本**」 タブに戻り、「**次へ: ネットワーク >** を選択します。 

1. 「**SQL Database の作成**」 ブレードの 「**ネットワーク**」 タブ で、次の設定を指定します (他の設定は既定値のままにします)。

    | 設定 | 値 | 
    | --- | --- |
    | 接続方法 | **パブリック エンドポイント** |    
    | Azure サービスとリソースにサーバーへのアクセスを許可する | **いいえ** |
    | 現在のクライアント IP アドレスを追加する | **はい** |

1. 「**次へ:**」 を選択します。**追加設定 >**」 を選択します。 

1. 「**SQL Database の作成**」 ブレードの 「**追加の設定**」 タブで、次の設定を指定します (他の設定は既定値のままにします)。

    | 設定 | 値 | 
    | --- | --- |
    | 既存のデータを使用する | **サンプル** |
    | Advanced Data Security を有効にする | **今すぐではない後で** |

1. 「**Review + create**」 を選択し、「**作成**」 を選択します。 

    >**注**: SQL Database が作成されるのを待ちます。プロビジョニングにはおよそ 2 分かかる場合があります。 


#### タスク 2: Azure SQL データベースに接続してクエリを実行する

1. Azure portal で、**SQL Database** を検索して選択し、「**SQL Database**」 ブレードで、新しく作成された **az30303a-db1** Azure SQL データベースを表すエントリを選択します。

1. 「SQL Database」 ブレードで、「**クエリ エディター (プレビュー)**」 を選択します。

1. 「**SQL Server の認証**」 セクションの 「**パスワード**」 テキストボックスに、「**Pa55w.rd1234**」を入力し、「**OK**」 を選択します。

1. 「**クエリエディター (プレビュー)**」 ペインの 「**クエリ 1**」 タブで、次のクエリを入力し、「**実行**」 を選択します。

    ```SQL
    SELECT TOP 20 pc.Name as CategoryName, p.name as ProductName
    FROM SalesLT.ProductCategory pc
    JOIN SalesLT.Product p
    ON pc.productcategoryid = p.productcategoryid;
    ```

1. 「**結果**」 タブを確認して、クエリが正常に完了したことを確認します。


### 演習 2: データ ストアとして Azure SQL データベースを使用する .NET Core コンソール アプリを実装する
  
この演習の主なタスクは次のとおりです。

1. Azure SQL データベースの ADO.NET 接続情報を特定する

1. .NET コア コンソール アプリを作成して構成する

1. .NET Core コンソール アプリをテストする

1. SQL データベース ファイアウォールを構成する

1. .NET Core コンソール アプリの機能を確認する

1. ラボにデプロイした Azure リソースを削除する


#### タスク 1: Azure SQL データベースの ADO.NET 接続情報を特定する

1. Azure portal 内の、前の演習でデプロイした Azure SQL データベースのブレードの 「**設定**」 セクションで、「**接続設定**」 を選択します。

1. 「**ADO.NET**」 タブで、SQL 認証の ADO.NET 接続文字列をメモします。


#### タスク 2: .NET コア コンソール アプリを作成して構成する

1. Azure portal で、検索テキストボックスのすぐ右にあるツールバー アイコンを選択して、「**Cloud Shell**」 ペインを表示します。

1. **Bash** または **PowerShell** のいずれかを選択するためのプロンプトが表示されたら、「**Bash**」 を選択します。 

    >**注**: **Cloud Shell** を初めて起動し、「**ストレージがマウントされていません**」というメッセージが表示された場合は、このラボで使用しているサブスクリプションを選択し、「**ストレージの作成**」を選択します。 

1. Cloud Shell ペインから次のコマンドを実行して、**az30303a1** という名前の新しいフォルダーを作成し、それを現在のディレクトリとして設定します。

   ```sh
   mkdir az30303a1
   cd az30303a1/
   ```

1. Cloud Shell ペインから次のコマンドを実行して、デスクトップ テンプレートに基づく.NET Core ベースのアプリの新しいアプリ プロジェクト ファイルを作成します。

   ```sh
   dotnet new console
   ```

1. Cloud Shell ペインで、組み込みのエディターを使用して、**az30303a1.csproj** ファイルを開き、`<Project>` タグの間に次の XML 要素を追加してファイルを変更します。 

   ```xml
   <ItemGroup>
       <PackageReference Include="System.Data.SqlClient" Version="4.6.0" />
   </ItemGroup>
   ```

1. **az30303a1.csproj** ファイルを保存して閉じます。

1. Cloud Shell ペインで、組み込みのエディターを使用して、**Program.cs** ファイルを開き、内容を次のコードに置き換えてファイルを変更します。 

   ```cs
   using System;
   using System.Data.SqlClient;
   using System.Text;

   namespace sqltest
   {
       class Program
       {
           static void Main(string「] args)
           {
               try 
               { 
                   SqlConnectionStringBuilder builder = new SqlConnectionStringBuilder();
                   builder.ConnectionString="<your_ado_net_connection_string>";

                   using (SqlConnection connection = new SqlConnection(builder.ConnectionString))
                   {
                       Console.WriteLine("\nQuery data example:");
                       Console.WriteLine("=========================================\n");
        
                       connection.Open();       
                       StringBuilder sb = new StringBuilder();
                       sb.Append("SELECT TOP 20 pc.Name as CategoryName, p.name as ProductName ");
                       sb.Append("FROM [SalesLT].[ProductCategory] pc ");
                       sb.Append("JOIN [SalesLT].[Product] p ");
                       sb.Append("ON pc.productcategoryid = p.productcategoryid;");
                       String sql = sb.ToString();

                       using (SqlCommand command = new SqlCommand(sql, connection))
                       {
                           using (SqlDataReader reader = command.ExecuteReader())
                           {
                               while (reader.Read())
                               {
                                   Console.WriteLine("{0} {1}", reader.GetString(0), reader.GetString(1));
                               }
                           }
                       }                    
                   }
               }
               catch (SqlException e)
               {
                   Console.WriteLine(e.ToString());
               }
               Console.WriteLine("\nDone. Press enter.");
               Console.ReadLine(); 
           }
       }
   }
   ```

1. エディター ウィンドウは開いたままにします。 

1. Azure portal のブレードの **az30303a-db1** データベースの接続文字列を表示するブレードで、ADO.NET 接続文字列をコピーします。 

1. エディター ウィンドウに戻り、プレースホルダー `<your_ado_net_connection_string> ` を、前の手順でコピーした接続文字列の値に置き換えます。

1. エディター ウィンドウにコピーした接続文字列で、プレースホルダー `{your_password}` を **Pa55w.rd1234** に置き換えます。

1. **Program.cs** ファイルを保存して閉じます


#### タスク 3: .NET Core コンソール アプリをテストする

1. Cloud Shell ペインから次のコマンドを実行して、新しく作成された .NET Core ベースのコンソール アプリをコンパイルします。

   ```sh
   dotnet restore
   ```

1. Cloud Shell ペインから次のコマンドを実行して、新しく作成された .NET Core ベースのコンソール アプリを実行します。

   ```sh
   dotnet run
   ```

1. コンソール アプリを実行するとエラーが発生することに注意してください。 

    >**注**: Cloud Shell セッションを実行している仮想マシンに割り当てられた IP アドレスからの接続は明示的に許可する必要があるため、これは予想されたものです。


#### タスク 4: SQL データベース ファイアウォールを構成する

1. Cloud Shell ペインから次の操作を実行して、Cloud Shell セッションを実行している仮想マシンのパブリック IP アドレスを識別します。

   ```sh
   curl -s checkip.dyndns.org | sed -e 's/.*Current IP Address: //' -e 's/<.*$//'
   ```

1. Azure portal 内の **az30303a-db1** データベースの接続文字列を表示するブレードで、「**概要**」 を選択し、ツールバーで 「**サーバーのファイアウォールを設定する**」 を選択します。

1. 「**ファイアウォール**」 ブレードで、次のエントリを変更し、「**保存**」 を選択します。

    | 設定 | 値 | 
    | --- | --- |
    | ルール名 | **cloudshell** |
    | 開始 IP | このタスクで以前に特定した IP アドレス |
    | 終了 IP | このタスクで以前に特定した IP アドレス |

    >**注**: Cloud Shell セッションを再起動すると IP アドレスが変更されるため、これは明らかにラボの目的でのみ使用することを意図しています。


#### タスク 5: .NET Core コンソール アプリの機能を確認する

1. Cloud Shell ペインから次のコマンドを実行して、新しく作成された .NET Core ベースのコンソール アプリを実行します。

   ```sh
   dotnet run
   ```

1. 今回はコンソール アプリの実行が成功し、Azure portal の SQL Database ブレード内のクエリ エディターに表示される結果と同じ結果が返されることに注意してください。 


#### タスク 6: ラボにデプロイした Azure リソースを削除する

1. 「Cloud Shell」 ウィンドウから次のコマンドを実行して、この演習で作成したリソース グループを一覧表示します。

   ```sh
   az group list --query "[?starts_with(name,'az30303')]".name --output tsv
   ```

    > **注**: このラボで作成したリソース グループのみが出力に含まれていることを確認します。このグループは、このタスクで削除されます。

1. 「Cloud Shell」 ウィンドウから次を実行して、このラボで作成したリソース グループを削除します

   ```sh
   az group list --query "[?starts_with(name,'az30303')]".name --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

1. 「Cloud Shell」 ウィンドウを閉じます。
