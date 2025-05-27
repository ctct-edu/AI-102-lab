---
lab:
 title: 'Azure OpenAI サービスによるアプリケーション開発'
---

# Azure OpenAI サービスによるアプリケーション開発

Azure OpenAI サービスを使用する際、開発者がプロンプトをどのように形作るかが、生成 AI モデルの応答に大きな影響を与えます。Azure OpenAI モデルは、明確かつ簡潔に要求された場合、コンテンツを調整およびフォーマットすることができます。この演習では、類似のコンテンツに対する異なるプロンプトが、AI モデルの応答をどのように形作り、要件をより満たすようにするかを学びます。

この演習のシナリオでは、野生動物のマーケティングキャンペーンに取り組んでいるソフトウェア開発者の役割を果たします。生成 AI を使用して広告メールを改善し、チームに適用できる記事を分類する方法を探ります。この演習で使用されるプロンプトエンジニアリング技術は、さまざまなユースケースに同様に適用できます。

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
1. Azure AI Foundry ポータルで、左側のペインの **デプロイ** ページを選択し、既存のモデルデプロイメントを表示します。まだない場合は、**＋ モデルのデプロイ** から **基本モデルのデプロイ** を選択し、モデル選択画面から **gpt-4o** モデルを選択し、確認を選択して新しいデプロイメントを作成します：
    - **デプロイメント名**: *一意の名前を入力*
    - **デプロイメントタイプ**: Standard

    以下は、モデルの詳細を展開し、設定します。
    - **モデルバージョン**: *デフォルトバージョンを使用*
    - **トークン毎分レート制限**: 5K\*
    - **コンテンツフィルター**: デフォルト
    - **動的クォータの有効化**: 無効

    > \* 毎分 5,000 トークンのレート制限は、この演習を完了するのに十分であり、同じサブスクリプションを使用している他の人のための容量も残ります。

## プロンプトエンジニアリング技術の探索

まず、Chat プレイグラウンドでいくつかのプロンプトエンジニアリング技術を探索してみましょう。

1. **プレイグラウンド** セクションで **Chat** ページを選択します。**Chat** プレイグラウンドページは、ボタンの行と 2 つのメインパネルで構成されています（画面解像度に応じて、水平に右から左、または垂直に上から下に配置される場合があります）：
    - **構成** - デプロイメントを選択し、システムメッセージを定義し、デプロイメントとの対話のためのパラメータを設定します。
    - **チャットセッション** - チャットメッセージを送信し、応答を表示します。

1. **デプロイメント** の下で、gpt-4o モデルデプロイメントが選択されていることを確認します。
1. デフォルトの **システムメッセージ** を確認します。これは *You are an AI assistant that helps people find information.* であるはずです。
1. **チャットセッション** で、次のクエリを送信します：

    ```prompt
    どんな種類の記事ですか？
    ---
    カリフォルニア州では深刻な干ばつが予想されています。
    この干ばつにより、カリフォルニアの住民は水の不足と乾燥した芝生に直面しています。
    南カリフォルニアの当局は、約800万人の住民に対して屋外での水使用を週に1日だけに制限するという初めての措置を取ることを発表しました。
    この状況により、日常生活がどのように変わるかはまだ多くの点で不明ですが、当局は事態が深刻であり、年内にさらに厳しい制限が導入される可能性があると警告しています。
    ```
    応答は記事の説明を提供します。しかし、記事の分類のためにより具体的な形式が必要な場合を考えてみましょう。
1. **構成** セクションでシステムメッセージを `あなたはニュース記事を分類するニュースアグリゲーターです。` に変更します。
1. 新しいシステムメッセージの下で、**セクションの追加** ボタンを選択し、**例** を選択します。次に、次の例を追加します。

    **ユーザー:**

    ```prompt
    どんな種類の記事ですか？
    ---
    ニューヨーク・ベースボーラーズがシカゴに大勝

    ニューヨーク・ベースボーラーズは昨夜、シカゴ・サイクロンズに対して5-0の完封勝利を収めました。7回裏の終盤での3ランホームランにより、勝利が確定しました。

    ピッチャーのマリオ・ロジャースは96球を投げ、ニューヨークのためにヒットを2本しか許さないという今年最高のパフォーマンスを見せました。

    シカゴ・サイクロンズの2本のヒットは2回と5回に出ましたが、ランナーを本塁に返すことはできませんでした。
    ---
    ```
    
   **アシスタント:**

    ```prompt
    スポーツ
    ```
1. 次のテキストで別の例を追加します。

    **ユーザー:**
    
    ```prompt
    この文章をカテゴリ分けしてください： 
    ---
    オスカーの喜びの瞬間

    先週のオスカーはかなりのものでした！

    あるスキャンダルがショーを盗んだかもしれませんが、今年のアカデミー賞には私たちを喜びで満たし、涙さえも誘った瞬間が満載でした。 これらの俳優や女優は、本当に感動的な演技を見せてくれ、笑いも提供し、冬を乗り切る手助けをしてくれました。

    ロビン・クラインの歴史的な勝利から、あのケイシー・ジェンセン自身によるフルパフォーマンスまで、明日の再放送をお見逃しなく。
    ---
    ```
     
    **アシスタント:**

    ```prompt
    エンターテインメント
    ```

1. **構成** セクションの上部にある **変更を適用** ボタンを使用して、変更を保存します。
1. **チャットセッション** セクションで、次のプロンプトを再送信します：

    ```prompt
    どんな種類の記事ですか？
    ---
    カリフォルニア州では深刻な干ばつが予想されています。
    この干ばつにより、カリフォルニアの住民は水の不足と乾燥した芝生に直面しています。
    南カリフォルニアの当局は、約800万人の住民に対して屋外での水使用を週に1日だけに制限するという初めての措置を取ることを発表しました。
    この状況により、日常生活がどのように変わるかはまだ多くの点で不明ですが、当局は事態が深刻であり、年内にさらに厳しい制限が導入される可能性があると警告しています。
    ```

    より具体的なシステムメッセージと、期待されるクエリと応答の例の組み合わせにより、結果の形式が一貫します。

1. システムメッセージをデフォルトのテンプレートに戻し、`You are an AI assistant that helps people find information.` に変更し、例を含めずに変更を適用します。

1. **チャットセッション** セクションで、次のプロンプトを送信します：

    ```prompt
    # 1. Create a list of animals
    # 2. Create a list of whimsical names for those animals
    # 3. Combine them randomly into a list of 25 animal and name pairs
    ```

    モデルはおそらく、番号付きリストに分割されたプロンプトを満たす回答を返すでしょう。これは適切な回答ですが、実際に欲しかったのは、あなたが説明したタスクを実行するPythonプログラムを書くことだとしましょう。

12. システムメッセージを `You are a coding assistant helping write python code.` に変更し、変更を適用します。
13. 次のプロンプトをモデルに再送信します：

    ```
    # 1. Create a list of animals
    # 2. Create a list of whimsical names for those animals
    # 3. Combine them randomly into a list of 25 animal and name pairs
    ```

    モデルはコメントで要求されたことを実行する Python コードで正しく応答するはずです。

## Visual Studio Code でアプリを開発する準備

次に、Azure OpenAI サービス SDK を使用するアプリでプロンプトエンジニアリングを使用する方法を探索します。Visual Studio Code を使用してアプリを開発します。アプリのコードファイルは GitHub リポジトリに提供されています。

> **ヒント**: すでに **mslearn-openai** リポジトリをクローンしている場合は、Visual Studio Code で開きます。そうでない場合は、次の手順に従って開発環境にクローンします。

1. Visual Studio Code を起動します。
2. パレット（SHIFT+CTRL+P）を開き、**Git: Clone** コマンドを実行して `https://github.com/ctct-edu/AI-102-lab-code.git` リポジトリをローカルフォルダにクローンします（フォルダはどこでも構いません）。
3. リポジトリがクローンされたら、Visual Studio Code でフォルダを開きます。
    > **注意**: Visual Studio Code がコードの信頼を求めるポップアップメッセージを表示した場合は、**はい、著者を信頼します** オプションをクリックします。
4. リポジトリ内の C# コードプロジェクトをサポートするための追加ファイルがインストールされるのを待ちます。
    > **注意**: ビルドとデバッグに必要なアセットを追加するように求められた場合は、**今は追加しない** を選択します。

## アプリケーションの構成

C# と Python の両方のアプリケーションが提供されており、同じ機能を備えています。まず、非同期 API 呼び出しを使用して Azure OpenAI リソースを使用できるようにアプリケーションのいくつかの重要な部分を完了します。

1. Visual Studio Code の **エクスプローラー** ペインで **Labfiles/10-prompt-engineering** フォルダに移動し、言語の好みに応じて **CSharp** または **Python** フォルダを展開します。各フォルダには、Azure OpenAI 機能を統合するアプリの言語固有のファイルが含まれています。
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

1. **エクスプローラー** ペインで **CSharp** または **Python** フォルダ内のコードファイルを開きます。
    
    **C#のみ**、doブロック内のコメント ***Pause for system message update*** の直後に次のコードを追加します。
  
    ```csharp
    // Pause for system message update
    Console.OutputEncoding = Encoding.UTF8;
    ```

    既存のコードを消さないように注意してください。


1. コメント ***Add Azure OpenAI package*** を Azure OpenAI SDK ライブラリを追加するコードに置き換えます：

    **C#**: Program.cs

    ```csharp
    // Add Azure OpenAI package
    using Azure.AI.OpenAI;
    ```

    **Python**: prompt-engineering.py

    ```python
    # Add Azure OpenAI package
    from openai import AsyncAzureOpenAI
    ```

2. コードファイルで、コメント ***Configure the Azure OpenAI client*** を見つけ、Azure OpenAI クライアントを構成するコードを追加します：

    **C#**: Program.cs

    ```csharp
    // Configure the Azure OpenAI client
    OpenAIClient client = new OpenAIClient(new Uri(oaiEndpoint), new AzureKeyCredential(oaiKey));
    ```

    **Python**: prompt-engineering.py

    ```python
    # Configure the Azure OpenAI client
    client = AsyncAzureOpenAI(
        azure_endpoint = azure_oai_endpoint, 
        api_key=azure_oai_key,  
        api_version="2024-02-15-preview"
        )
    ```

3. Azure OpenAI モデルを呼び出す関数で、コメント ***Format and send the request to the model*** の下に、リクエストをフォーマットしてモデルに送信するコードを追加します。

    **C#**: Program.cs

    ```csharp
    // Format and send the request to the model
    var chatCompletionsOptions = new ChatCompletionsOptions()
    {
        Messages =
        {
            new ChatRequestSystemMessage(systemMessage),
            new ChatRequestUserMessage(userMessage)
        },
        Temperature = 0.7f,
        MaxTokens = 800,
        DeploymentName = oaiDeploymentName
    };
    
    // Get response from Azure OpenAI
    Response<ChatCompletions> response = await client.GetChatCompletionsAsync(chatCompletionsOptions);
    ```

    **Python**: prompt-engineering.py

    ```python
    # Format and send the request to the model
    messages =[
        {"role": "system", "content": system_message},
        {"role": "user", "content": user_message},
    ]
    
    print("\nSending request to Azure OpenAI model...\n")

    # Call the Azure OpenAI model
    response = await client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.7,
        max_tokens=800
    )
    ```

4. コードファイルへの変更を保存します。

## アプリケーションの実行

アプリが構成されたので、リクエストをモデルに送信し、応答を観察するために実行します。異なるオプションの唯一の違いはプロンプトの内容であり、他のすべてのパラメータ（トークン数や温度など）は各リクエストで同じままです。

1. 使用する言語のフォルダで `system.txt` を Visual Studio Code で開きます。各インタラクションごとに、このファイルに **システムメッセージ** を入力して保存します。各反復は、最初にシステムメッセージを変更するために一時停止します。
1. インタラクティブターミナルペインで、フォルダコンテキストが使用する言語のフォルダであることを確認します。次に、アプリケーションを実行するために次のコマンドを入力します。

    - **C#**: `dotnet run`
    - **Python**: `python prompt-engineering.py`

    > **ヒント**: ターミナルツールバーの **パネルサイズを最大化** (**^**) アイコンを使用して、コンソールテキストをより多く表示できます。

1. 最初の反復では、次のプロンプトを入力します：

    **システムメッセージ**

    ```prompt
    あなたはAIアシスタントです。
    ```
    \* C#の場合、本環境では日本語入力がうまく読み取られない場合があります。その場合は英語で入力してください。
    ```prompt
    You are an AI assistant
    ```

    **ユーザーメッセージ:**

    ```prompt
    新しい野生動物保護施設の紹介文を書いてください
    ```
    ```prompt
    Write an intro for a new wildlife Rescue
    ```

1. 出力を観察します。AI モデルはおそらく、野生動物救助のための良い一般的な紹介を生成します。
1. 次に、応答の形式を指定する次のプロンプトを入力します：

    **システムメッセージ**

    ```prompt
    あなたはメールを書くのを手助けするAIアシスタントです。
    ```
    ```prompt
    You are an AI assistant helping to write emails
    ```

    **ユーザーメッセージ:**

    ```prompt
    新しい野生動物保護施設のための宣伝メールを書いてください。次の情報を含めます：
     - 保護施設の名前はContoso
     - 象を専門に扱う
     - 当社のウェブサイトで寄付の呼びかけ
    ```
    ```prompt
    Write a promotional email for a new wildlife rescue, including the following: 
    - Rescue name is Contoso 
    - It specializes in elephants 
    - Call for donations to be given at our website
    ```

    > **ヒント**: VM での自動入力が複数行のプロンプトでうまく機能しない場合があります。その場合は、次の文をコピーして貼り付けてください。

    ```prompt
    新しい野生動物保護施設のための宣伝メールを書いてください。次の情報を含めます：・保護施設の名前はContoso・象を専門に扱う・当社のウェブサイトで寄付の呼びかけ
    ```


1. 出力を観察します。今回は、特定の動物が含まれ、寄付の呼びかけが含まれたメールの形式が表示されるでしょう。
1. 次に、内容をさらに指定する次のプロンプトを入力します：

    **システムメッセージ**

    ```prompt
    あなたはメールを書くのを手助けするAIアシスタントです。
    ```
    ```prompt
    You are an AI assistant helping to write emails
    ```

    **ユーザーメッセージ:**

    ```prompt
    新しい野生動物保護施設のための宣伝メールを書いてください。次の情報を含めます：
        - 保護施設の名前はContoso
        - 象、シマウマ、キリンを専門に扱う
        - 当社のウェブサイトで寄付の呼びかけ
        - 署名の後に、現在保護施設にいる動物のリストを表形式で含めます。これらの動物には、象、シマウマ、ゴリラ、トカゲ、ジャックラビットが含まれます。
    ```
    ```prompt
    Write a promotional email for a new wildlife rescue, including the following: 
    - Rescue name is Contoso 
    - It specializes in elephants, as well as zebras and giraffes 
    - Call for donations to be given at our website 
    \n Include a list of the current animals we have at our rescue after the signature, in the form of a table. These animals include elephants, zebras, gorillas, lizards, and jackrabbits.
    ```

    または、

    ```prompt
    新しい野生動物保護施設のための宣伝メールを書いてください。次の情報を含めます：・保護施設の名前はContoso・象、シマウマ、キリンを専門に扱う・当社のウェブサイトで寄付の呼びかけ・署名の後に、現在保護施設にいる動物のリストを表形式で含めます。これらの動物には、象、シマウマ、ゴリラ、トカゲ、ジャックラビットが含まれます。
    ```
    ```prompt
    Write a promotional email for a new wildlife rescue, including the following: - Rescue name is Contoso - It specializes in elephants, as well as zebras and giraffes - Call for donations to be given at our website. Include a list of the current animals we have at our rescue after the signature, in the form of a table. These animals include elephants, zebras, gorillas, lizards, and jackrabbits.
    ```


1. 出力を観察し、明確な指示に基づいてメールがどのように変化したかを確認します。
1. 次に、システムメッセージにトーンに関する詳細を追加する次のプロンプトを入力します：

    **システムメッセージ**

    ```prompt
    あなたは新しいビジネスへの関心を高めるための宣伝メールを書くのを手助けするAIアシスタントです。あなたのトーンは軽く、雑談的で、必ず少なくとも2つのジョークを含めます。
    ```
    ```prompt
    You are an AI assistant that helps write promotional emails to generate interest in a new business. Your tone is light, chit-chat oriented and you always include at least two jokes.
    ```

    **ユーザーメッセージ:**

    ```prompt
    新しい野生動物保護施設のための宣伝メールを書いてください。次の情報を含めます：
        - 保護施設の名前はContoso
        - 象、シマウマ、キリンを専門に扱う
        - 当社のウェブサイトで寄付の呼びかけ
        - 署名の後に、現在保護施設にいる動物のリストを表形式で含めます。これらの動物には、象、シマウマ、ゴリラ、トカゲ、ジャックラビットが含まれます。
    ```
    ```prompt
    Write a promotional email for a new wildlife rescue, including the following: 
    - Rescue name is Contoso 
    - It specializes in elephants, as well as zebras and giraffes 
    - Call for donations to be given at our website 
    \n Include a list of the current animals we have at our rescue after the signature, in the form of a table. These animals include elephants, zebras, gorillas, lizards, and jackrabbits.
    ```

    または、

    ```prompt
    新しい野生動物保護施設のための宣伝メールを書いてください。次の情報を含めます：・保護施設の名前はContoso・象、シマウマ、キリンを専門に扱う・当社のウェブサイトで寄付の呼びかけ・署名の後に、現在保護施設にいる動物のリストを表形式で含めます。これらの動物には、象、シマウマ、ゴリラ、トカゲ、ジャックラビットが含まれます。
    ```
    ```prompt
    Write a promotional email for a new wildlife rescue, including the following: - Rescue name is Contoso - It specializes in elephants, as well as zebras and giraffes - Call for donations to be given at our website. Include a list of the current animals we have at our rescue after the signature, in the form of a table. These animals include elephants, zebras, gorillas, lizards, and jackrabbits.
    ```


1. 出力を観察します。今回は、形式は似ていますが、より非公式なトーンで、ジョークが含まれている可能性が高いです。
1. 最後の反復では、メール生成から逸脱し、*グラウンディングコンテキスト* を探索します。ここでは、シンプルなシステムメッセージを提供し、アプリを変更してユーザープロンプトの冒頭にグラウンディングコンテキストを提供します。アプリは次にユーザー入力を追加し、グラウンディングコンテキストから情報を抽出してユーザープロンプトに回答します。
1. `grounding.txt` ファイルを開き、挿入するグラウンディングコンテキストを簡単に読みます。
1. アプリで、コメント ***Format and send the request to the model*** の直後、既存のコードの前に、`grounding.txt` からテキストを読み込み、ユーザープロンプトにグラウンディングコンテキストを追加するための次のコードスニペットを追加します。

    **C#**: Program.cs

    ```csharp
    // Format and send the request to the model
    Console.WriteLine("\nAdding grounding context from grounding.txt");
    string groundingText = System.IO.File.ReadAllText("grounding.txt");
    userMessage = groundingText + userMessage;
    ```

    **Python**: prompt-engineering.py

    ```python
    # Format and send the request to the model
    print("\nAdding grounding context from grounding.txt")
    grounding_text = open(file="grounding.txt", encoding="utf8").read().strip()
    user_message = grounding_text + user_message
    ```

1. ファイルを保存し、アプリを再実行します。
1. 次のプロンプトを入力します（**システムメッセージ** は引き続き `system.txt` に入力して保存します）。

   **システムメッセージ**

   ```prompt
    あなたは情報を見つけるのを手助けするAIアシスタントです。プロンプトで提供されたテキストから回答を提供し、簡潔に応答します。
   ```
    ```prompt
    You're an AI assistant who helps people find information. You'll provide answers from the text provided in the prompt, and respond concisely.
    ```

   **ユーザーメッセージ:**

    ```prompt
    Contosoで子どもたちに人気のある動物は何ですか？
    ```
    ```prompt
    What animal is the favorite of children at Contoso?
    ```

> **ヒント**: Azure OpenAI からの完全な応答を表示したい場合は、**printFullResponse** 変数を `True` に設定し、アプリを再実行できます。

## クリーンアップ

Azure OpenAI リソースの使用が終わったら、**Azure ポータル** (`https://portal.azure.com`) でデプロイメントまたはリソース全体を削除することを忘れないでください。
