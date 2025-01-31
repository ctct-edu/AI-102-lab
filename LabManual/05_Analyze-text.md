---
lab:
    title: 'テキストを分析する'
    module: 'モジュール 3 - 自然言語処理ソリューションの開発'

---

# テキストを分析する

**Azure Language** は、言語検出、感情分析、キーフレーズ抽出、およびエンティティ認識を含むテキストの分析をサポートします。

例えば、旅行代理店が会社のウェブサイトに投稿されたホテルのレビューを処理したいとします。Azure AI Language を使用することで、各レビューが書かれている言語、レビューの感情（ポジティブ、中立、ネガティブ）、レビューで議論されている主なトピックを示すキーフレーズ、およびレビューで言及されている場所、ランドマーク、人物などの名前付きエンティティを特定できます。

## *Azure AI Language* リソースをプロビジョニングする

サブスクリプションにまだリソースがない場合は、Azure サブスクリプションに **Azure AI Language サービス** リソースをプロビジョニングする必要があります。

1. `https://portal.azure.com` で Azure ポータルを開き、Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。
1. **リソースの作成** を選択します。
1. 検索フィールドで **Language service** を検索します。結果で **Language Service** の下の **作成** を選択します。
1. **リソースの作成を続行** を選択します。
1. 次の設定を使用してリソースをプロビジョニングします:
    - **サブスクリプション**: *Azure サブスクリプション*。
    - **リソースグループ**: *リソースグループを選択または作成*。
    - **リージョン**: *利用可能なリージョンを選択*
    - **名前**: *一意の名前を入力*。
    - **価格レベル**: **F0** (*無料*) または **S** (*標準*) を選択します。F が利用できない場合。
    - **Responsible AI Notice**: 同意します。
1. **確認と作成** を選択し、次に **作成** を選択してリソースをプロビジョニングします。
1. デプロイメントが完了するのを待ち、デプロイされたリソースに移動します。
1. **リソース管理** セクションの **キーとエンドポイント** ページを表示します。この演習の後半でこのページの情報が必要になります。

## Visual Studio Code でアプリを開発する準備をする

Visual Studio Code を使用してテキスト分析アプリを開発します。アプリのコードファイルは GitHub リポジトリに提供されています。

> **ヒント**: すでに **mslearn-ai-language** リポジトリをクローンしている場合は、Visual Studio Code で開きます。そうでない場合は、次の手順に従って開発環境にクローンします。

1. Visual Studio Code を起動します。
2. パレット (SHIFT+CTRL+P) を開き、**Git: Clone** コマンドを実行して `https://github.com/MicrosoftLearning/mslearn-ai-language` リポジトリをローカルフォルダーにクローンします (フォルダーはどこでも構いません)。
3. リポジトリがクローンされたら、Visual Studio Code でフォルダーを開きます。
    > **注**: Visual Studio Code がコードを信頼するように求めるポップアップメッセージを表示した場合は、ポップアップで **Yes, I trust the authors** オプションをクリックします。
4. リポジトリ内の C# コードプロジェクトをサポートするための追加ファイルがインストールされるのを待ちます。
    > **注**: ビルドおよびデバッグに必要なアセットを追加するように求められた場合は、**Not Now** を選択します。

## アプリケーションを構成する

C# と Python の両方のアプリケーションが提供されており、要約をテストするためのサンプルテキストファイルも提供されています。両方のアプリは同じ機能を備えています。まず、Azure AI Language リソースを使用できるようにアプリケーションのいくつかの重要な部分を完成させます。

1. Visual Studio Code の **Explorer** ペインで、**Labfiles/01-analyze-text** フォルダーに移動し、希望する言語に応じて **CSharp** または **Python** フォルダーとその中の **text-analysis** フォルダーを展開します。各フォルダーには、Azure AI Language テキスト分析機能を統合するための言語固有のファイルが含まれています。
2. コードファイルを含む **text-analysis** フォルダーを右クリックして統合ターミナルを開き、希望する言語に応じて次のコマンドを実行して Azure AI Language Text Analytics SDK パッケージをインストールします。Python の演習では、`dotenv` パッケージもインストールします:

    **C#**:

    ```
    dotnet add package Azure.AI.TextAnalytics --version 5.3.0
    ```

    **Python**:

    ```
    pip install azure-ai-textanalytics==5.3.0
    pip install python-dotenv
    ```

3. **Explorer** ペインで、**text-analysis** フォルダー内の構成ファイルを開きます:
    - **C#**: appsettings.json
    - **Python**: .env

4. 構成値を更新して、作成した Azure Language リソースの **エンドポイント** と **キー** を含めます (Azure ポータルの Azure AI Language リソースの **キーとエンドポイント** ページで利用可能)。
5. 構成ファイルを保存します。
6. **text-analysis** フォルダーにはクライアントアプリケーションのコードファイルが含まれています:
    - **C#**: Program.cs
    - **Python**: text-analysis.py
   
    コードファイルを開き、既存の名前空間参照の下にある **Import namespaces** コメントの下に、Text Analytics SDK を使用するために必要な名前空間をインポートするための次の言語固有のコードを追加します:

    **C#**: Programs.cs

    ```csharp
    // import namespaces
    using Azure;
    using Azure.AI.TextAnalytics;
    ```

    **Python**: text-analysis.py

    ```python
    # import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.textanalytics import TextAnalyticsClient
    ```

7. **Main** 関数で、構成ファイルから Azure AI Language サービスのエンドポイントとキーを読み込むコードがすでに提供されていることを確認します。次に、**Create client using endpoint and key** コメントを見つけ、Text Analysis API のクライアントを作成するための次のコードを追加します:

    **C#**: Programs.cs

    ```C#
    // Create client using endpoint and key
    AzureKeyCredential credentials = new AzureKeyCredential(aiSvcKey);
    Uri endpoint = new Uri(aiSvcEndpoint);
    TextAnalyticsClient aiClient = new TextAnalyticsClient(endpoint, credentials);
    ```

    **Python**: text-analysis.py

    ```Python
    # Create client using endpoint and key
    credential = AzureKeyCredential(ai_key)
    ai_client = TextAnalyticsClient(endpoint=ai_endpoint, credential=credential)
    ```

8. 変更を保存し、**text-analysis** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します:
    - **C#**: `dotnet run`
    - **Python**: `python text-analysis.py`
    > **ヒント**: ターミナルツールバーの **Maximize panel size** (**^**) アイコンを使用して、コンソールテキストの表示領域を広げることができます。

9. コードがエラーなしで実行され、**reviews** フォルダー内の各レビュー テキストファイルの内容が表示されることを確認します。アプリケーションは Text Analytics API のクライアントを正常に作成しますが、それを使用していません。次の手順でこれを修正します。

## 言語を検出するコードを追加する

API のクライアントを作成したので、それを使用して各レビューが書かれている言語を検出しましょう。

1. プログラムの **Main** 関数で、**Get language** コメントを見つけます。その下に、各レビュー ドキュメントの言語を検出するために必要なコードを追加します:

    **C#**: Programs.cs

    ```csharp
    // Get language
    DetectedLanguage detectedLanguage = aiClient.DetectLanguage(text);
    Console.WriteLine($"\nLanguage: {detectedLanguage.Name}");
    ```

    **Python**: text-analysis.py

    ```python
    # Get language
    detectedLanguage = ai_client.detect_language(documents=[text])[0]
    print('\nLanguage: {}'.format(detectedLanguage.primary_language.name))
    ```

   > **注**: *この例では、各レビューが個別に分析され、各ファイルに対してサービスへの個別の呼び出しが行われます。別のアプローチとして、ドキュメントのコレクションを作成し、単一の呼び出しでサービスに渡すことができます。どちらのアプローチでも、サービスからの応答はドキュメントのコレクションで構成されます。そのため、上記の Python コードでは、応答内の最初の (および唯一の) ドキュメントのインデックス ([0]) が指定されています。*

1. 変更を保存します。次に、**text-analysis** フォルダーの統合ターミナルに戻り、プログラムを再実行します。
1. 出力を確認し、今回は各レビューの言語が識別されていることを確認します。

## 感情を評価するコードを追加する

*感情分析* は、テキストを *ポジティブ* または *ネガティブ* (または場合によっては *中立* または *混合*) と分類するために一般的に使用される手法です。ソーシャルメディアの投稿、製品レビュー、その他のテキストの感情が有用な洞察を提供する可能性があるアイテムを分析するためによく使用されます。

1. プログラムの **Main** 関数で、**Get sentiment** コメントを見つけます。その下に、各レビュー ドキュメントの感情を検出するために必要なコードを追加します:

    **C#**: Program.cs

    ```csharp
    // Get sentiment
    DocumentSentiment sentimentAnalysis = aiClient.AnalyzeSentiment(text);
    Console.WriteLine($"\nSentiment: {sentimentAnalysis.Sentiment}");
    ```

    **Python**: text-analysis.py

    ```python
    # Get sentiment
    sentimentAnalysis = ai_client.analyze_sentiment(documents=[text])[0]
    print("\nSentiment: {}".format(sentimentAnalysis.sentiment))
    ```

1. 変更を保存します。次に、**text-analysis** フォルダーの統合ターミナルに戻り、プログラムを再実行します。
1. 出力を確認し、レビューの感情が検出されていることを確認します。

## キーフレーズを識別するコードを追加する

テキストの主なトピックを判断するのに役立つキーフレーズを識別することは有用です。

1. プログラムの **Main** 関数で、**Get key phrases** コメントを見つけます。その下に、各レビュー ドキュメントのキーフレーズを検出するために必要なコードを追加します:

    **C#**: Program.cs

    ```csharp
    // Get key phrases
    KeyPhraseCollection phrases = aiClient.ExtractKeyPhrases(text);
    if (phrases.Count > 0)
    {
        Console.WriteLine("\nKey Phrases:");
        foreach(string phrase in phrases)
        {
            Console.WriteLine($"\t{phrase}");
        }
    }
    ```

    **Python**: text-analysis.py

    ```python
    # Get key phrases
    phrases = ai_client.extract_key_phrases(documents=[text])[0].key_phrases
    if len(phrases) > 0:
        print("\nKey Phrases:")
        for phrase in phrases:
            print('\t{}'.format(phrase))
    ```

1. 変更を保存します。次に、**text-analysis** フォルダーの統合ターミナルに戻り、プログラムを再実行します。
1. 出力を確認し、各ドキュメントにレビューの内容に関する洞察を提供するキーフレーズが含まれていることを確認します。

## エンティティを抽出するコードを追加する

多くの場合、ドキュメントやその他のテキストには、人、場所、期間、その他のエンティティが言及されています。Text Analytics API は、テキスト内の複数のカテゴリ (およびサブカテゴリ) のエンティティを検出できます。

1. プログラムの **Main** 関数で、**Get entities** コメントを見つけます。その下に、各レビューで言及されているエンティティを識別するために必要なコードを追加します:

    **C#**: Program.cs

    ```csharp
    // Get entities
    CategorizedEntityCollection entities = aiClient.RecognizeEntities(text);
    if (entities.Count > 0)
    {
        Console.WriteLine("\nEntities:");
        foreach(CategorizedEntity entity in entities)
        {
            Console.WriteLine($"\t{entity.Text} ({entity.Category})");
        }
    }
    ```

    **Python**: text-analysis.py

    ```python
    # Get entities
    entities = ai_client.recognize_entities(documents=[text])[0].entities
    if len(entities) > 0:
        print("\nEntities")
        for entity in entities:
            print('\t{} ({})'.format(entity.text, entity.category))
    ```

1. 変更を保存します。次に、**text-analysis** フォルダーの統合ターミナルに戻り、プログラムを再実行します。
1. 出力を確認し、テキスト内で検出されたエンティティを確認します。

## リンクされたエンティティを抽出するコードを追加する

カテゴリ化されたエンティティに加えて、Text Analytics API は、Wikipedia などのデータソースへの既知のリンクがあるエンティティを検出できます。

1. プログラムの **Main** 関数で、**Get linked entities** コメントを見つけます。その下に、各レビューで言及されているリンクされたエンティティを識別するために必要なコードを追加します:


    **C#**: Program.cs

    ```csharp
    // Get linked entities
    LinkedEntityCollection linkedEntities = aiClient.RecognizeLinkedEntities(text);
    if (linkedEntities.Count > 0)
    {
        Console.WriteLine("\nLinks:");
        foreach(LinkedEntity linkedEntity in linkedEntities)
        {
            Console.WriteLine($"\t{linkedEntity.Name} ({linkedEntity.Url})");
        }
    }
    ```

    **Python**: text-analysis.py

    ```python
    # Get linked entities
    entities = ai_client.recognize_linked_entities(documents=[text])[0].entities
    if len(entities) > 0:
        print("\nLinks")
        for linked_entity in entities:
            print('\t{} ({})'.format(linked_entity.name, linked_entity.url))
    ```

1. 変更を保存します。次に、**text-analysis** フォルダーの統合ターミナルに戻り、プログラムを再実行します。
1. 出力を確認し、識別されたリンクされたエンティティを確認します。

## リソースのクリーンアップ

Azure AI Language サービスの探索が終了したら、この演習で作成したリソースを削除できます。以下の手順に従ってください。

1. `https://portal.azure.com` で Azure ポータルを開き、Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。
2. このラボで作成した Azure AI Language リソースに移動します。
3. リソース ページで **削除** を選択し、リソースを削除する手順に従います。

## 詳細情報

**Azure AI Language** の使用に関する詳細については、[ドキュメント](https://learn.microsoft.com/azure/ai-services/language-service/)を参照してください。
