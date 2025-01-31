---
lab:
 title: 'Azure AI Search のカスタムスキルを作成する'
 module: 'モジュール 12 - ナレッジマイニングソリューションの作成'
---

# Azure AI Search のカスタムスキルを作成する

Azure AI Search は、AI スキルのエンリッチメントパイプラインを使用して、ドキュメントから AI 生成フィールドを抽出し、それらを検索インデックスに含めます。使用できる包括的なビルトインスキルのセットがありますが、これらのスキルでは満たされない特定の要件がある場合は、カスタムスキルを作成できます。

この演習では、ドキュメント内の個々の単語の頻度を集計して、最も使用されている上位 5 つの単語のリストを生成するカスタムスキルを作成し、それを架空の旅行代理店 Margie's Travel の検索ソリューションに追加します。

## Visual Studio Code でアプリを開発する準備

Visual Studio Code を使用して検索アプリを開発します。アプリのコードファイルは GitHub リポジトリに提供されています。

> **ヒント**: すでに **mslearn-knowledge-mining** リポジトリをクローンしている場合は、Visual Studio Code で開きます。そうでない場合は、次の手順に従って開発環境にクローンします。

1. Visual Studio Code を起動します。
1. パレット（SHIFT+CTRL+P）を開き、**Git: Clone** コマンドを実行して `https://github.com/MicrosoftLearning/mslearn-knowledge-mining` リポジトリをローカルフォルダにクローンします（フォルダはどこでも構いません）。
1. リポジトリがクローンされたら、Visual Studio Code でフォルダを開きます。
1. リポジトリ内の C# コードプロジェクトをサポートするための追加ファイルがインストールされるのを待ちます。

    > **注意**: ビルドとデバッグに必要なアセットを追加するように求められた場合は、**今は追加しない** を選択します。

## Azure リソースの作成

> **注意**: 以前に **Azure AI Search ソリューションの作成** 演習を完了し、これらの Azure リソースがサブスクリプションにまだ存在する場合は、このセクションをスキップして **検索ソリューションの作成** セクションから開始できます。それ以外の場合は、以下の手順に従って必要な Azure リソースをプロビジョニングします。

1. Web ブラウザで Azure ポータル (`https://portal.azure.com`) を開き、Azure サブスクリプションに関連付けられた Microsoft アカウントを使用してサインインします。
2. 上部の検索バーで *Azure AI services* を検索し、**Azure AI services multi-service account** を選択し、次の設定で Azure AI services multi-service account リソースを作成します：
    - **サブスクリプション**: *Azure サブスクリプション*
    - **リソースグループ**: *リソースグループを選択または作成（制限付きサブスクリプションを使用している場合、新しいリソースグループを作成する権限がない場合があります。その場合は提供されたものを使用します）*
    - **リージョン**: *地理的に近い利用可能なリージョンから選択*
    - **名前**: *一意の名前を入力*
    - **価格レベル**: Standard S0

1. デプロイメントが完了したら、リソースに移動し、**概要** ページで **サブスクリプション ID** と **場所** を確認します。これらの値は、後の手順で必要になります。
1. Visual Studio Code で **Labfiles/02-search-skill** フォルダを展開し、**setup.cmd** を選択します。このバッチスクリプトを使用して、必要な Azure リソースを作成するために必要な Azure コマンドラインインターフェイス（CLI）コマンドを実行します。
1. **02-search-skill** フォルダを右クリックし、**統合ターミナルで開く** を選択します。
1. ターミナルペインで、次のコマンドを入力して、Azure サブスクリプションへの認証接続を確立します。

    ```powershell
    az login --output none
    ```

8. プロンプトが表示されたら、Azure サブスクリプションにサインインまたは選択します。次に、Visual Studio Code に戻り、サインインプロセスが完了するのを待ちます。
9. 次のコマンドを実行して、Azure の場所を一覧表示します。

    ```powershell
    az account list-locations -o table
    ```


10. 出力で、リソースグループの場所に対応する **Name** 値を見つけます（例：*East US* の対応する名前は *eastus* です）。

    > **重要**: **Name** 値を記録し、**location** 変数に設定します。

11. **setup.cmd** スクリプトで、**subscription_id**、**resource_group**、および **location** 変数の宣言を、サブスクリプション ID、リソースグループ名、および場所名の適切な値に変更します。変更を保存します。
     > **注意** 値に空白が含まれる場合は、"で囲んでください（USキーボードの場合、Shift + *で入力できます）。

12. **02-search-skill** フォルダのターミナルで、次のコマンドを入力してスクリプトを実行します：

    ```powershell
    ./setup
    ```

    > **注意**: スクリプトが失敗した場合は、正しい変数名で保存したことを確認し、再試行してください。

13. スクリプトが完了したら、出力を確認し、Azure リソースに関する次の情報をメモします（後でこれらの値が必要になります）：
    - ストレージアカウント名
    - ストレージ接続文字列
    - 検索サービスエンドポイント
    - 検索サービス管理キー
    - 検索サービスクエリキー

14. Azure ポータルでリソースグループを更新し、Azure Storage アカウント、Azure AI Services リソース、および Azure AI Search リソースが含まれていることを確認します。

## 検索ソリューションの作成

必要な Azure リソースが揃ったので、次のコンポーネントで構成される検索ソリューションを作成できます：

- Azure ストレージコンテナー内のドキュメントを参照する **データソース**。
- ドキュメントから AI 生成フィールドを抽出するスキルのエンリッチメントパイプラインを定義する **スキルセット**。
- 検索可能なドキュメントレコードのセットを定義する **インデックス**。
- データソースからドキュメントを抽出し、スキルセットを適用し、インデックスをポピュレートする **インデクサー**。

この演習では、Azure AI Search REST インターフェイスを使用して、これらのコンポーネントを JSON リクエストを送信して作成します。

1. Visual Studio Code で **02-search-skill** フォルダ内の **create-search** フォルダを展開し、**data_source.json** を選択します。このファイルには **margies-custom-data** という名前のデータソースの JSON 定義が含まれています。
2. **YOUR_CONNECTION_STRING** プレースホルダーを Azure ストレージアカウントの接続文字列に置き換えます。

    *接続文字列は、**setup.cmd**の出力やAzure ポータルのストレージアカウントの **アクセスキー** ページから確認できます*

3. 更新された JSON ファイルを保存して閉じます。
4. **create-search** フォルダ内の **skillset.json** を開きます。このファイルには **margies-custom-skillset** という名前のスキルセットの JSON 定義が含まれています。
5. スキルセット定義の上部にある **cognitiveServices** 要素で、**YOUR_AI_SERVICES_KEY** プレースホルダーを Azure AI Services リソースのいずれかのキーに置き換えます。

    *キーは **setup.cmd**の出力やAzure AI Services リソースの **キーとエンドポイント** ページで確認できます。*

6. 更新された JSON ファイルを保存して閉じます。
7. **create-search** フォルダーで **index.json** を開きます。このファイルには **margies-custom-index** という名前のインデックスの JSON 定義が含まれています。
8. インデックスの JSON を確認し、変更を加えずにファイルを閉じます。
9. **create-search** フォルダーで **indexer.json** を開きます。このファイルには **margies-custom-indexer** という名前のインデクサーの JSON 定義が含まれています。
10. インデクサーの JSON を確認し、変更を加えずにファイルを閉じます。
11. **create-search** フォルダーで **create-search.cmd** を開きます。このバッチスクリプトは、cURL ユーティリティを使用して JSON 定義を Azure AI Search リソースの REST インターフェイスに送信します。
12. **YOUR_SEARCH_URL** および **YOUR_ADMIN_KEY** 変数プレースホルダーを、Azure AI Search リソースの **Url** および管理キーのいずれかに置き換えます。

    *これらの値は、**setup.cmd**の出力やAzure AI Search リソースの **概要** および **キー** ページで確認できます。*

13. 更新されたバッチファイルを保存します。
14. **create-search** フォルダーを右クリックし、**統合ターミナルで開く** を選択します。
15. **create-search** フォルダーのターミナルペインで、次のコマンドを入力してバッチスクリプトを実行します。

    ```powershell
    ./create-search
    ```

16. スクリプトが完了したら、Azure ポータルで Azure AI Search リソースのページに移動し、**インデクサー** ページを選択してインデックス作成プロセスが完了するのを待ちます。

    *インデックス作成操作の進行状況を追跡するには、**更新** を選択できます。完了するまでに数分かかる場合があります。*

## インデックスを検索する

インデックスが作成されたので、検索できます。

1. Azure AI Search リソースのページに移動し、**インデクス** ページを選択して 作成されたインデクスを選択します。
2. Search explorer で、**クエリ文字列** ボックスに次のクエリ文字列を入力し、**検索** を選択します。

    ```
    search=London&$select=url,sentiment,keyphrases&$filter=metadata_author eq 'Reviewer' and sentiment eq 'positive'
    ```

    このクエリは、*London* を言及し、*Reviewer* によって作成され、ポジティブな **sentiment** ラベルを持つすべてのドキュメントの **url**、**sentiment**、および **keyphrases** を取得します (つまり、London を言及するポジティブなレビュー)。

## カスタムスキルのための Azure 関数を作成する

検索ソリューションには、ドキュメントからの情報でインデックスを強化する多くの組み込み AI スキルが含まれています。これには、前のタスクで見た感情スコアやキーフレーズのリストが含まれます。

インデックスをさらに強化するために、カスタムスキルを作成できます。たとえば、各ドキュメントで最も頻繁に使用される単語を特定することが有用ですが、この機能を提供する組み込みスキルはありません。

単語数機能をカスタムスキルとして実装するには、好みの言語で Azure 関数を作成します。

> **注**: この演習では、Azure ポータルのコード編集機能を使用して簡単な Node.JS 関数を作成します。実際のソリューションでは、通常、Visual Studio Code などの開発環境を使用して、好みの言語 (たとえば C#、Python、Node.JS、または Java) で関数アプリを作成し、DevOps プロセスの一環として Azure に公開します。

1. Azure ポータルの 上部の検索バーで Function App を検索し、次の設定で新しい **Function App** リソースを作成します：
    - **ホスティングプラン**: Consumption
    - **サブスクリプション**: *あなたのサブスクリプション*
    - **リソースグループ**: *Azure AI Search リソースと同じリソースグループ*
    - **Function App 名**: *一意の名前*
    - **ランタイムスタック**: Node.js
    - **バージョン**: 18 LTS
    - **リージョン**: *Azure AI Search リソースと同じリージョン*
    - **オペレーティングシステム**: Windows

2. デプロイが完了するのを待ち、デプロイされた Function App リソースに移動します。
3. **概要** ページで、ページの下部にある **関数の作成** を選択し、次の設定で新しい関数を作成します：
    - **テンプレートを選択**
        - **テンプレート**: HTTP Trigger    
    - **テンプレートの詳細**：
        - **関数名**: wordcount
        - **承認レベル**: Function

    > **注**: 関数の作成エラーが発生した場合は、ページを更新してください。リソースが正常に作成されているはずです。

4. *wordcount* 関数が作成されるのを待ちます。次に、そのページで **コード + テスト** タブを選択します。
5. デフォルトの関数コードを次のコードに置き換えます：

```javascript
module.exports = async function (context, req) {
    context.log('JavaScript HTTP trigger function processed a request.');

    if (req.body && req.body.values) {

        vals = req.body.values;

        // Array of stop words to be ignored
        var stopwords = ['', 'i', 'me', 'my', 'myself', 'we', 'our', 'ours', 'ourselves', 'you', 
        "youre", "youve", "youll", "youd", 'your', 'yours', 'yourself', 
        'yourselves', 'he', 'him', 'his', 'himself', 'she', "shes", 'her', 
        'hers', 'herself', 'it', "its", 'itself', 'they', 'them', 
        'their', 'theirs', 'themselves', 'what', 'which', 'who', 'whom', 
        'this', 'that', "thatll", 'these', 'those', 'am', 'is', 'are', 'was',
        'were', 'be', 'been', 'being', 'have', 'has', 'had', 'having', 'do', 
        'does', 'did', 'doing', 'a', 'an', 'the', 'and', 'but', 'if', 'or', 
        'because', 'as', 'until', 'while', 'of', 'at', 'by', 'for', 'with', 
        'about', 'against', 'between', 'into', 'through', 'during', 'before', 
        'after', 'above', 'below', 'to', 'from', 'up', 'down', 'in', 'out', 
        'on', 'off', 'over', 'under', 'again', 'further', 'then', 'once', 'here', 
        'there', 'when', 'where', 'why', 'how', 'all', 'any', 'both', 'each', 
        'few', 'more', 'most', 'other', 'some', 'such', 'no', 'nor', 'not', 
        'only', 'own', 'same', 'so', 'than', 'too', 'very', 'can', 'will',
        'just', "dont", 'should', "shouldve", 'now', "arent", "couldnt", 
        "didnt", "doesnt", "hadnt", "hasnt", "havent", "isnt", "mightnt", "mustnt",
        "neednt", "shant", "shouldnt", "wasnt", "werent", "wont", "wouldnt"];

        res = {"values":[]};

        for (rec in vals)
        {
            // Get the record ID and text for this input
            resVal = {recordId:vals[rec].recordId, data:{}};
            txt = vals[rec].data.text;

            // remove punctuation and numerals
            txt = txt.replace(/[^ A-Za-z_]/g,"").toLowerCase();

            // Get an array of words
            words = txt.split(" ")

            // count instances of non-stopwords
            wordCounts = {}
            for(var i = 0; i < words.length; ++i) {
                word = words[i];
                if (stopwords.includes(word) == false )
                {
                    if (wordCounts[word])
                    {
                        wordCounts[word] ++;
                    }
                    else
                    {
                        wordCounts[word] = 1;
                    }
                }
            }

            // Convert wordcounts to an array
            var topWords = [];
            for (var word in wordCounts) {
                topWords.push([word, wordCounts[word]]);
            }

            // Sort in descending order of count
            topWords.sort(function(a, b) {
                return b[1] - a[1];
            });

            // Get the first ten words from the first array dimension
            resVal.data.text = topWords.slice(0,9)
              .map(function(value,index) { return value[0]; });

            res.values[rec] = resVal;
        };

        context.res = {
            body: JSON.stringify(res),
            headers: {
            'Content-Type': 'application/json'
        }

        };
    }
    else {
        context.res = {
            status: 400,
            body: {"errors":[{"message": "Invalid input"}]},
            headers: {
            'Content-Type': 'application/json'
        }

        };
    }
};
```

6. 関数を保存し、**Test/Run** ペインを開きます。
7. **Test/Run** ペインで、既存の **Body** を次の JSON に置き換えます。これは、1 つ以上のドキュメントのデータを含むレコードが処理のために送信される Azure AI Search スキルによって期待されるスキーマを反映しています：

    ```json
    {
        "values": [
            {
                "recordId": "a1",
                "data":
                {
                "text":  "Tiger, tiger burning bright in the darkness of the night.",
                "language": "en"
                }
            },
            {
                "recordId": "a2",
                "data":
                {
                "text":  "The rain in spain stays mainly in the plains! That's where you'll find the rain!",
                "language": "en"
                }
            }
        ]
    }
    ```

8. **Run** をクリックし、関数によって返される HTTP 応答コンテンツを表示します。これは、スキルを消費する際に Azure AI Search によって期待されるスキーマを反映しています。この場合、応答は各ドキュメント内の最大 10 個の用語で構成され、出現頻度の降順で表示されます：

    ```json
    {
        "values": [
        {
            "recordId": "a1",
            "data": {
                "text": [
                "tiger",
                "burning",
                "bright",
                "darkness",
                "night"
                ]
            }
        },
        {
            "recordId": "a2",
            "data": {
                "text": [
                    "rain",
                    "spain",
                    "stays",
                    "mainly",
                    "plains",
                    "thats",
                    "youll",
                    "find"
                ]
            }
        }
        ]
    }
    ```

9. **Test/Run** ペインを閉じ、**wordcount** 関数ブレードで **Get function URL** をクリックします。次に、デフォルトキーの URL をクリップボードにコピーします。次の手順でこれが必要になります。

## カスタムスキルを検索ソリューションに追加する

次に、関数を検索ソリューションのスキルセットにカスタムスキルとして含め、それが生成する結果をインデックスのフィールドにマッピングする必要があります。

1. Visual Studio Code で **02-search-skill/update-search** フォルダーを開き、**update-skillset.json** ファイルを開きます。これにはスキルセットの JSON 定義が含まれています。
2. スキルセットの定義を確認します。以前と同じスキルに加えて、新しい **WebApiSkill** スキル **get-top-words** が含まれています。
3. **get-top-words** スキルの定義を編集し、**uri** 値を Azure 関数の URL に設定します (前の手順でクリップボードにコピーしたもの)。**YOUR-FUNCTION-APP-URL** を置き換えます。
4. スキルセット定義の上部にある **cognitiveServices** 要素で、**YOUR_AI_SERVICES_KEY** プレースホルダーを Azure AI Services リソースのいずれかのキーに置き換えます。

    *キーは Azure ポータルの **キーとエンドポイント** ページで確認できます。*

5. 更新された JSON ファイルを保存して閉じます。
6. **update-search** フォルダーで **update-index.json** を開きます。このファイルには **margies-custom-index** インデックスの JSON 定義が含まれており、インデックス定義の下部に **top_words** という追加フィールドがあります。
7. インデックスの JSON を確認し、変更を加えずにファイルを閉じます。
8. **update-search** フォルダーで **update-indexer.json** を開きます。このファイルには **margies-custom-indexer** の JSON 定義が含まれており、**top_words** フィールドの追加マッピングがあります。
9. インデクサーの JSON を確認し、変更を加えずにファイルを閉じます。
10. **update-search** フォルダーで **update-search.cmd** を開きます。このバッチスクリプトは、cURL ユーティリティを使用して更新された JSON 定義を Azure AI Search リソースの REST インターフェイスに送信します。
11. **YOUR_SEARCH_URL** および **YOUR_ADMIN_KEY** 変数プレースホルダーを、Azure AI Search リソースの **Url** および管理キーのいずれかに置き換えます。

    *これらの値は Azure ポータルの **概要** および **キー** ページで確認できます。*

12. 更新されたバッチファイルを保存します。
13. **update-search** フォルダーを右クリックし、**統合ターミナルで開く** を選択します。
14. **update-search** フォルダーのターミナルペインで、次のコマンドを入力してバッチスクリプトを実行します。

    ```powershell
    ./update-search
    ```

15. スクリプトが完了したら、Azure ポータルで Azure AI Search リソースのページに移動し、**インデクサー** ページを選択してインデックス作成プロセスが完了するのを待ちます。

    *インデックス作成操作の進行状況を追跡するには、**更新** を選択できます。完了するまでに数分かかる場合があります。*

## インデックスを検索する

インデックスが作成されたので、検索できます。

1. Azure AI Search リソースのブレードの上部で **Search explorer** を選択します。
2. Search explorer で、ビューを **JSON ビュー** に変更し、次の検索クエリを送信します：

    ```json
    {
      "search": "Las Vegas",
      "select": "url,top_words"
    }
    ```

    このクエリは、*Las Vegas* を言及するすべてのドキュメントの **url** および **top_words** フィールドを取得します。

## クリーンアップ

演習が完了したので、不要になったすべてのリソースを削除します。Azure リソースを削除します：

1. Azure ポータルで **リソースグループ** を選択します。
1. 不要なリソースグループを選択し、**リソースグループの削除** を選択します。

## 詳細情報

Azure AI Search のカスタムスキルの作成について詳しくは、[Azure AI Search ドキュメント](https://docs.microsoft.com/azure/search/cognitive-search-custom-skill-interface) を参照してください。
