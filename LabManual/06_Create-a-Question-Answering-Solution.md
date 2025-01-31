---
lab:
    title: '質問応答ソリューションを作成する'
    module: 'モジュール 6 - Azure AI Language を使用して質問応答ソリューションを作成する'
---

# 質問応答ソリューションを作成する

最も一般的な会話シナリオの1つは、よくある質問 (FAQ) のナレッジベースを通じてサポートを提供することです。多くの組織は、FAQ をドキュメントやウェブページとして公開しており、少数の質問と回答のペアには効果的ですが、大規模なドキュメントは検索が難しく時間がかかることがあります。
**Azure AI Language** には、質問と回答のペアのナレッジベースを作成し、自然言語入力を使用してクエリを実行できる *質問応答* 機能が含まれており、ユーザーが提出した質問に対する回答をボットが検索するためのリソースとして最も一般的に使用されます。

## *Azure AI Language* リソースをプロビジョニングする

サブスクリプションにまだリソースがない場合は、**Azure AI Language サービス** リソースをプロビジョニングする必要があります。さらに、質問応答のためのナレッジベースを作成およびホストするには、**Question Answering** 機能を有効にする必要があります。

1. `https://portal.azure.com` で Azure ポータルを開き、Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。
1. **リソースの作成** を選択します。
1. 検索フィールドで **Language service** を検索します。結果で **Language Service** の下の **作成** を選択します。
1. **Custom question answering** ブロックを選択します。次に **リソースの作成を続行** を選択します。次の設定を入力する必要があります:
    - **サブスクリプション**: *Azure サブスクリプション*
    - **リソースグループ**: *リソースグループを選択または作成*。
    - **リージョン**: *利用可能な場所を選択*
    - **名前**: *一意の名前を入力*
    - **価格レベル**: **F0** (*無料*) または **S** (*標準*) を選択します。F が利用できない場合。
    - **Azure Search リージョン**: *Language リソースと同じグローバルリージョンの場所を選択*
    - **Azure Search 価格レベル**: Free (F) (*このレベルが利用できない場合は、Basic (B) を選択*)
    - **Responsible AI Notice**: *同意*

1. **作成 + レビュー** を選択し、次に **作成** を選択します。
    > **注**: カスタム質問応答は、質問と回答のナレッジベースをインデックス化およびクエリするために Azure Search を使用します。
1. デプロイメントが完了するのを待ち、デプロイされたリソースに移動します。
1. **キーとエンドポイント** ページを表示します。この演習の後半でこのページの情報が必要になります。

## 質問応答プロジェクトを作成する

Azure AI Language リソースで質問応答のためのナレッジベースを作成するには、Language Studio ポータルを使用して質問応答プロジェクトを作成できます。この場合、Microsoft Learn に関する質問と回答を含むナレッジベースを作成します。

1. 新しいブラウザタブで https://language.cognitive.azure.com/ にアクセスし、Azure サブスクリプションに関連付けられている Microsoft アカウントでサインインします。
1. 言語リソースの選択を求められた場合は、次の設定を選択します:
    - **Azure Directory**: サブスクリプションを含む Azure ディレクトリ。
    - **Azure サブスクリプション**: Azure サブスクリプション。
    - **リソースタイプ**: Language
    - **リソース名**: 以前に作成した Azure AI Language リソース。

    言語リソースの選択を求められない場合は、サブスクリプションに複数の Language リソースがある可能性があります。その場合は次の手順に従います:
    1. ページ上部のバーで **設定 (⚙)** ボタンを選択します。
    2. **設定** ページで **リソース** タブを表示します。
    3. 作成した言語リソースを選択し、**リソースの切り替え** をクリックします。
    4. ページ上部で **Language Studio** をクリックして Language Studio ホームページに戻ります。

1. ポータルの上部で **新規作成** メニューから **カスタム質問応答** を選択します。
1. **プロジェクトの作成** ウィザードの **言語設定の選択** ページで、**すべてのプロジェクトの言語を選択** オプションを選択し、言語として **英語** を選択します。次に **次へ** を選択します。
1. **基本情報の入力** ページで、次の詳細を入力します:
    - **名前**: `LearnFAQ`
    - **説明**: `FAQ for Microsoft Learn`
    - **回答が返されない場合のデフォルトの回答**: `Sorry, I don't understand the question`
1. **次へ** を選択します。
1. **確認と完了** ページで **プロジェクトの作成** を選択します。

## ナレッジベースにソースを追加する

ナレッジベースをゼロから作成することもできますが、既存の FAQ ページやドキュメントから質問と回答をインポートすることが一般的です。この場合、Microsoft Learn の既存の FAQ ウェブページからデータをインポートし、一般的な会話のやり取りをサポートするために事前定義された「雑談」質問と回答もインポートします。

1. 質問応答プロジェクトの **ソースの管理** ページで、**╋ ソースを追加** リストから **URL** を選択します。次に **URL の追加** ダイアログボックスで **╋ URL を追加** を選択し、次の名前と URL を設定してから **すべて追加** を選択してナレッジベースに追加します:
    - **名前**: `Learn FAQ Page`
    - **URL**: `https://docs.microsoft.com/en-us/learn/support/faq`

1. 質問応答プロジェクトの **ソースの管理** ページで、**╋ ソースを追加** リストから **雑談(chit chat)** を選択します。次に **雑談の追加** ダイアログボックスで **フレンドリー** を選択し、**雑談の追加** を選択します。

## ナレッジベースを編集する

ナレッジベースには、Microsoft Learn FAQ からの質問と回答のペアが入力され、会話の *雑談* 質問と回答のペアが補足されています。追加の質問と回答のペアを追加してナレッジベースを拡張できます。

1. Language Studio の **LearnFAQ** プロジェクトで、**ナレッジベースの編集** ページを選択して既存の質問と回答のペアを表示します (ヒントが表示された場合は、読んで **了解** を選択して閉じるか、**すべてスキップ** を選択します)。
1. ナレッジベースの **質問と回答のペア** タブで **＋** を選択し、次の設定で新しい質問と回答のペアを作成します:
    - **ソース**: `https://docs.microsoft.com/en-us/learn/support/faq`
    - **質問**: `What are Microsoft credentials?`
    - **回答**: `Microsoft credentials enable you to validate and prove your skills with Microsoft technologies.`

1. **完了** を選択します。
1. 作成された **What are Microsoft credentials?** 質問のページで、**代替質問(Alternate questions)** を展開します。次に、代替質問として `How can I demonstrate my Microsoft technology skills?` を追加します。

場合によっては、ユーザーが回答をフォローアップできるように、*マルチターン* 会話を作成して、ユーザーが必要な回答にたどり着くまで質問を反復的に絞り込むことが理にかなっています。

1. 認定質問の回答の下で、**フォローアップ プロンプト** を展開し、次のフォローアップ プロンプトを追加します:
    - **ユーザーに表示されるプロンプトのテキスト**: `Learn more about credentials`
    - **新しいペアへのリンクを作成** タブを選択し、次のテキストを入力します: `You can learn more about credentials on [the Microsoft credentials page](https://docs.microsoft.com/learn/credentials/).`
    - **コンテキスト フローでのみ表示** を選択します。このオプションを選択すると、元の認定質問からのフォローアップ質問のコンテキストでのみ回答が返されます。

1. **プロンプトを追加** を選択します。

## ナレッジベースをトレーニングおよびテストする

ナレッジベースが作成されたので、Language Studio でテストできます。

1. 左側の **質問と回答のペア** タブの下にある **保存** ボタンを選択して、ナレッジベースへの変更を保存します。
1. 変更が保存されたら、**テスト** ボタンを選択してテスト ペインを開きます。
1. テスト ペインの上部で **短い回答の応答を含める** を選択解除します (すでに選択解除されている場合はそのまま)。次に、下部にメッセージ `Hello` を入力します。適切な応答が返されるはずです。
1. テスト ペインの下部にメッセージ `What is Microsoft Learn?` を入力します。FAQ から適切な応答が返されるはずです。
1. メッセージ `Thanks!` を入力します。適切な雑談の応答が返されるはずです。
1. メッセージ `Tell me about Microsoft credentials` を入力します。作成した回答が返され、フォローアップ プロンプトのリンクが表示されるはずです。
1. **Learn more about credentials** フォローアップ リンクを選択します。認定ページへのリンクを含むフォローアップ回答が返されるはずです。
1. ナレッジベースのテストが完了したら、テスト ペインを閉じます。

## ナレッジベースをデプロイする

ナレッジベースは、クライアント アプリケーションが質問に回答するために使用できるバックエンド サービスを提供します。ナレッジベースを公開し、クライアントからその REST インターフェイスにアクセスする準備が整いました。

1. Language Studio の **LearnFAQ** プロジェクトで、左側のナビゲーション メニューから **ナレッジベースのデプロイ** ページを選択します。
1. ページの上部で **デプロイ** を選択します。次に、ナレッジベースをデプロイすることを確認するために **デプロイ** を選択します。
1. デプロイメントが完了したら、**予測 URL を取得** を選択して、ナレッジベースの REST エンドポイントを表示し、サンプル リクエストに次のパラメーターが含まれていることを確認します:
    - **projectName**: プロジェクトの名前 (おそらく *LearnFAQ*)
    - **deploymentName**: デプロイメントの名前 (おそらく *production*)

1. 予測 URL ダイアログ ボックスを閉じます。

## Visual Studio Code でアプリを開発する準備をする

Visual Studio Code を使用して質問応答アプリを開発します。アプリのコードファイルは GitHub リポジトリに提供されています。

> **ヒント**: すでに **mslearn-ai-language** リポジトリをクローンしている場合は、Visual Studio Code で開きます。そうでない場合は、次の手順に従って開発環境にクローンします。

1. Visual Studio Code を起動します。
2. パレット (SHIFT+CTRL+P) を開き、**Git: Clone** コマンドを実行して `https://github.com/MicrosoftLearning/mslearn-ai-language` リポジトリをローカルフォルダーにクローンします (フォルダーはどこでも構いません)。
3. リポジトリがクローンされたら、Visual Studio Code でフォルダーを開きます。
    > **注**: Visual Studio Code がコードを信頼するように求めるポップアップメッセージを表示した場合は、ポップアップで **Yes, I trust the authors** オプションをクリックします。
4. リポジトリ内の C# コードプロジェクトをサポートするための追加ファイルがインストールされるのを待ちます。
    > **注**: ビルドおよびデバッグに必要なアセットを追加するように求められた場合は、**Not Now** を選択します。

## アプリケーションを構成する

C# と Python の両方のアプリケーションが提供されており、要約をテストするためのサンプルテキストファイルも提供されています。両方のアプリは同じ機能を備えています。まず、Azure AI Language リソースを使用できるようにアプリケーションのいくつかの重要な部分を完成させます。

1. Visual Studio Code の **Explorer** ペインで、**Labfiles/02-qna** フォルダーに移動し、希望する言語に応じて **CSharp** または **Python** フォルダーとその中の **qna-app** フォルダーを展開します。各フォルダーには、Azure AI Language 質問応答機能を統合するための言語固有のファイルが含まれています。
2. コードファイルを含む **qna-app** フォルダーを右クリックして統合ターミナルを開き、希望する言語に応じて次のコマンドを実行して Azure AI Language 質問応答 SDK パッケージをインストールします:

    **C#**:

    ```
    dotnet add package Azure.AI.Language.QuestionAnswering
    ```

    **Python**:

    ```
    pip install azure-ai-language-questionanswering
    ```

3. **Explorer** ペインで、**qna-app** フォルダー内の構成ファイルを開きます:
    - **C#**: appsettings.json
    - **Python**: .env

4. 構成値を更新して、作成した Azure Language リソースの **エンドポイント** と **キー** を含めます (Azure ポータルの Azure AI Language リソースの **キーとエンドポイント** ページで利用可能)。デプロイされたナレッジベースのプロジェクト名とデプロイメント名もこのファイルに含める必要があります。
5. 構成ファイルを保存します。

## アプリケーションにコードを追加する

必要な SDK ライブラリをインポートし、デプロイされたプロジェクトへの認証接続を確立し、質問を送信するために必要なコードを追加する準備が整いました。

1. **qna-app** フォルダーにはクライアントアプリケーションのコードファイルが含まれています:
    - **C#**: Program.cs
    - **Python**: qna-app.py
    コードファイルを開き、既存の名前空間参照の下にある **Import namespaces** コメントの下に、Text Analytics SDK を使用するために必要な名前空間をインポートするための次の言語固有のコードを追加します:

    **C#**: Programs.cs

    ```csharp
    // import namespaces
    using Azure;
    using Azure.AI.Language.QuestionAnswering;
    ```

    **Python**: qna-app.py

    ```python
    # import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.language.questionanswering import QuestionAnsweringClient
    ```

1. **Main** 関数で、構成ファイルから Azure AI Language サービスのエンドポイントとキーを読み込むコードがすでに提供されていることを確認します。次に、**Create client using endpoint and key** コメントを見つけ、Text Analysis API のクライアントを作成するための次のコードを追加します:

    **C#**: Programs.cs

    ```C#
    // Create client using endpoint and key
    AzureKeyCredential credentials = new AzureKeyCredential(aiSvcKey);
    Uri endpoint = new Uri(aiSvcEndpoint);
    QuestionAnsweringClient aiClient = new QuestionAnsweringClient(endpoint, credentials);
    ```

    **Python**: qna-app.py

    ```Python
    # Create client using endpoint and key
    credential = AzureKeyCredential(ai_key)
    ai_client = QuestionAnsweringClient(endpoint=ai_endpoint, credential=credential)
    ```

1. **Main** 関数で、**Submit a question and display the answer** コメントを見つけ、コマンドラインから質問を繰り返し読み取り、サービスに送信し、回答の詳細を表示するための次のコードを追加します:

    **C#**: Programs.cs

    ```C#
    // Submit a question and display the answer
    string user_question = "";
    while (user_question.ToLower() != "quit")
        {
            Console.Write("Question: ");
            user_question = Console.ReadLine();
            QuestionAnsweringProject project = new QuestionAnsweringProject(projectName, deploymentName);
            Response<AnswersResult> response = aiClient.GetAnswers(user_question, project);
            foreach (KnowledgeBaseAnswer answer in response.Value.Answers)
            {
                Console.WriteLine(answer.Answer);
                Console.WriteLine($"Confidence: {answer.Confidence:P2}");
                Console.WriteLine($"Source: {answer.Source}");
                Console.WriteLine();
            }
        }
    ```

    **Python**: qna-app.py

    ```Python
    # Submit a question and display the answer
    user_question = ''
    while user_question.lower() != 'quit':
        user_question = input('\nQuestion:\n')
        response = ai_client.get_answers(question=user_question,
                                        project_name=ai_project_name,
                                        deployment_name=ai_deployment_name)
        for candidate in response.answers:
            print(candidate.answer)
            print("Confidence: {}".format(candidate.confidence))
            print("Source: {}".format(candidate.source))
    ```

1. 変更を保存し、**qna-app** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します:
    - **C#**: `dotnet run`
    - **Python**: `python qna-app.py`
    > **ヒント**: ターミナルツールバーの **Maximize panel size** (**^**) アイコンを使用して、コンソールテキストの表示領域を広げることができます。

1. プロンプトが表示されたら、質問応答プロジェクトに送信する質問を入力します。例えば `What is a learning path?`。
1. 返された回答を確認します。
1. さらに質問をします。終了するには `quit` と入力します。

## リソースのクリーンアップ

Azure AI Language サービスの探索が終了したら、この演習で作成したリソースを削除できます。以下の手順に従ってください。

1. `https://portal.azure.com` で Azure ポータルを開き、Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。
2. このラボで作成した Azure AI Language リソースに移動します。
3. リソース ページで **削除** を選択し、リソースを削除する手順に従います。

## 詳細情報

Azure AI Language での質問応答の詳細については、[Azure AI Language ドキュメント](https://learn.microsoft.com/azure/ai-services/language-service/question-answering/overview)を参照してください。
