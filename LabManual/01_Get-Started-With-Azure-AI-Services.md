---
lab:
    title: 'Azure AI サービスの使用を開始する'
    module: 'モジュール 2 - Azure AI サービスを使用した AI アプリの開発'
---

# Azure AI サービスの使用を開始する

この演習では、Azure サブスクリプションに **Azure AI サービス** リソースを作成し、クライアント アプリケーションから使用することで、Azure AI サービスの使用を開始します。この演習の目的は、特定のサービスに精通することではなく、開発者として Azure AI サービスをプロビジョニングし、使用する一般的なパターンに慣れることです。

## Visual Studio Code でリポジトリをクローンする

Visual Studio Code を使用してコードを開発します。アプリのコード ファイルは GitHub リポジトリに提供されています。

> **ヒント**: すでに **mslearn-ai-services** リポジトリをクローンしている場合は、Visual Studio Code で開きます。そうでない場合は、次の手順に従って開発環境にクローンします。

1. Visual Studio Code を起動します。
2. パレット (SHIFT+CTRL+P) を開き、**Git: Clone** コマンドを実行して `https://github.com/MicrosoftLearning/mslearn-ai-services` リポジトリをローカル フォルダーにクローンします (フォルダーはどこでも構いません)。
3. リポジトリがクローンされたら、Visual Studio Code でフォルダーを開きます。
4. 必要に応じて、リポジトリ内の C# コード プロジェクトをサポートするための追加ファイルがインストールされるのを待ちます。
 > **注**: ビルドおよびデバッグに必要なアセットを追加するように求められた場合は、**Not Now** を選択します。
5. `Labfiles/01-use-azure-ai-services` フォルダーを展開します。C# と Python の両方のコードが提供されています。希望する言語のフォルダーを展開します。

## Azure AI サービス リソースをプロビジョニングする

Azure AI サービスは、アプリケーションに組み込むことができる人工知能機能をカプセル化したクラウドベースのサービスです。特定の API (たとえば、**Language** や **Vision**) 用に個別の Azure AI サービス リソースをプロビジョニングすることも、複数の Azure AI サービス API へのアクセスを単一のエンドポイントとキーで提供する単一の **Azure AI サービス** リソースをプロビジョニングすることもできます。この場合、単一の **Azure AI サービス** リソースを使用します。

1. `https://portal.azure.com` で Azure ポータルを開き、Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。
2. 上部の検索バーで *Azure AI services* を検索し、**Azure AI Services** を選択して、次の設定で Azure AI サービス マルチサービス アカウント リソースを作成します。
 - **サブスクリプション**: *Azure サブスクリプション*
 - **リソース グループ**: *リソース グループを選択または作成 (制限付きサブスクリプションを使用している場合は、新しいリソース グループを作成する権限がない場合があります - 提供されたものを使用します)*
 - **リージョン**: *利用可能なリージョンを選択*
 - **名前**: *一意の名前を入力*
 - **価格レベル**: Standard S0
3. 必要なチェックボックスを選択してリソースを作成します。
4. デプロイメントが完了するのを待ち、デプロイメントの詳細を表示します。
5. リソースに移動し、その **キーとエンドポイント** ページを表示します。このページには、リソースに接続し、開発するアプリケーションから使用するために必要な情報が含まれています。具体的には:
 - クライアント アプリケーションがリクエストを送信できる HTTP *エンドポイント*。
 - 認証に使用できる 2 つの *キー* (クライアント アプリケーションはどちらのキーでも認証に使用できます)。
 - リソースがホストされている *場所*。これは一部の API へのリクエストに必要です。

## REST インターフェイスを使用する

Azure AI サービス API は REST ベースであるため、HTTP 経由で JSON リクエストを送信することで使用できます。この例では、**Language** REST API を使用して言語検出を行うコンソール アプリケーションを探索しますが、基本的な原則はすべての Azure AI サービス API に共通です。

> **注**: この演習では、**C#** または **Python** のいずれかの REST API を使用することができます。以下の手順では、希望する言語に適した操作を行います。

1. Visual Studio Code で、希望する言語に応じて **C-Sharp** または **Python** フォルダーを展開します。
2. **rest-client** フォルダーの内容を表示し、設定ファイルが含まれていることを確認します:
 - **C#**: appsettings.json
 - **Python**: .env
 設定ファイルを開き、Azure AI サービス リソースの **エンドポイント** と認証 **キー** を反映するように設定値を更新します。変更を保存します。
3. **rest-client** フォルダーにはクライアント アプリケーションのコード ファイルが含まれていることを確認します:
 - **C#**: Program.cs
 - **Python**: rest-client.py
 コード ファイルを開き、含まれているコードを確認し、次の詳細に注意します:
 - HTTP 通信を有効にするためにさまざまな名前空間がインポートされています。
 - **Main** 関数のコードは、Azure AI サービス リソースのエンドポイントとキーを取得します。これらは、Text Analytics サービスに REST リクエストを送信するために使用されます。
 - プログラムはユーザー入力を受け取り、**GetLanguage** 関数を使用して、入力されたテキストの言語を検出するために Text Analytics 言語検出 REST API を呼び出します。
 - API に送信されるリクエストは、入力データを含む JSON オブジェクトで構成されます。この場合、各オブジェクトには **id** と **text** が含まれています。
 - サービスのキーは、クライアント アプリケーションを認証するためにリクエスト ヘッダーに含まれています。
 - サービスからの応答は JSON オブジェクトであり、クライアント アプリケーションが解析できます。
4. **rest-client** フォルダーを右クリックし、*Open in Integrated Terminal* を選択して、次のコマンドを実行します:
 
    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    pip install python-dotenv
    python rest-client.py
    ```
5. プロンプトが表示されたら、いくつかのテキストを入力し、サービスによって検出された言語を確認します。たとえば、「Hello」、「Bonjour」、「Gracias」を入力してみてください。
6. アプリケーションのテストが終了したら、「quit」と入力してプログラムを停止します。

## SDK を使用する

Azure AI サービス REST API を直接使用するコードを書くこともできますが、Microsoft C#、Python、Java、Node.js などの多くの人気プログラミング言語向けのソフトウェア開発キット (SDK) もあります。SDK を使用すると、Azure AI サービスを消費するアプリケーションの開発が大幅に簡素化されます。

1. Visual Studio Code で、希望する言語に応じて **C-Sharp** または **Python** フォルダーの下の **sdk-client** フォルダーを展開し、`cd ../sdk-client` を実行して関連する **sdk-client** フォルダーに移動します。
2. 希望する言語に応じて、次のコマンドを実行して Text Analytics SDK パッケージをインストールします:
    **C#**

    ```
    dotnet add package Azure.AI.TextAnalytics --version 5.3.0
    ```

    **Python**

    ```
    pip install azure-ai-textanalytics==5.3.0
    ```

3. **sdk-client** フォルダーの内容を表示し、設定ファイルが含まれていることを確認します:
 - **C#**: appsettings.json
 - **Python**: .env
 設定ファイルを開き、Azure AI サービス リソースの **エンドポイント** と認証 **キー** を反映するように設定値を更新します。変更を保存します。

4. **sdk-client** フォルダーにはクライアント アプリケーションのコード ファイルが含まれていることを確認します:
 - **C#**: Program.cs
 - **Python**: sdk-client.py
 コード ファイルを開き、含まれているコードを確認し、次の詳細に注意します:
 - インストールした SDK の名前空間がインポートされています。
 - **Main** 関数のコードは、Azure AI サービス リソースのエンドポイントとキーを取得します。これらは、SDK を使用して Text Analytics サービスのクライアントを作成するために使用されます。
 - **GetLanguage** 関数は、SDK を使用してサービスのクライアントを作成し、入力されたテキストの言語を検出するためにクライアントを使用します。

5. ターミナルに戻り、**sdk-client** フォルダーにいることを確認してから、次のコマンドを入力してプログラムを実行します:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python sdk-client.py
    ```
6. プロンプトが表示されたら、いくつかのテキストを入力し、サービスによって検出された言語を確認します。たとえば、「Goodbye」、「Au revoir」、「Hasta la vista」を入力してみてください。

7. アプリケーションのテストが終了したら、「quit」と入力してプログラムを停止します。

> **注**: Unicode 文字セットを必要とする一部の言語は、このシンプルなコンソール アプリケーションでは認識されない場合があります。

## リソースのクリーンアップ

このラボで作成した Azure リソースを他のトレーニング モジュールに使用しない場合は、追加の料金が発生しないように削除できます。

1. `https://portal.azure.com` で Azure ポータルを開き、このラボで作成したリソースを検索バーで検索します。
2. リソース ページで **削除** を選択し、リソースを削除する手順に従います。あるいは、リソース グループ全体を削除して、すべてのリソースを一度にクリーンアップすることもできます。

## 詳細情報

Azure AI サービスの詳細については、[Azure AI サービスのドキュメント](https://learn.microsoft.com/ja-jp/azure/ai-services/what-are-ai-services)を参照してください。
