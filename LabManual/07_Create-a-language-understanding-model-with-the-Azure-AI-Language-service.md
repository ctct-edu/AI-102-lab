---
lab:
 title: 'Azure AI Language サービスで言語理解モデルを作成する'
 module: 'モジュール 5 - 言語理解ソリューションの作成'
---

# Azure AI Language サービスで言語理解モデルを作成する

> **注意**
> Azure AI Language サービスの会話型言語理解機能は現在プレビュー中であり、変更される可能性があります。場合によってはモデルのトレーニングが失敗することがあります。その場合は再試行してください。

Azure AI Language サービスを使用すると、アプリケーションがユーザーからの自然言語入力を解釈し、ユーザーの意図（達成したいこと）を予測し、その意図が適用されるエンティティを識別するための*会話型言語理解*モデルを定義できます。

例えば、時計アプリケーションの会話型言語モデルは、次のような入力を処理することが期待されます：
*ロンドンの時間は何時ですか？*
このような入力は、ユーザーが言ったり入力したりする可能性のある*発話*（utterance）の例であり、特定の場所（この場合はロンドン）の時間を取得するという意図（intent）を持っています。

> **注意**
> 会話型言語モデルのタスクは、ユーザーの意図を予測し、その意図が適用されるエンティティを識別することです。会話型言語モデルが実際に意図を満たすためのアクションを実行することはありません。例えば、時計アプリケーションは、ユーザーがロンドンの時間を知りたいと判断するために会話型言語モデルを使用できますが、クライアントアプリケーション自体が正しい時間を決定し、ユーザーに提示するロジックを実装する必要があります。

## *Azure AI Language* リソースのプロビジョニング

サブスクリプションにまだない場合は、Azure サブスクリプションに **Azure AI Language サービス** リソースをプロビジョニングする必要があります。

1. `https://portal.azure.com` で Azure ポータルを開き、Azure サブスクリプションに関連付けられた Microsoft アカウントを使用してサインインします。
1. 上部の検索フィールドで **Azure AI サービス** を検索します。結果で **Language Service** の下の **作成** を選択します。
1. **リソースの作成を続行** を選択します。
1. 次の設定を使用してリソースをプロビジョニングします：
    - **サブスクリプション**: *Azure サブスクリプション*。
    - **リソースグループ**: *リソースグループを選択または作成*。
    - **リージョン**: *次のリージョンから選択*
        - オーストラリア東部
        - 中央インド
        - 中国東部 2
        - 米国東部
        - 米国東部 2
        - 北ヨーロッパ
        - 米国南中部
        - スイス北部
        - 英国南部
        - 西ヨーロッパ
        - 米国西部 2
        - 米国西部 3
    - **名前**: *一意の名前を入力*。
    - **価格レベル**: **F0**（*無料*）または **S**（*標準*）を選択します（F が利用できない場合）。
    - **責任ある AI 通知**: 同意します。
1. **確認と作成** を選択し、次に **作成** を選択してリソースをプロビジョニングします。
1. デプロイメントが完了するのを待ち、デプロイされたリソースに移動します。
1. **キーとエンドポイント** ページを表示します。このページの情報は後の演習で必要になります。

## 会話型言語理解プロジェクトの作成

作成したオーサリングリソースを使用して、会話型言語理解プロジェクトを作成します。

1. 新しいブラウザタブで `https://language.cognitive.azure.com/` の Azure AI Language Studio ポータルを開き、Azure サブスクリプションに関連付けられた Microsoft アカウントでサインインします。
1. 言語リソースの選択を求められた場合は、次の設定を選択します：
    - **Azure ディレクトリ**: サブスクリプションを含む Azure ディレクトリ。
    - **Azure サブスクリプション**: Azure サブスクリプション。
    - **リソースタイプ**: Language。
    - **言語リソース**: 以前に作成した Azure AI Language リソース。

    プロンプトが表示されない場合は、サブスクリプションに複数の言語リソースがある可能性があります。その場合は：
    1. ページ上部の **設定 (⚙)** ボタンを選択します。
    2. **設定** ページで **リソース** タブを表示します。
    3. 作成した言語リソースを選択し、**リソースの切り替え** をクリックします。
    4. ページ上部の **Language Studio** をクリックして Language Studio ホームページに戻ります。

1. ポータルの上部で **新規作成** メニューから **会話型言語理解** を選択します。
1. **プロジェクトの作成** ダイアログボックスで、**基本情報の入力** ページに次の詳細を入力し、**次へ** を選択します：
    - **名前**: `Clock`
    - **発話の主要言語**: 英語
    - **プロジェクトで複数の言語を有効にしますか？**: *未選択*
    - **説明**: `自然言語時計`

1. **確認と完了** ページで **作成** を選択します。

### 意図の作成

新しいプロジェクトで最初に行うことは、いくつかの意図を定義することです。最終的にモデルは、自然言語発話を送信するユーザーがどの意図を要求しているかを予測します。

> **ヒント**: プロジェクト作業中にヒントが表示された場合は、それを読み、**わかりました** を選択して閉じるか、**すべてスキップ** を選択します。

1. **スキーマ定義** ページの **意図** タブで、**＋ 追加** を選択して `GetTime` という名前の新しい意図を追加します。
1. **GetTime** 意図がリストされていることを確認し（デフォルトの **None** 意図と共に）、次の追加の意図を追加します：
    - `GetDay`
    - `GetDate`

### 各意図にサンプル発話をラベル付けする

モデルがユーザーの要求する意図を予測できるようにするために、各意図にいくつかのサンプル発話をラベル付けする必要があります。

1. 左側のペインで **データラベリング** ページを選択します。

   > **ヒント**: ペインを **>>** アイコンで展開してページ名を表示し、**<<** アイコンで再度非表示にできます。

1. 新しい **GetTime** 意図を選択し、発話 `what is the time?` を入力します。これにより、意図のサンプル入力として発話が追加されます。
1. **GetTime** 意図に次の追加の発話を追加します：
    - `what's the time?`
    - `what time is it?`
    - `tell me the time`

    > **注意**
    > 新しい発話を追加するには、意図の横のテキストボックスに発話を入力し、ENTER キーを押します。

1. **GetDay** 意図を選択し、その意図のサンプル入力として次の発話を追加します：
    - `what day is it?`
    - `what's the day?`
    - `what is the day today?`
    - `what day of the week is it?`
1. **GetDate** 意図を選択し、次の発話を追加します：
    - `what date is it?`
    - `what's the date?`
    - `what is the date today?`
    - `what's today's date?`
1. 各意図に発話を追加したら、**変更を保存** を選択します。

### モデルのトレーニングとテスト

いくつかの意図を追加したので、言語モデルをトレーニングし、ユーザー入力から正しく予測できるか確認しましょう。

1. 左側のペインで **トレーニングジョブ** を選択します。次に **+ トレーニングジョブの開始** を選択します。
1. **トレーニングジョブの開始** ダイアログで、新しいモデルをトレーニングするオプションを選択し、名前を `Clock` とします。**標準トレーニング** モードとデフォルトの **データ分割** オプションを選択します。
1. モデルのトレーニングプロセスを開始するには、**トレーニング** を選択します。
1. トレーニングが完了すると（数分かかる場合があります）、ジョブの **ステータス** が **トレーニング成功** に変わります。
1. **モデルのパフォーマンス** ページを選択し、**Clock** モデルを選択します。トレーニング時に実行された評価によって生成された全体および意図ごとの評価指標（*精度*、*再現率*、および *F1 スコア*）と *混同行列* を確認します（サンプル発話の数が少ないため、すべての意図が結果に含まれていない場合があります）。

    > **注意**
    > 評価指標の詳細については、[ドキュメント](https://learn.microsoft.com/azure/ai-services/language-service/conversational-language-understanding/concepts/evaluation-metrics)を参照してください。

1. **モデルのデプロイ** ページに移動し、**デプロイの追加** を選択します。
1. **デプロイの追加** ダイアログで、**新しいデプロイ名を作成** を選択し、`production` と入力します。
1. **モデル** フィールドで **Clock** モデルを選択し、**デプロイ** を選択します。デプロイには時間がかかる場合があります。
1. モデルがデプロイされたら、**デプロイのテスト** ページを選択し、**デプロイ名** フィールドで **production** デプロイを選択します。
1. 空のテキストボックスに次のテキストを入力し、**テストの実行** を選択します：
    `what's the time now?`
    返された結果を確認し、予測された意図（**GetTime** であるはず）と、モデルが計算した予測意図の確率を示す信頼スコアが含まれていることを確認します。JSON タブには、各潜在的な意図の比較信頼度が表示されます（信頼スコアが最も高いものが予測された意図です）。
1. テキストボックスをクリアし、次のテキストで別のテストを実行します：
    `tell me the time`
    再度、予測された意図と信頼スコアを確認します。
1. 次のテキストを試してください：
    `what's the day today?`
    モデルが **GetDay** 意図を予測することを期待します。

## エンティティの追加

これまで、意図にマッピングされる単純な発話を定義しました。ほとんどの実際のアプリケーションには、意図のコンテキストを取得するために特定のデータエンティティを抽出する必要がある、より複雑な発話が含まれます。

### 学習エンティティの追加

最も一般的なエンティティは *学習* エンティティであり、モデルは例に基づいてエンティティ値を識別することを学習します。

1. Language Studio で **スキーマ定義** ページに戻り、**エンティティ** タブで **＋ 追加** を選択して新しいエンティティを追加します。
1. **エンティティの追加** ダイアログボックスで、エンティティ名を `Location` と入力し、**学習済み** タブが選択されていることを確認します。次に **エンティティの追加** を選択します。
1. **Location** エンティティが作成されたら、**データラベリング** ページに戻ります。
1. **GetTime** 意図を選択し、次の新しい例の発話を入力します：

    `what time is it in London?`

1. 発話が追加されたら、**London** という単語を選択し、表示されるドロップダウンリストで **Location** を選択して、「London」が場所の例であることを示します。
1. **GetTime** 意図に次の例の発話を追加します：

    `Tell me the time in Paris?`

1. 発話が追加されたら、**Paris** という単語を選択し、**Location** エンティティにマッピングします。
1. **GetTime** 意図に次の例の発話を追加します：

    `what's the time in New York?`

1. 発話が追加されたら、**New York** という単語を選択し、**Location** エンティティにマッピングします。
1. 新しい発話を保存するには、**変更を保存** を選択します。

### リストエンティティの追加

場合によっては、エンティティの有効な値を特定の用語と同義語のリストに制限することができ、アプリが発話内のエンティティのインスタンスを識別するのに役立ちます。

1. Language Studio で **スキーマ定義** ページに戻り、**エンティティ** タブで **＋ 追加** を選択して新しいエンティティを追加します。
1. **エンティティの追加** ダイアログボックスで、エンティティ名を `Weekday` と入力し、**リスト** エンティティタブを選択します。次に **エンティティの追加** を選択します。
1. **Weekday** エンティティのページで、**学習済み** セクションで **必須ではない** が選択されていることを確認します。次に、**リスト** セクションで **＋ 新しいリストの追加** を選択します。次に、次の値と同義語を入力し、**保存** を選択します：

    | リストキー | 同義語 |
    | ------------------- | --------- |
    | `Sunday` | `Sun` |

    > **注意**
    > 新しいリストのフィールドに値 `Sunday` を入力し、「値を入力して Enter キーを押します...」と表示されているフィールドをクリックして同義語を入力し、ENTER キーを押します。

1. 前の手順を繰り返して、次のリストコンポーネントを追加します：

   | 値 | 同義語 |
   | ------------------- | --------- |
   | `Monday` | `Mon` |
   | `Tuesday` | `Tue, Tues` |
   | `Wednesday` | `Wed, Weds` |
   | `Thursday` | `Thur, Thurs` |
   | `Friday` | `Fri` |
   | `Saturday` | `Sat` |

1. リスト値を追加して保存したら、**データラベリング** ページに戻ります。
1. **GetDate** 意図を選択し、次の新しい例の発話を入力します：

    `what date was it on Saturday?`

1. 発話が追加されたら、**Saturday** という単語を選択し、表示されるドロップダウンリストで **Weekday** を選択します。
1. **GetDate** 意図に次の例の発話を追加します：

    `what date will it be on Friday?`

1. 発話が追加されたら、**Friday** を **Weekday** エンティティにマッピングします。
1. **GetDate** 意図に次の例の発話を追加します：

    `what will the date be on Thurs?`

1. 発話が追加されたら、**Thurs** を **Weekday** エンティティにマッピングします。
1. 新しい発話を保存するには、**変更を保存** を選択します。

### 組み込みエンティティの追加

Azure AI Language サービスは、会話型アプリケーションで一般的に使用される一連の *組み込み* エンティティを提供します。

1. Language Studio で **スキーマ定義** ページに戻り、**エンティティ** タブで **＋ 追加** を選択して新しいエンティティを追加します。

1. **エンティティの追加** ダイアログボックスで、エンティティ名を `Date` と入力し、**組み込み** エンティティタブを選択します。次に **エンティティの追加** を選択します。

1. **Date** エンティティのページで、**学習済み** セクションで **必須ではない** が選択されていることを確認します。次に、**組み込み** セクションで **＋ 新しい組み込みの追加** を選択します。

1. **組み込みの選択** リストで **DateTime** を選択し、**保存** を選択します。

1. 組み込みエンティティを追加した後、**データラベリング** ページに戻ります。

1. **GetDay** 意図を選択し、次の新しい例の発話を入力します：

    `what day was 01/01/1901?`

1. 発話が追加されたら、**01/01/1901** という単語を選択し、表示されるドロップダウンリストで **Date** を選択します。

1. **GetDay** 意図に次の例の発話を追加します：
    `what day will it be on Dec 31st 2099?`

1. 発話が追加されたら、**Dec 31st 2099** を **Date** エンティティにマッピングします。

1. 新しい発話を保存するには、**変更を保存** を選択します。

### モデルの再トレーニング

スキーマを変更したので、モデルを再トレーニングして再テストする必要があります。

1. **トレーニングジョブ** ページで **トレーニングジョブの開始** を選択します。

1. **トレーニングジョブの開始** ダイアログで、**既存のモデルを上書き** を選択し、**Clock** モデルを指定します。**トレーニング** を選択してモデルをトレーニングします。プロンプトが表示されたら、既存のモデルを上書きすることを確認します。

1. トレーニングが完了すると、ジョブの **ステータス** が **トレーニング成功** に更新されます。

1. **モデルのパフォーマンス** ページを選択し、**Clock** モデルを選択します。トレーニング時に実行された評価によって生成された評価指標（*精度*、*再現率*、および *F1 スコア*）と *混同行列* を確認します（サンプル発話の数が少ないため、すべての意図が結果に含まれていない場合があります）。

1. **モデルのデプロイ** ページで **デプロイの追加** を選択します。

1. **デプロイの追加** ダイアログで、**既存のデプロイ名を上書き** を選択し、**production** を選択します。

1. **モデル** フィールドで **Clock** モデルを選択し、**デプロイ** を選択してデプロイします。これには時間がかかる場合があります。
1. モデルがデプロイされたら、**デプロイのテスト** ページで **デプロイ名** フィールドの **production** デプロイを選択し、次のテキストでテストします：

    `what's the time in Edinburgh?`

1. 返された結果を確認し、**GetTime** 意図と「Edinburgh」というテキスト値を持つ **Location** エンティティが予測されることを期待します。

1. 次の発話をテストしてみてください：

    `what time is it in Tokyo?`

    `what date is it on Friday?`

    `what's the date on Weds?`

    `what day was 01/01/2020?`

    `what day will Mar 7th 2030 be?`

## クライアントアプリからモデルを使用する

実際のプロジェクトでは、意図とエンティティを反復的に洗練し、再トレーニングし、再テストして、予測性能に満足するまで行います。テストして予測性能に満足したら、REST インターフェイスまたはランタイム固有の SDK を呼び出してクライアントアプリで使用できます。

### Visual Studio Code でアプリを開発する準備

Visual Studio Code を使用して言語理解アプリを開発します。アプリのコードファイルは GitHub リポジトリに提供されています。

> **ヒント**: すでに **mslearn-ai-language** リポジトリをクローンしている場合は、Visual Studio Code で開きます。そうでない場合は、次の手順に従って開発環境にクローンします。

1. Visual Studio Code を起動します。
2. パレット（SHIFT+CTRL+P）を開き、**Git: Clone** コマンドを実行して `https://github.com/MicrosoftLearning/mslearn-ai-language` リポジトリをローカルフォルダにクローンします（フォルダはどこでも構いません）。
3. リポジトリがクローンされたら、Visual Studio Code でフォルダを開きます。
    > **注意**: Visual Studio Code がコードの信頼を求めるポップアップメッセージを表示した場合は、**はい、著者を信頼します** オプションをクリックします。
4. リポジトリ内の C# コードプロジェクトをサポートするための追加ファイルがインストールされるのを待ちます。
    > **注意**: ビルドとデバッグに必要なアセットを追加するように求められた場合は、**今は追加しない** を選択します。

### アプリケーションの構成

C# と Python の両方のアプリケーションが提供されており、要約をテストするためのサンプルテキストファイルも提供されています。両方のアプリは同じ機能を備えています。まず、Azure AI Language リソースを使用できるようにアプリケーションのいくつかの重要な部分を完了します。

1. Visual Studio Code の **エクスプローラー** ペインで **Labfiles/03-language** フォルダに移動し、言語の好みに応じて **CSharp** または **Python** フォルダを展開し、その中にある **clock-client** フォルダを展開します。各フォルダには、Azure AI Language 質問応答機能を統合するアプリの言語固有のファイルが含まれています。
2. コードファイルを含む **clock-client** フォルダを右クリックし、統合ターミナルを開きます。次に、言語の好みに応じて適切なコマンドを実行して Azure AI Language 会話型言語理解 SDK パッケージをインストールします：

    **C#**:

    ```
    dotnet add package Azure.AI.Language.Conversations --version 1.1.0
    ```

    **Python**:

    ```
    pip install azure-ai-language-conversations
    ```

3. **エクスプローラー** ペインで **clock-client** フォルダ内の構成ファイルを開きます。
    - **C#**: appsettings.json
    - **Python**: .env

4. 構成ファイルの値を更新して、作成した Azure Language リソースの **エンドポイント** と **キー** を含めます（Azure ポータルの Azure AI Language リソースの **キーとエンドポイント** ページで利用可能）。
5. 構成ファイルを保存します。

### アプリケーションにコードを追加する

必要な SDK ライブラリをインポートし、デプロイされたプロジェクトに認証された接続を確立し、質問を送信するために必要なコードを追加します。

1. **clock-client** フォルダにはクライアントアプリケーションのコードファイルが含まれています：
    - **C#**: Program.cs
    - **Python**: clock-client.py
    コードファイルを開き、既存の名前空間参照の下にある **Import namespaces** コメントを見つけます。次に、このコメントの下に、Text Analytics SDK を使用するために必要な名前空間をインポートするための言語固有のコードを追加します。

    **C#**: Programs.cs

    ```c#
    // import namespaces
    using Azure;
    using Azure.AI.Language.Conversations;
    ```

    **Python**: clock-client.py

    ```python
    # Import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.language.conversations import ConversationAnalysisClient
    ```

1. **Main** 関数で、予測エンドポイントとキーを構成ファイルから読み込むコードがすでに提供されていることを確認します。次に、**Create a client for the Language service model** コメントを見つけ、Language Service アプリの予測クライアントを作成するための次のコードを追加します。

    **C#**: Programs.cs

    ```c#
    // Create a client for the Language service model
    Uri endpoint = new Uri(predictionEndpoint);
    AzureKeyCredential credential = new AzureKeyCredential(predictionKey);

    ConversationAnalysisClient client = new ConversationAnalysisClient(endpoint, credential);
    ```

    **Python**: clock-client.py

    ```python
    # Create a client for the Language service model
    client = ConversationAnalysisClient(
        ls_prediction_endpoint, AzureKeyCredential(ls_prediction_key))
    ```

1. **Main** 関数のコードがユーザー入力を求めるまでのプロンプトを表示することを確認します。このループ内で、**Call the Language service model to get intent and entities** コメントを見つけ、次のコードを追加します。

    **C#**: Programs.cs

    ```c#
    // Call the Language service model to get intent and entities
    var projectName = "Clock";
    var deploymentName = "production";
    var data = new
    {
        analysisInput = new
        {
            conversationItem = new
            {
                text = userText,
                id = "1",
                participantId = "1",
            }
        },
        parameters = new
        {
            projectName,
            deploymentName,
            // Use Utf16CodeUnit for strings in .NET.
            stringIndexType = "Utf16CodeUnit",
        },
        kind = "Conversation",
    };
    // Send request
    Response response = await client.AnalyzeConversationAsync(RequestContent.Create(data));
    dynamic conversationalTaskResult = response.Content.ToDynamicFromJson(JsonPropertyNames.CamelCase);
    dynamic conversationPrediction = conversationalTaskResult.Result.Prediction;   
    var options = new JsonSerializerOptions { WriteIndented = true };
    Console.WriteLine(JsonSerializer.Serialize(conversationalTaskResult, options));
    Console.WriteLine("--------------------\n");
    Console.WriteLine(userText);
    var topIntent = "";
    if (conversationPrediction.Intents[0].ConfidenceScore > 0.5)
    {
        topIntent = conversationPrediction.TopIntent;
    }
    ```

    **Python**: clock-client.py

    ```python
    # Call the Language service model to get intent and entities
    cls_project = 'Clock'
    deployment_slot = 'production'

    with client:
        query = userText
        result = client.analyze_conversation(
            task={
                "kind": "Conversation",
                "analysisInput": {
                    "conversationItem": {
                        "participantId": "1",
                        "id": "1",
                        "modality": "text",
                        "language": "en",
                        "text": query
                    },
                    "isLoggingEnabled": False
                },
                "parameters": {
                    "projectName": cls_project,
                    "deploymentName": deployment_slot,
                    "verbose": True
                }
            }
        )

    top_intent = result["result"]["prediction"]["topIntent"]
    entities = result["result"]["prediction"]["entities"]

    print("view top intent:")
    print("\ttop intent: {}".format(result["result"]["prediction"]["topIntent"]))
    print("\tcategory: {}".format(result["result"]["prediction"]["intents"][0]["category"]))
    print("\tconfidence score: {}\n".format(result["result"]["prediction"]["intents"][0]["confidenceScore"]))

    print("view entities:")
    for entity in entities:
        print("\tcategory: {}".format(entity["category"]))
        print("\ttext: {}".format(entity["text"]))
        print("\tconfidence score: {}".format(entity["confidenceScore"]))

    print("query: {}".format(result["result"]["query"]))
    ```

    Language サービスモデルへの呼び出しは、最も可能性の高い意図と入力発話で検出されたエンティティを含む予測/結果を返します。クライアントアプリケーションは、今度はその予測を使用して適切なアクションを決定し、実行する必要があります。

1. **Apply the appropriate action** コメントを見つけ、アプリケーションがサポートする意図（**GetTime**、**GetDate**、**GetDay**）をチェックし、関連するエンティティが検出されたかどうかを確認してから、適切な応答を生成する既存の関数を呼び出す次のコードを追加します。

    **C#**: Programs.cs

    ```c#
    // Apply the appropriate action
    switch (topIntent)
    {
        case "GetTime":
            var location = "local";           
            // Check for a location entity
            foreach (dynamic entity in conversationPrediction.Entities)
            {
                if (entity.Category == "Location")
                {
                    //Console.WriteLine($"Location Confidence: {entity.ConfidenceScore}");
                    location = entity.Text;
                }
            }
            // Get the time for the specified location
            string timeResponse = GetTime(location);
            Console.WriteLine(timeResponse);
            break;
        case "GetDay":
            var date = DateTime.Today.ToShortDateString();            
            // Check for a Date entity
            foreach (dynamic entity in conversationPrediction.Entities)
            {
                if (entity.Category == "Date")
                {
                    //Console.WriteLine($"Location Confidence: {entity.ConfidenceScore}");
                    date = entity.Text;
                }
            }            
            // Get the day for the specified date
            string dayResponse = GetDay(date);
            Console.WriteLine(dayResponse);
            break;
        case "GetDate":
            var day = DateTime.Today.DayOfWeek.ToString();
            // Check for entities            
            // Check for a Weekday entity
            foreach (dynamic entity in conversationPrediction.Entities)
            {
                if (entity.Category == "Weekday")
                {
                    //Console.WriteLine($"Location Confidence: {entity.ConfidenceScore}");
                    day = entity.Text;
                }
            }          
            // Get the date for the specified day
            string dateResponse = GetDate(day);
            Console.WriteLine(dateResponse);
            break;
        default:
            // Some other intent (for example, "None") was predicted
            Console.WriteLine("Try asking me for the time, the day, or the date.");
            break;
    }
    ```

    **Python**: clock-client.py

    ```python
    # Apply the appropriate action
    if top_intent == 'GetTime':
        location = 'local'
        # Check for entities
        if len(entities) > 0:
            # Check for a location entity
            for entity in entities:
                if 'Location' == entity["category"]:
                    # ML entities are strings, get the first one
                    location = entity["text"]
        # Get the time for the specified location
        print(GetTime(location))

    elif top_intent == 'GetDay':
        date_string = date.today().strftime("%m/%d/%Y")
        # Check for entities
        if len(entities) > 0:
            # Check for a Date entity
            for entity in entities:
                if 'Date' == entity["category"]:
                    # Regex entities are strings, get the first one
                    date_string = entity["text"]
        # Get the day for the specified date
        print(GetDay(date_string))

    elif top_intent == 'GetDate':
        day = 'today'
        # Check for entities
        if len(entities) > 0:
            # Check for a Weekday entity
            for entity in entities:
                if 'Weekday' == entity["category"]:
                # List entities are lists
                    day = entity["text"]
        # Get the date for the specified day
        print(GetDate(day))

    else:
        # Some other intent (for example, "None") was predicted
        print('Try asking me for the time, the day, or the date.')
    ```

1. 変更を保存し、**clock-client** フォルダの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します：

    - **C#**: `dotnet run`
    - **Python**: `python clock-client.py`

    > **ヒント**: ターミナルツールバーの **パネルサイズを最大化** (**^**) アイコンを使用して、コンソールテキストをより多く表示できます。

1. プロンプトが表示されたら、アプリケーションをテストするための発話を入力します。例えば、次のように試してください：

    *Hello*

    *What time is it?*

    *What's the time in London?*

    *What's the date?*

    *What date is Sunday?*

    *What day is it?*

    *What day is 01/01/2025?*

    > **注意**: アプリケーションのロジックは意図的にシンプルであり、いくつかの制限があります。例えば、時間を取得する場合、サポートされる都市の範囲が制限されており、夏時間は無視されます。この目的は、Language Service を使用する典型的なパターンの例を示すことです。アプリケーションは次の手順を実行する必要があります：
    >    1. 予測エンドポイントに接続する。
    >    2. 発話を送信して予測を取得する。
    >    3. 予測された意図とエンティティに適切に応答するロジックを実装する。

1. テストが終了したら、*quit* と入力します。

## リソースのクリーンアップ

Azure AI Language サービスの探索が終了したら、この演習で作成したリソースを削除できます。手順は次のとおりです：

1. `https://portal.azure.com` で Azure ポータルを開き、Azure サブスクリプションに関連付けられた Microsoft アカウントでサインインします。
2. このラボで作成した Azure AI Language リソースに移動します。
3. リソースページで **削除** を選択し、リソースを削除する手順に従います。

## さらなる情報

Azure AI Language の会話型言語理解について詳しく知るには、[Azure AI Language ドキュメント](https://learn.microsoft.com/azure/ai-services/language-service/conversational-language-understanding/overview)を参照してください。
