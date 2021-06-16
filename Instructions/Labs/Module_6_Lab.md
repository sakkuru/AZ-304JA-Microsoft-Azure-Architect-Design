---
lab:
    title: '6: Azure SQL Database ベースのアプリケーションを実装する'
    module: 'モジュール 6: データベースのソリューションを設計する'
---

# ラボ: Azure SQL Database ベースのアプリケーションを実装する
# 受講生用ラボ マニュアル

## ラボ シナリオ

Adatum 社には、.NET コア ベースのフロントエンドと SQL Server ベースのバックエンドを持つ 2 層のアプリケーションがあります。Adatum エンタープライズ アーキテクチャ チームは、Azure SQL Database をデータ層として活用することで、これらのアプリケーションを実装する可能性を探っています。既存の SQL Server バックエンドの断続的な使用、予測不能な使用、およびフロントエンド アプリに組み込まれている待ち時間に対する比較的高い許容度を考慮すると、Adatum は Azure SQL データベースのサーバーレス層を検討しています。 

サーバーレスは、ワークロードの需要と 1 秒あたりのコンピューティングの請求に基づいてコンピューティングを自動的にスケーリングする、個々の Azure SQL Database インスタンスのコンピューティング レベルです。サーバーレス コンピューティング層は、ストレージのみが課金される非アクティブ期間中にデータベースを自動的に一時停止し、アクティビティが戻ったときにデータベースを自動的に再開することもできます。

Adatum エンタープライズ アーキテクチャ チームは、サイト間 VPN または ExpressRoute を介したハイブリッド接続に依存することなく、アプリがオンプレミスの場所から接続できる必要があるシナリオで、特定の範囲の IP アドレスへの受信接続を確実に休止できるようにするために、Azure SQL Database が提供するネットワーク レベルのセキュリティの評価にも関心を持っています。 
  
これらの目標を達成するために、Adatum アーキテクチャ チームは、次のような Azure SQL データベース ベースのアプリケーションをテストします。

-  Azure SQL データベースのサーバーレス層の実装

-  Azure SQL データベースをデータ ストアとして使用する .NET Core コンソール アプリの実装


## 目標
  
このラボを終了すると、次のことができるようになります。

-  Azure SQL Database のサーバーレス層を実装する

-  データ ストアとして Azure SQL Database を使用する .NET Core ベースのコンソール アプリを構成する


## ラボ環境
  
Windows サーバー管理者の資格情報

-  ユーザー名: **Student**

-  パスワード: **Pa55w.rd1234**

推定時間: 60 分


## ラボ ファイル

-  なし


### 演習 1: Azure SQL Database のプロビジョニング
  
この演習の主なタスクは次のとおりです。

1. Azure SQL データベースの作成 

1. Azure SQL データベースへ接続しクエリする


#### タスク 1: Azure SQL データベースの作成 

1. ラボ コンピューターから、Web ブラウザーを起動し、 [[Azure portal]](https://portal.azure.com) に移動し、このラボで使用するサブスクリプションの所有者ロールを持つユーザー アカウントの資格情報を指定してサインインします。

1. Azure portal で **SQL Database** を検索して選択し、**「SQL Database」** ブレードで **「+ 追加」** を選択します。

1. 「**SQL Database の作成**」 ブレードの 「**基本**」 タブで、次の設定を指定します (既定値を使用して他の設定を指定します)。

    | 設定 | 値 | 
    | --- | --- |
    | サブスクリプション | このラボで使用する Azure サブスクリプションの名前 |
    | リソース グループ | 新しいリソース グループ名 **az30303a-LabRG** |
    | データベース名 | **az30303a-db1** | 

1. 「**サーバー**」 ドロップダウン リストのすぐ下にある 「**新規作成**」 ブレードで、「**新しいサーバー**」 を選択し、次の設定を指定して 「**OK**」 を選択します (他のユーザーは既定値のままにします)。

    | 設定 | 値 | 
    | --- | --- |
    | サーバー名 | 有効で、グローバルにユニークな任意の名前 | 
    | サーバー管理者のログイン | **sqladmin** |
    | パスワード | **Pa55w.rd1234** |
    | 場所 | SQL データベースをプロビジョニングできる Azure リージョンの名前 |

1. **「コンピューティング + ストレージ」** ラベルの横にある、**「データベースの構成」** リンクを選択します。

1. **「構成」** ブレードで、**「サーバーレス」** を選択し、対応するハードウェア構成と自動一時停止の遅延設定を確認し、**「自動一時停止を有効にする」** チェック ボックスをオンのままにして、**「適用」** を選択します。

1. **「SQL Database の作成」** ブレードの **「基本」** タブに戻り、**「次へ」** を選択します。**ネットワーク >** 

1. **「SQL データベースの作成」** ブレードの **「ネットワーク」** タブで、次の設定を指定します (他は既定値のままにします)。

    | 設定 | 値 | 
    | --- | --- |
    | 接続方法 | **パブリック エンドポイント** |  
    | Azure サービスとリソースのサーバーへのアクセスを許可する | **なし** |
    | 現在のクライアント IP アドレスを追加する | **はい** |

1. 「**次へ: セキュリティ >**」 を選択します。

1. **「SQL データベースの作成」** ブレードの **「セキュリティ」** タブで、次の設定を指定します (他は既定値のままにします)。

    | 設定 | 値 | 
    | --- | --- |
    | Azure Defender for SQL を有効にする | **後で** |

1. 「**次へ: 追加の設定 >**」 を選択します。 

1. **「SQL Database の作成」** ブレードの **「追加設定」** タブで、次の設定を指定します (既定値を使用し、他はそのままにします)。

    | 設定 | 値 | 
    | --- | --- |
    | 既存のデータを使用する | **サンプル** |

1. **「Review + create」** を選択し、次に **「作成」** を選択します。 

    >**注**: SQL Database が作成されるのを待ちます。プロビジョニングには約 2 分かかります。 


#### タスク 2: Azure SQL データベースへ接続しクエリする

1. Azure portal で、**「SQL データベース」** を検索して選択し、**「SQL Database」** ブレードで、新しく作成された **az30303a-db1** Azure SQL データベースを表すエントリを選択します。

1. 「SQL Database ブレードで、**「クエリ エディター (プレビュー)」** を選択します。

1. **「SQL Server 認証」** セクションの **「パスワード」** ボックスに、**Pa55w.rd1234** と入力し、**「OK」** をクリックします。

1. **「クエリ エディター (プレビュー)」** ペインの **「クエリ 1」** タブで、次のクエリを入力し、**「実行」** を選択します。

    ```SQL
    SELECT TOP 20 pc.Name as CategoryName, p.name as ProductName
    FROM SalesLT.ProductCategory pc
    JOIN SalesLT.Product p
    ON pc.productcategoryid = p.productcategoryid;
    ```

1. **「結果」** タブを確認して、クエリが正常に完了したことを確認します。


### 演習 2: Azure SQL データベースをデータ ストアとして使用する .NET Core コンソール アプリを実装する
  
この演習の主なタスクは次のとおりです。

1. Azure SQL データベースの ADO.NET 接続情報を識別する

1. .NET Core アプリの作成と構成

1. .NET コンソール アプリをテストする

1. Azure SQL データベース ファイアウォールを構成する

1. .NET Core コンソール アプリの機能を確認する

1. ラボでデプロイされている Azure リソースを削除する


#### タスク 1: Azure SQL データベースの ADO.NET 接続情報を識別する

1. Azure portal で、前の演習で展開した Azure SQL データベースのブレードの **「設定」** で **「接続文字列」** を選択します。

1. **「ADO.NET」** タブで、SQL 認証の ADO.NET 接続文字列を確認します。


#### タスク 2: .NET Core アプリの作成と構成

1. Azure portal で、検索テキスト ボックスの右側にあるツール バー アイコンを選択して **「クラウド シェル」 ** ウィンドウを開きます。

1. **「Bash」** または **「PowerShell」** のどちらかを選択するためのプロンプトが表示されたら、**「Bash」** を選択します。 

    >**注**: **Cloud Shell** を起動するのが初めてであり、「**ストレージがマウントされていません**」のメッセージが表示された場合は、このラボで使用しているサブスクリプションを選択し、「**ストレージを作成**」 を選択します。 

1. 「Cloud Shell」 ペインで次の手順を実行して **az30303a1** という名前の新しいフォルダーを作成し、それを現在のディレクトリとして設定します。

   ```sh
   mkdir az30303a1
   cd az30303a1/
   ```

1. 「Cloud Shell」 ウィンドウで、次の手順を実行して、デスクトップ テンプレートに基づく .NET Core ベースのアプリの新しいアプリ プロジェクト ファイルを作成します。

   ```sh
   dotnet new console
   ```

1. 「クラウド シェル」 ウィンドウで、組み込みのエディターを使用して、`<Project>` タグの間に次の XML 要素を追加して、**az30303a1.csproj** ファイルを開いて変更します。 

   ```xml
   <ItemGroup>
       <PackageReference Include="System.Data.SqlClient" Version="4.6.0" />
   </ItemGroup>
   ```

1. **az30303a1.csproj** ファイルを保存して閉じます。

1. 「Cloud Shell」 ウィンドウで、組み込みのエディターを使用して、コンテンツを次のコードに置き換えることで、**Program.cs** ファイルを開いて変更します。 

   ```cs
   using System;
   using System.Data.SqlClient;
   using System.Text;

   namespace sqltest
   {
       class Program
       {
           static void Main(string[] args)
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

1. エディター ウィンドウを開いたままにします。 

1. Azure portal で、**az30303a-db1** データベースの接続文字列を表示するブレードで、ADO.NET 接続文字列をコピーします。 

1. エディター ウィンドウに戻り、プレースホルダー `<your_ado_net_connection_string>` を前の手順でコピーした接続文字列の値に置き換えます。

1. エディター ウィンドウにコピーした接続文字列で、プレース ホルダー `{your_password}` を **Pa55w.rd1234** に置き換えます。

1. **Program.cs** ファイルを保存して閉じます。


#### タスク 3: .NET コンソール アプリをテストする

1. 「Cloud Shell」 ウィンドウで、次の手順を実行して、新しく作成された .NET Core ベースのコンソール アプリをコンパイルします。

   ```sh
   dotnet restore
   ```

1. 「Cloud Shell」 ウィンドウで、次のコマンドを実行して、新しく作成された .NET Core ベースのコンソール アプリを実行します。

   ```sh
   dotnet run
   ```

1. コンソール アプリの実行によってエラーがトリガーされることに注意してください。 

    >**注**: Cloud Shell セッションを実行している仮想マシンに割り当てられた IP アドレスからの接続は明示的に許可される必要があるため、これは予期されます。


#### タスク 4: Azure SQL データベース ファイアウォールを構成する

1. Cloud Shell ペインから、次の操作を実行して、Cloud Shell セッションを実行している仮想マシンのパブリック IP アドレスを識別します。

   ```sh
   curl -s checkip.dyndns.org | sed -e 's/.*Current IP Address: //' -e 's/<.*$//'
   ```

1. Azure Portal で、**az30303a-db1** データベースの接続文字列を表示するブレードで、**「概要」 ** を選択し、ツール バーで **「サーバー ファイアウォールの設定」** を選択します。

1. **ファイアウォールの設定** ブレードで、次のエントリーを設定してから **「保存」** を選びます。

    | 設定 | 値 | 
    | --- | --- |
    | ルール名 | **cloudshell** |
    | 開始 IP | このタスクで以前に識別した IP アドレス |
    | 終了 IP | このタスクで以前に識別した IP アドレス |

    >**注**: 「Cloud Shell」 セッションを再起動すると IP アドレスが変更されるため、ラボの目的でのみこの機能を使用します。


#### タスク 5: .NET Core コンソール アプリの機能を確認する

1. 「Cloud Shell」 ウィンドウで、次のコマンドを実行して、新しく作成された .NET Core ベースのコンソール アプリを実行します。

   ```sh
   dotnet run
   ```

1. コンソール アプリの実行は今回正常に実行され、Azure Portal SQL Database ブレード内のクエリ エディターに表示されるものと同じ結果が返されることに注意してください。 


#### タスク 6: ラボでデプロイされている Azure リソースを削除する

1. 「Cloud Shell」 ウィンドウで、次の手順を実行して、この演習で作成したリソース グループを一覧表示します。

   ```sh
   az group list --query "[?starts_with(name,'az30303')]".name --output tsv
   ```

    > **注**: このラボで作成したリソース グループのみが出力に含まれていることを確認します。このグループは、このタスクで削除されます。

1. 「Cloud Shell」 ウィンドウから次を実行して、このラボで作成したリソース グループを削除します

   ```sh
   az group list --query "[?starts_with(name,'az30303')]".name --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

1. 「Cloud Shell」 ウィンドウから次の手順を実行して、**az30303a1** という名前のフォルダ－を削除します。

   ```sh
   rm -r ~/az30303a1
   ```

1. 「Cloud Shell」 ペインを閉じます。
