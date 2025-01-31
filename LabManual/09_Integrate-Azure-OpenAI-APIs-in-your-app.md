---
lab:
 title: 'Azure OpenAI SDK をアプリで使用する'
---

# Azure OpenAI API をアプリで使用する

Azure OpenAI サービスを使用すると、開発者はチャットボット、言語モデル、および自然な人間の言語を理解するアプリケーションを作成できます。Azure OpenAI は、事前トレーニングされた AI モデルへのアクセス、およびこれらのモデルをカスタマイズおよび微調整するための API とツールのスイートを提供します。この演習では、Azure OpenAI でモデルをデプロイし、それを自分のアプリケーションで使用する方法を学びます。

この演習のシナリオでは、ハイキングのおすすめを提供するために生成 AI を使用できるアプリを実装するように依頼されたソフトウェア開発者の役割を果たします。この演習で使用される技術は、Azure OpenAI API を使用する任意のアプリに適用できます。

この演習には約 **30** 分かかります。

## Azure OpenAI リソースのプロビジョニング

まだ持っていない場合は、Azure サブスクリプションに Azure OpenAI リソースをプロビジョニングします。

1. **Azure ポータル** (`https://portal.azure.com`) にサインインします。
1. 次の設定で **Azure OpenAI** リソースを作成します：

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
1. 確認と作成のページまで**次へ**を選択し(つまり、ネットワークとタグの画面はデフォルトのままです)、最後に **作成** を選択してリソースをプロビジョニングします。
1. デプロイメントが完了するのを待ちます。その後、Azure ポータルでデプロイされた Azure OpenAI リソースに移動します。

## モデルのデプロイ

Azure には、モデルをデプロイ、管理、および探索するための **Azure AI Foundry ポータル** という Web ベースのポータルがあります。Azure AI Foundry ポータルを使用してモデルをデプロイすることで、Azure OpenAI の探索を開始します。

> **注意**: Azure AI Foundry ポータルを使用していると、実行するタスクを提案するメッセージボックスが表示される場合があります。これらを閉じて、この演習の手順に従ってください。

1. Azure ポータルで、Azure OpenAI リソースの **概要** ページで、**開始** セクションの下部までスクロールし、**AI Foundry ポータル**（以前は AI Studio）に移動するボタンを選択します。
1. Azure AI Foundry ポータルで、左側のペインの **デプロイ** ページを選択し、既存のモデルデプロイメントを表示します。まだない場合は、**＋ モデルのデプロイ** から **基本モデルのデプロイ** を選択し、モデル選択画面から **gpt-35-turbo-16k** モデルを選択し、確認を選択して新しいデプロイメントを作成します：
    - **デプロイメント名**: *一意の名前を入力*
    - **デプロイメントタイプ**: Standard

    以下は、モデルの詳細を展開し、設定します。
    - **モデルバージョン**: *デフォルトバージョンを使用*
    - **トークン毎分レート制限**: 5K\*
    - **コンテンツフィルター**: デフォルト
    - **動的クォータの有効化**: 無効

    > \* 毎分 5,000 トークンのレート制限は、この演習を完了するのに十分であり、同じサブスクリプションを使用している他の人のための容量も残ります。

## Visual Studio Code でアプリを開発する準備

Visual Studio Code を使用して Azure OpenAI アプリを開発します。アプリのコードファイルは GitHub リポジトリに提供されています。

> **ヒント**: すでに **mslearn-openai** リポジトリをクローンしている場合は、Visual Studio Code で開きます。そうでない場合は、次の手順に従って開発環境にクローンします。

1. Visual Studio Code を起動します。
2. パレット（SHIFT+CTRL+P）を開き、**Git: Clone** コマンドを実行して `https://github.com/MicrosoftLearning/mslearn-openai` リポジトリをローカルフォルダにクローンします（フォルダはどこでも構いません）。
3. リポジトリがクローンされたら、Visual Studio Code でフォルダを開きます。

    > **注意**: Visual Studio Code がコードの信頼を求めるポップアップメッセージを表示した場合は、**はい、著者を信頼します** オプションをクリックします。

4. リポジトリ内の C# コードプロジェクトをサポートするための追加ファイルがインストールされるのを待ちます。

    > **注意**: ビルドとデバッグに必要なアセットを追加するように求められた場合は、**今は追加しない** を選択します。

## アプリケーションの構成

C# と Python の両方のアプリケーションが提供されており、同じ機能を備えています。まず、Azure OpenAI リソースを使用できるようにアプリケーションのいくつかの重要な部分を完了します。

1. Visual Studio Code の **エクスプローラー** ペインで **Labfiles/02-azure-openai-api** フォルダに移動し、言語の好みに応じて **CSharp** または **Python** フォルダを展開します。各フォルダには、Azure OpenAI 機能を統合するアプリの言語固有のファイルが含まれています。
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
    - モデルデプロイメントのために指定した **デプロイメント名**（Azure AI Foundry ポータルの **デプロイメント** ページで利用可能）

5. 構成ファイルを保存します。

## Azure OpenAI サービスを使用するコードの追加

Azure OpenAI SDK を使用してデプロイされたモデルを利用する準備が整いました。

1. **エクスプローラー** ペインで **CSharp** または **Python** フォルダ内のコードファイルを開き、コメント ***Add Azure OpenAI package*** を Azure OpenAI SDK ライブラリを追加するコードに置き換えます：

    **C#**: Program.cs

    ```csharp
    // Add Azure OpenAI package
    using Azure.AI.OpenAI;
    ```
    
    **Python**: test-openai-model.py
    
    ```python
    # Add Azure OpenAI package
    from openai import AzureOpenAI
    ```

1. 使用する言語のアプリケーションコードで、コメント ***Initialize the Azure OpenAI client...*** を次のコードに置き換えて、クライアントを初期化し、システムメッセージを定義します。

    **C#**: Program.cs

    ```csharp
    // Initialize the Azure OpenAI client
    OpenAIClient client = new OpenAIClient(new Uri(oaiEndpoint), new AzureKeyCredential(oaiKey));
    
    // System message to provide context to the model
    string systemMessage = "I am a hiking enthusiast named Forest who helps people discover hikes in their area. If no area is specified, I will default to near Rainier National Park. I will then provide three suggestions for nearby hikes that vary in length. I will also share an interesting fact about the local nature on the hikes when making a recommendation.";
    ```

    **Python**: test-openai-model.py

    ```python
    # Initialize the Azure OpenAI client
    client = AzureOpenAI(
            azure_endpoint = azure_oai_endpoint, 
            api_key=azure_oai_key,  
            api_version="2024-02-15-preview"
            )
    
    # Create a system message
    system_message = """I am a hiking enthusiast named Forest who helps people discover hikes in their area. 
        If no area is specified, I will default to near Rainier National Park. 
        I will then provide three suggestions for nearby hikes that vary in length. 
        I will also share an interesting fact about the local nature on the hikes when making a recommendation.
        """
    ```

1. コメント ***Add code to send request...*** を次のコードに置き換えて、リクエストを構築し、`messages` や `temperature` などのモデルのさまざまなパラメータを指定します。

    **C#**: Program.cs

    ```csharp
    // Add code to send request...
    // Build completion options object
    ChatCompletionsOptions chatCompletionsOptions = new ChatCompletionsOptions()
    {
        Messages =
        {
            new ChatRequestSystemMessage(systemMessage),
            new ChatRequestUserMessage(inputText),
        },
        MaxTokens = 400,
        Temperature = 0.7f,
        DeploymentName = oaiDeploymentName
    };

    // Send request to Azure OpenAI model
    ChatCompletions response = client.GetChatCompletions(chatCompletionsOptions);

    // Print the response
    string completion = response.Choices[0].Message.Content;
    Console.WriteLine("Response: " + completion + "\n");
    ```

    **Python**: test-openai-model.py

    ```python
    # Add code to send request...
    # Send request to Azure OpenAI model
    response = client.chat.completions.create(
        model=azure_oai_deployment,
        temperature=0.7,
        max_tokens=400,
        messages=[
            {"role": "system", "content": system_message},
            {"role": "user", "content": input_text}
        ]
    )
    generated_text = response.choices[0].message.content

    # Print the response
    print("Response: " + generated_text + "\n")
    ```

1. コードファイルへの変更を保存します。

## アプリケーションのテスト

アプリが構成されたので、リクエストをモデルに送信し、応答を観察するために実行します。

1. インタラクティブターミナルペインで、フォルダコンテキストが使用する言語のフォルダであることを確認します。次に、アプリケーションを実行するために次のコマンドを入力します。

    - **C#**: `dotnet run`
    - **Python**: `python test-openai-model.py`

    > **ヒント**: ターミナルツールバーの **パネルサイズを最大化** (**^**) アイコンを使用して、コンソールテキストをより多く表示できます。

1. プロンプトが表示されたら、`What hike should I do near Rainier?` と入力します。
1. 出力を観察し、応答が *messages* 配列に追加したシステムメッセージのガイドラインに従っていることを確認します。
1. `Where should I hike near Boise? I'm looking for something of easy difficulty, between 2 to 3 miles, with moderate elevation gain.` と入力し、出力を観察します。
1. 使用する言語のコードファイルで、リクエストの *temperature* パラメータ値を **1.0** に変更し、ファイルを保存します。
1. 上記のプロンプトを使用してアプリケーションを再度実行し、出力を観察します。

温度を上げると、ランダム性が増すため、同じテキストを提供しても応答が変わることがよくあります。何度か実行して、出力がどのように変わるかを確認してください。同じ入力で異なる温度値を試してみてください。

## 会話履歴の維持

ほとんどの実際のアプリケーションでは、会話の以前の部分を参照する機能により、AI エージェントとのより現実的な対話が可能になります。Azure OpenAI API は設計上ステートレスですが、プロンプトに会話の履歴を提供することで、AI モデルが過去のメッセージを参照できるようになります。

1. アプリを再度実行し、`Where is a good hike near Boise?` と入力します。
1. 出力を観察し、次に `How difficult is the second hike you suggested?` と入力します。
1. モデルの応答は、おそらく参照しているハイキングを理解できないことを示します。それを修正するために、モデルが過去の会話メッセージを参照できるようにします。
1. アプリケーションで、送信する次のプロンプトに以前のプロンプトと応答を追加する必要があります。**system message** の定義の下に次のコードを追加します。

    **C#**: Program.cs

    ```csharp
    // Initialize messages list
    var messagesList = new List<ChatRequestMessage>()
    {
        new ChatRequestSystemMessage(systemMessage),
    };
    ```

    **Python**: test-openai-model.py

    ```python
    # Initialize messages array
    messages_array = [{"role": "system", "content": system_message}]
    ```

1. コメント ***Add code to send request...*** の下にあるすべてのコードをコメントから **while** ループの終わりまで次のコードに置き換え、ファイルを保存します。コードはほとんど同じですが、会話履歴を保存するためにメッセージ配列を使用するようになりました。

    **C#**: Program.cs

    ```csharp
    // Add code to send request...
    // Build completion options object
    messagesList.Add(new ChatRequestUserMessage(inputText));

    ChatCompletionsOptions chatCompletionsOptions = new ChatCompletionsOptions()
    {
        MaxTokens = 1200,
        Temperature = 0.7f,
        DeploymentName = oaiDeploymentName
    };

    // Add messages to the completion options
    foreach (ChatRequestMessage chatMessage in messagesList)
    {
        chatCompletionsOptions.Messages.Add(chatMessage);
    }

    // Send request to Azure OpenAI model
    ChatCompletions response = client.GetChatCompletions(chatCompletionsOptions);

    // Return the response
    string completion = response.Choices[0].Message.Content;

    // Add generated text to messages list
    messagesList.Add(new ChatRequestAssistantMessage(completion));

    Console.WriteLine("Response: " + completion + "\n");
    ```

    **Python**: test-openai-model.py

    ```python
    # Add code to send request...
    # Send request to Azure OpenAI model
    messages_array.append({"role": "user", "content": input_text})

    response = client.chat.completions.create(
        model=azure_oai_deployment,
        temperature=0.7,
        max_tokens=1200,
        messages=messages_array
    )
    generated_text = response.choices[0].message.content
    # Add generated text to messages array
    messages_array.append({"role": "assistant", "content": generated_text})

    # Print generated text
    print("Summary: " + generated_text + "\n")
    ```

1. ファイルを保存します。追加したコードでは、以前の入力と応答をプロンプト配列に追加することで、モデルが会話の履歴を理解できるようにしています。
1. ターミナルペインで、次のコマンドを入力してアプリケーションを実行します。

    - **C#**: `dotnet run`
    - **Python**: `python test-openai-model.py`

1. アプリを再度実行し、`Where is a good hike near Boise?` と入力します。
1. 出力を観察し、次に `How difficult is the second hike you suggested?` と入力します。
1. モデルが提案した2番目のハイキングについての応答を得る可能性が高く、より現実的な会話が提供されます。以前の回答を参照して追加のフォローアップ質問をすることができ、毎回履歴がモデルの回答にコンテキストを提供します。

    > **ヒント**: トークン数は 1200 に設定されているため、会話が長く続くとアプリケーションが利用可能なトークンを使い果たし、不完全なプロンプトが生成される可能性があります。実際の使用では、履歴の長さを最新の入力と応答に制限することで、必要なトークンの数を制御するのに役立ちます。

## クリーンアップ

Azure OpenAI リソースの使用が終わったら、**Azure ポータル** (`https://portal.azure.com`) でデプロイメントまたはリソース全体を削除することを忘れないでください。
