---
lab:
    title: '画像内のテキストを読み取る'
    module: 'モジュール 11 - 画像およびドキュメント内のテキストを読み取る'
---

# 画像内のテキストを読み取る

光学文字認識 (OCR) は、画像やドキュメント内のテキストを読み取るコンピュータ ビジョンの一部です。**Azure AI Vision** サービスは、テキストを読み取るための API を提供しており、この演習でそれを探索します。

## このコースのリポジトリをクローンする

まだ行っていない場合は、このコースのコード リポジトリをクローンする必要があります。

1. Visual Studio Code を起動します。
2. パレット (SHIFT+CTRL+P) を開き、**Git: Clone** コマンドを実行して `https://github.com/MicrosoftLearning/mslearn-ai-vision` リポジトリをローカル フォルダーにクローンします (フォルダーはどこでも構いません)。
3. リポジトリがクローンされたら、Visual Studio Code でフォルダーを開きます。
4. リポジトリ内の C# コード プロジェクトをサポートするための追加ファイルがインストールされるのを待ちます。
 > **注**: ビルドおよびデバッグに必要なアセットを追加するように求められた場合は、**Not Now** を選択します。*Detected an Azure Function Project in folder* というメッセージが表示された場合は、そのメッセージを閉じても問題ありません。

## Azure AI サービス リソースをプロビジョニングする

サブスクリプションにまだリソースがない場合は、**Azure AI サービス** リソースをプロビジョニングする必要があります。

1. `https://portal.azure.com` で Azure ポータルを開き、Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。
2. 上部の検索バーで *Azure AI services* を検索し、**Azure AI Services** を選択して、次の設定で Azure AI サービス マルチサービス アカウント リソースを作成します。
    - **サブスクリプション**: *Azure サブスクリプション*
    - **リソース グループ**: *リソース グループを選択または作成 (制限付きサブスクリプションを使用している場合は、新しいリソース グループを作成する権限がない場合があります - 提供されたものを使用します)*
    - **リージョン**: *East US、France Central、Korea Central、North Europe、Southeast Asia、West Europe、West US、または East Asia から選択\**
    - **名前**: *一意の名前を入力*
    - **価格レベル**: Standard S0

    \*Azure AI Vision 4.0 の機能は現在これらのリージョンでのみ利用可能です。

3. 必要なチェックボックスを選択してリソースを作成します。
4. デプロイメントが完了するのを待ち、デプロイメントの詳細を表示します。
5. リソースがデプロイされたら、その **キーとエンドポイント** ページを表示します。次の手順でエンドポイントとキーのいずれかが必要になります。

## Azure AI Vision SDK を使用する準備をする

この演習では、Azure AI Vision SDK を使用してテキストを読み取る部分的に実装されたクライアント アプリケーションを完成させます。

> **注**: **C#** または **Python** のいずれかの SDK を使用することができます。以下の手順では、希望する言語に適した操作を行います。

1. Visual Studio Code の **Explorer** ペインで、**Labfiles\\05-ocr** フォルダーに移動し、希望する言語に応じて **C-Sharp** または **Python** フォルダーを展開します。
2. **read-text** フォルダーを右クリックして統合ターミナルを開き、希望する言語に応じて次のコマンドを実行して Azure AI Vision SDK パッケージをインストールします:

    **C#**
    
    ```
    dotnet add package Azure.AI.Vision.ImageAnalysis -v 1.0.0-beta.3
    ```

    > **Note**: If you are prompted to install dev kit extensions, you can safely close the message.

    **Python**
    
    ```
    pip install azure-ai-vision-imageanalysis==1.0.0b3
    ```

3. **read-text** フォルダーの内容を表示し、設定ファイルが含まれていることを確認します:
    - **C#**: appsettings.json
    - **Python**: .env
    設定ファイルを開き、Azure AI サービス リソースの **エンドポイント** と認証 **キー** を反映するように設定値を更新します。変更を保存します。

## Azure AI Vision SDK を使用して画像からテキストを読み取る

**Azure AI Vision SDK** の機能の 1 つは、画像からテキストを読み取ることです。この演習では、Azure AI Vision SDK を使用して画像からテキストを読み取る部分的に実装されたクライアント アプリケーションを完成させます。

1. **read-text** フォルダーにはクライアント アプリケーションのコード ファイルが含まれています:

    - **C#**: Program.cs
    - **Python**: read-text.py

    コード ファイルを開き、既存の名前空間参照の下にある **Import namespaces** コメントの下に、Azure AI Vision SDK を使用するために必要な名前空間をインポートするための次の言語固有のコードを追加します:

    **C#**
    
    ```C#
    // Import namespaces
    using Azure.AI.Vision.ImageAnalysis;
    ```
    
    **Python**
    
    ```Python
    # import namespaces
    from azure.ai.vision.imageanalysis import ImageAnalysisClient
    from azure.ai.vision.imageanalysis.models import VisualFeatures
    from azure.core.credentials import AzureKeyCredential
    ```

2. クライアント アプリケーションのコード ファイルで、**Main** 関数に設定を読み込むコードが提供されています。次に、**Authenticate Azure AI Vision client** コメントの下に、Azure AI Vision クライアント オブジェクトを作成して認証するための次の言語固有のコードを追加します:

    **C#**
    
    ```C#
    // Authenticate Azure AI Vision client
    ImageAnalysisClient client = new ImageAnalysisClient(
        new Uri(aiSvcEndpoint),
        new AzureKeyCredential(aiSvcKey));
    ```
    
    **Python**
    
    ```Python
    # Authenticate Azure AI Vision client
    cv_client = ImageAnalysisClient(
        endpoint=ai_endpoint,
        credential=AzureKeyCredential(ai_key)
    )
    ```

3. **Main** 関数で、追加したコードの下に、画像ファイルのパスを指定し、その画像パスを **GetTextRead** 関数に渡すコードがあることを確認します。この関数はまだ完全には実装されていません。

4. **GetTextRead** 関数の本文にコードを追加します。**Use Analyze image function to read text in image** コメントを見つけ、その下に次の言語固有のコードを追加します。`Analyze` 関数を呼び出す際に視覚的な機能が指定されていることに注意してください:

    **C#**

    ```C#
    // Use Analyze image function to read text in image
    ImageAnalysisResult result = client.Analyze(
        BinaryData.FromStream(stream),
        // Specify the features to be retrieved
        VisualFeatures.Read);
    
    stream.Close();
    
    // Display analysis results
    if (result.Read != null)
    {
        Console.WriteLine($"Text:");
    
        // Prepare image for drawing
        System.Drawing.Image image = System.Drawing.Image.FromFile(imageFile);
        Graphics graphics = Graphics.FromImage(image);
        Pen pen = new Pen(Color.Cyan, 3);
        
        foreach (var line in result.Read.Blocks.SelectMany(block => block.Lines))
        {
            // Return the text detected in the image
    
    
        }
            
        // Save image
        String output_file = "text.jpg";
        image.Save(output_file);
        Console.WriteLine("\nResults saved in " + output_file + "\n");   
    }
    ```
    
    **Python**
    
    ```Python
    # Use Analyze image function to read text in image
    result = cv_client.analyze(
        image_data=image_data,
        visual_features=[VisualFeatures.READ]
    )

    # Display the image and overlay it with the extracted text
    if result.read is not None:
        print("\nText:")

        # Prepare image for drawing
        image = Image.open(image_file)
        fig = plt.figure(figsize=(image.width/100, image.height/100))
        plt.axis('off')
        draw = ImageDraw.Draw(image)
        color = 'cyan'

        for line in result.read.blocks[0].lines:
            # Return the text detected in the image

            
        # Save image
        plt.imshow(image)
        plt.tight_layout(pad=0)
        outputfile = 'text.jpg'
        fig.savefig(outputfile)
        print('\n  Results saved in', outputfile)
    ```

5. **GetTextRead** 関数に追加したコードで、**Return the text detected in the image** コメントの下に次のコードを追加します (このコードは、画像のテキストをコンソールに出力し、画像のテキストを強調表示する **text.jpg** 画像を生成します):

    **C#**
    
    ```C#
    // Return the text detected in the image
    Console.WriteLine($"   '{line.Text}'");
    
    // Draw bounding box around line
    var drawLinePolygon = true;
    
    // Return the position bounding box around each line
    
    
    
    // Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    
    
    
    // Draw line bounding polygon
    if (drawLinePolygon)
    {
        var r = line.BoundingPolygon;
    
        Point[] polygonPoints = {
            new Point(r[0].X, r[0].Y),
            new Point(r[1].X, r[1].Y),
            new Point(r[2].X, r[2].Y),
            new Point(r[3].X, r[3].Y)
        };
    
        graphics.DrawPolygon(pen, polygonPoints);
    }
    ```
    
    **Python**
    
    ```Python
    # Return the text detected in the image
    print(f"  {line.text}")    
    
    drawLinePolygon = True
    
    r = line.bounding_polygon
    bounding_polygon = ((r[0].x, r[0].y),(r[1].x, r[1].y),(r[2].x, r[2].y),(r[3].x, r[3].y))
    
    # Return the position bounding box around each line
    
    
    # Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    
    
    # Draw line bounding polygon
    if drawLinePolygon:
        draw.polygon(bounding_polygon, outline=color, width=3)
    ```

6. **read-text/images** フォルダーで **Lincoln.jpg** を選択して、コードが処理するファイルを表示します。
7. アプリケーションのコード ファイルで、**Main** 関数内のユーザーがメニュー オプション **1** を選択した場合に実行されるコードを確認します。このコードは **GetTextRead** 関数を呼び出し、*Lincoln.jpg* 画像ファイルのパスを渡します。
8. 変更を保存し、**read-text** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

9. プロンプトが表示されたら **1** を入力し、画像から抽出されたテキストを確認します。
10. **read-text** フォルダーで **text.jpg** 画像を選択し、各テキストの *行* の周りにポリゴンがあることを確認します。
11. Visual Studio Code のコード ファイルに戻り、**Return the position bounding box around each line** コメントを見つけます。その下に次のコードを追加します:

    **C#**
    
    ```C#
    // Return the position bounding box around each line
    Console.WriteLine($"   Bounding Polygon: [{string.Join(" ", line.BoundingPolygon)}]");  
    ```
    
    **Python**
    
    ```Python
    # Return the position bounding box around each line
    print("   Bounding Polygon: {}".format(bounding_polygon))
    ```

12. 変更を保存し、**read-text** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

13. プロンプトが表示されたら **1** を入力し、画像内の各テキスト行とその位置を確認します。
14. Visual Studio Code のコード ファイルに戻り、**Return each word detected in the image and the position bounding box around each word with the confidence level of each word** コメントを見つけます。その下に次のコードを追加します:

    **C#**
    
    ```C#
    // Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    foreach (DetectedTextWord word in line.Words)
    {
        Console.WriteLine($"     Word: '{word.Text}', Confidence {word.Confidence:F4}, Bounding Polygon: [{string.Join(" ", word.BoundingPolygon)}]");
        
        // Draw word bounding polygon
        drawLinePolygon = false;
        var r = word.BoundingPolygon;
    
        Point[] polygonPoints = {
            new Point(r[0].X, r[0].Y),
            new Point(r[1].X, r[1].Y),
            new Point(r[2].X, r[2].Y),
            new Point(r[3].X, r[3].Y)
        };
    
        graphics.DrawPolygon(pen, polygonPoints);
    }
    ```
    
    **Python**
    
    ```Python
    # Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    for word in line.words:
        r = word.bounding_polygon
        bounding_polygon = ((r[0].x, r[0].y),(r[1].x, r[1].y),(r[2].x, r[2].y),(r[3].x, r[3].y))
        print(f"    Word: '{word.text}', Bounding Polygon: {bounding_polygon}, Confidence: {word.confidence:.4f}")
    
        # Draw word bounding polygon
        drawLinePolygon = False
        draw.polygon(bounding_polygon, outline=color, width=3)
    ```

15. 変更を保存し、**read-text** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

16. プロンプトが表示されたら **1** を入力し、画像内の各テキスト行とその位置を確認します。各単語の信頼度も返されることに注意してください。
17. **read-text** フォルダーで **text.jpg** 画像を選択し、各 *単語* の周りにポリゴンがあることを確認します。

## Azure AI Vision SDK を使用して手書きのテキストを読み取る

前の演習では、画像から明確に定義されたテキストを読み取りましたが、手書きのメモや書類からテキストを読み取る必要がある場合もあります。幸いなことに、**Azure AI Vision SDK** は、明確に定義されたテキストを読み取るために使用したのと同じコードで手書きのテキストも読み取ることができます。前の演習と同じコードを使用しますが、今回は別の画像を使用します。

1. **read-text/images** フォルダーで **Note.jpg** を選択して、コードが処理するファイルを表示します。
2. アプリケーションのコード ファイルで、**Main** 関数内のユーザーがメニュー オプション **2** を選択した場合に実行されるコードを確認します。このコードは **GetTextRead** 関数を呼び出し、*Note.jpg* 画像ファイルのパスを渡します。
3. **read-text** フォルダーの統合ターミナルから、次のコマンドを入力してプログラムを実行します:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

4. プロンプトが表示されたら **2** を入力し、メモ画像から抽出されたテキストを確認します。
5. **read-text** フォルダーで **text.jpg** 画像を選択し、メモの各 *単語* の周りにポリゴンがあることを確認します。

## リソースのクリーンアップ

このラボで作成した Azure リソースを他のトレーニング モジュールに使用しない場合は、追加の料金が発生しないように削除できます。以下の手順に従ってください。

1. `https://portal.azure.com` で Azure ポータルを開き、Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。
2. 上部の検索バーで *Azure AI services multi-service account* を検索し、このラボで作成した Azure AI サービス マルチサービス アカウント リソースを選択します。
3. リソース ページで **削除** を選択し、リソースを削除する手順に従います。

## 詳細情報

**Azure AI Vision** サービスを使用してテキストを読み取る方法の詳細については、[Azure AI Vision ドキュメント](https://learn.microsoft.com/azure/ai-services/computer-vision/concept-ocr)を参照してください。
