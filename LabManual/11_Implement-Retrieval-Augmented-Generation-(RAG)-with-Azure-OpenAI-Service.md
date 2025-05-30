---
lab:
 title: 'Azure OpenAI サービスでの検索強化生成 (RAG) の実装'
---

# Azure OpenAI サービスでの検索強化生成 (RAG) の実装

Azure OpenAI サービスを使用すると、基盤となる LLM のインテリジェンスを使用して独自のデータを利用できます。モデルを特定のトピックに対してのみデータを使用するように制限することも、事前トレーニングされたモデルの結果と組み合わせることもできます。

この演習のシナリオでは、Margie's Travel Agency のソフトウェア開発者としての役割を果たします。Azure AI Search を使用して独自のデータをインデックス化し、Azure OpenAI と組み合わせてプロンプトを強化する方法を探ります。

この演習には約 **30** 分かかります。

## Azure リソースのプロビジョニング

この演習を完了するには、次のものが必要です：

- Azure OpenAI リソース
- Azure AI Search リソース
- Azure Storage アカウント リソース

1. **Azure ポータル** (`https://portal.azure.com`) にサインインします。
2. 次の設定で **Azure OpenAI** リソースを作成します：
    - **サブスクリプション**: *Azure OpenAI サービスへのアクセスが承認された Azure サブスクリプションを選択*
    - **リソースグループ**: *リソースグループを選択または作成*
    - **リージョン**: *次のリージョンからランダムに選択*\*
        - オーストラリア東部
        - カナダ東部
        - 米国東部
        - 米国東部 2
        - フランス中部
        - 日本東部
        - 米国中北部
        - スウェーデン中部
        - スイス北部
        - 英国南部
    - **名前**: *一意の名前を入力*
    - **価格レベル**: Standard S0

    > \* Azure OpenAI リソースは地域ごとのクォータによって制約されます。リストされたリージョンには、この演習で使用されるモデルタイプのデフォルトクォータが含まれています。ランダムにリージョンを選択することで、他のユーザーとサブスクリプションを共有しているシナリオで、単一のリージョンがクォータ制限に達するリスクを減らすことができます。演習の後半でクォータ制限に達した場合は、別のリージョンに新しいリソースを作成する必要があるかもしれません。

3. Azure OpenAI リソースがプロビジョニングされている間に、次の設定で **Azure AI Search** リソースを作成します：
    - **サブスクリプション**: *Azure OpenAI リソースをプロビジョニングしたサブスクリプション*
    - **リソースグループ**: *Azure OpenAI リソースをプロビジョニングしたリソースグループ*
    - **サービス名**: *一意の名前を入力*
    - **場所**: *Azure OpenAI リソースをプロビジョニングしたリージョン*
    - **価格レベル**: Basic

4. Azure AI Search リソースがプロビジョニングされている間に、次の設定で **ストレージアカウント** リソースを作成します：
    - **サブスクリプション**: *Azure OpenAI リソースをプロビジョニングしたサブスクリプション*
    - **リソースグループ**: *Azure OpenAI リソースをプロビジョニングしたリソースグループ*
    - **ストレージアカウント名**: *一意の名前を入力*
    - **リージョン**: *Azure OpenAI リソースをプロビジョニングしたリージョン*
    - **パフォーマンス**: Standard
    - **冗長性**: ローカル冗長ストレージ (LRS)

5. 3 つのリソースすべてが Azure サブスクリプションに正常にデプロイされた後、Azure ポータルでそれらを確認し、次の情報を収集します（後の演習で必要になります）：
    - 作成した Azure OpenAI リソースの **エンドポイント** と **キー**（Azure ポータルの Azure OpenAI リソースの **キーとエンドポイント** ページで利用可能）
    - Azure AI Search サービスのエンドポイント（Azure ポータルの Azure AI Search リソースの概要ページの **Url** 値）。
    - Azure AI Search リソースの **プライマリ管理キー**（Azure ポータルの Azure AI Search リソースの **キー** ページで利用可能）。

## データのアップロード

生成 AI モデルで使用するプロンプトを独自のデータで補強します。この演習では、データは架空の *Margies Travel* 会社の旅行パンフレットのコレクションで構成されています。

1. 新しいブラウザタブで、`https://aka.ms/own-data-brochures` からパンフレットデータのアーカイブをダウンロードします。パンフレットを PC のフォルダに抽出します(Extracl All...)。
1. Azure ポータルで、ストレージアカウントに移動し、**ストレージブラウザ** ページを表示します。
1. **Blob コンテナー** を選択し、`margies-travel` という名前の新しいコンテナーを追加します。
1. **margies-travel** コンテナーを選択し、以前に抽出した .pdf パンフレットを Blob コンテナーのルートフォルダにアップロードします。

## AI モデルのデプロイ

この演習では、次の 2 つの AI モデルを使用します：

- テキスト埋め込みモデル：パンフレットのテキストをベクトル化して、プロンプトの補強に効率的に使用できるようにインデックス化します。

- GPT モデル：プロンプトに基づいて応答を生成するためにアプリケーションで使用します。

これらのモデルをデプロイするには、AI Foundry を使用します。

1. Azure ポータルで Azure OpenAI リソースに移動します。次に、リンクを使用して **Azure AI Foundry ポータル** でリソースを開きます。
1. Azure AI Foundry ポータルの **デプロイ** ページで、既存のモデルデプロイメントを表示します。次に、**＋ モデルのデプロイ** から **基本モデルのデプロイ** を選択し、モデル選択画面から **text-embedding-ada-002** モデルの新しいベースモデルデプロイメントを作成します：
    - **デプロイメント名**: *一意の名前を入力*
    - **デプロイメントタイプ**: Standard

    以下は、モデルの詳細を展開し、設定します。
    - **モデルバージョン**: *デフォルトバージョンを使用*
    - **トークン毎分レート制限**: 5K\*
    - **コンテンツフィルター**: デフォルト
    - **動的クォータの有効化**: 無効

1. テキスト埋め込みモデルがデプロイされた後、**デプロイ** ページに戻り、次の設定で **gpt-4o** モデルの
新しいデプロイメントを作成します：
    - **デプロイメント名**: gpt-4o
    - **デプロイメントタイプ**: Standard

    以下は、モデルの詳細を展開し、設定します。
    - **モデルバージョン**: *デフォルトバージョンを使用*
    - **トークン毎分レート制限**: 150K\*
    - **コンテンツフィルター**: デフォルト
    - **動的クォータの有効化**: 有効
    > \* 毎分 5,000 トークンのレート制限は、この演習を完了するのに十分であり、同じサブスクリプションを使用している他の人のための容量も残ります。

## インデックスの作成

独自のデータをプロンプトで簡単に使用できるようにするために、Azure AI Search を使用してインデックス化します。インデックス作成プロセス中に以前デプロイしたテキスト埋め込みモデルを使用してテキストデータをベクトル化します（これにより、インデックス内の各テキストトークンが数値ベクトルで表され、生成 AI モデルがテキストを表現する方法と互換性があります）。

1. Azure ポータルで Azure AI Search リソースに移動します。
1. **概要** ページで **データのインポートとベクトル化** を選択します。
1. **データ接続の設定** ページで **Azure Blob Storage** を選択し、続けて**RAG**を選択します。次の設定でデータソースを構成します：
    - **サブスクリプション**: ストレージアカウントをプロビジョニングした Azure サブスクリプション。
    - **Blob ストレージアカウント**: 以前に作成したストレージアカウント。
    - **Blob コンテナー**: margies-travel
    - **Blob フォルダー**: 空白のままにします。
    - **Parsing mode**: デフォルトのままにします。
    - **ドキュメントレイアウト検出を有効可**: 未選択
    - **削除追跡の有効化**: 未選択
    - **マネージド ID を使用して認証**: 未選択

1. **テキストのベクトル化** ページで次の設定を選択します：
    - **種類**: Azure OpenAI
    - **サブスクリプション**: Azure OpenAI サービスをプロビジョニングした Azure サブスクリプション。
    - **Azure OpenAI サービス**: Azure OpenAI サービスリソース
    - **モデルデプロイメント**: text-embedding-ada-002
    - **認証タイプ**: API キー
    - **Azure OpenAI サービスへの接続がアカウントに追加のコストを発生させることを認識しています**: 選択

1. 次のページで、画像のベクトル化や AI スキルでのデータ抽出のオプションを選択しないでください。
1. 次のページで、セマンティックランキングを有効にし、インデクサーを一度実行するようにスケジュールします。
1. 最終ページで、**オブジェクト名プレフィックス** を `margies-index` に設定し、インデックスを作成します。

## Visual Studio Code でアプリを開発する準備

次に、Azure OpenAI サービス SDK を使用するアプリで独自のデータを使用する方法を探索します。Visual Studio Code を使用してアプリを開発します。アプリのコードファイルは GitHub リポジトリに提供されています。

> **ヒント**: すでに **mslearn-openai** リポジトリをクローンしている場合は、Visual Studio Code で開きます。そうでない場合は、次の手順に従って開発環境にクローンします。

1. Visual Studio Code を起動します。
2. パレット（SHIFT+CTRL+P）を開き、**Git: Clone** コマンドを実行して `https://github.com/MicrosoftLearning/mslearn-openai` リポジトリをローカルフォルダにクローンします（フォルダはどこでも構いません）。
3. リポジトリがクローンされたら、Visual Studio Code でフォルダを開きます。

    > **注意**: Visual Studio Code がコードの信頼を求めるポップアップメッセージを表示した場合は、**はい、著者を信頼します** オプションをクリックします。

4. リポジトリ内の C# コードプロジェクトをサポートするための追加ファイルがインストールされるのを待ちます。
    > **注意**: ビルドとデバッグに必要なアセットを追加するように求められた場合は、**今は追加しない** を選択します。

## アプリケーションの構成

C# と Python の両方のアプリケーションが提供されており、同じ機能を備えています。まず、Azure OpenAI リソースを使用できるようにアプリケーションのいくつかの重要な部分を完了します。

1. Visual Studio Code の **エクスプローラー** ペインで **Labfiles/06-use-own-data** フォルダに移動し、言語の好みに応じて **CSharp** または **Python** フォルダを展開します。各フォルダには、Azure OpenAI 機能を統合するアプリの言語固有のファイルが含まれています。
2. コードファイルを含む **CSharp** または **Python** フォルダを右クリックし、統合ターミナルを開きます。次に、言語の好みに応じて適切なコマンドを実行して Azure OpenAI SDK パッケージをインストールします：

    **C#**:

    ```
    dotnet add package Azure.AI.OpenAI --version 1.0.0-beta.14
    ```

    **Python**:

    ```
    pip install openai==1.55.3
    ```

3. **エクスプローラー** ペインで **CSharp** または **Python** フォルダ内の構成ファイルを開きます：

    - **C#**: appsettings.json
    - **Python**: .env

4. 構成ファイルの値を更新して、次の情報を含めます：
    - 作成した Azure OpenAI リソースの **エンドポイント** と **キー**（Azure ポータルの Azure OpenAI リソースの **キーとエンドポイント** ページで利用可能）
    - gpt-4o モデルデプロイメントのために指定した **デプロイメント名**（Azure AI Foundry ポータルの **デプロイメント** ページで利用可能）
    - 検索サービスのエンドポイント（Azure ポータルの検索リソースの概要ページの **Url** 値）。
    - 検索リソースの **キー**（Azure ポータルの検索リソースの **キー** ページで利用可能 - 管理キーのいずれかを使用できます）
    - 検索インデックスの名前（`margies-index` であるはずです）。

5. 構成ファイルを保存します。

### Azure OpenAI サービスを使用するコードの追加

Azure OpenAI SDK を使用してデプロイされたモデルを利用する準備が整いました。

1. **エクスプローラー** ペインで **CSharp** または **Python** フォルダ内のコードファイルを開き、コメント ***Configure your data source*** を Azure OpenAI SDK ライブラリを追加するコードに置き換えます：

    **C#**: ownData.cs

    ```csharp
    // Configure your data source
    AzureSearchChatExtensionConfiguration ownDataConfig = new()
    {
            SearchEndpoint = new Uri(azureSearchEndpoint),
            Authentication = new OnYourDataApiKeyAuthenticationOptions(azureSearchKey),
            IndexName = azureSearchIndex
    };
    ```

    **Python**: ownData.py

    ```python
    # Configure your data source
    extension_config = dict(dataSources = [  
            { 
                "type": "AzureCognitiveSearch", 
                "parameters": { 
                    "endpoint":azure_search_endpoint, 
                    "key": azure_search_key, 
                    "indexName": azure_search_index,
                }
            }]
        )
    ```

2. 残りのコードを確認し、データソース設定に関する情報を提供するためにリクエストボディで *extensions* を使用していることに注意します。
3. コードファイルへの変更を保存します。

## アプリケーションの実行

アプリが構成されたので、リクエストをモデルに送信し、応答を観察するために実行します。異なるオプションの唯一の違いはプロンプトの内容であり、他のすべてのパラメータ（トークン数や温度など）は各リクエストで同じままです。

1. インタラクティブターミナルペインで、フォルダコンテキストが使用する言語のフォルダであることを確認します。次に、アプリケーションを実行するために次のコマンドを入力します。
    - **C#**: `dotnet run`
    - **Python**: `python ownData.py`

    > **ヒント**: ターミナルツールバーの **パネルサイズを最大化** (**^**) アイコンを使用して、コンソールテキストをより多く表示できます。

2. プロンプト `Tell me about London` に対する応答を確認します。これには、回答とともに、検索サービスから取得したプロンプトを補強するために使用されたデータの詳細が含まれているはずです。

    > **ヒント**: 検索インデックスからの引用を表示したい場合は、コードファイルの上部近くにある変数 ***show citations*** を **true** に設定します。

## クリーンアップ

Azure OpenAI リソースの使用が終わったら、**Azure ポータル** (`https://portal.azure.com`) でリソースを削除することを忘れないでください。ストレージアカウントと検索リソースも含めて削除してください。これらは比較的大きなコストがかかる可能性があります。
