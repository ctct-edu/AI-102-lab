---
lab:
  title: 'Azure AI Vision を使用して画像を分析する'
  module: 'モジュール 2 - Azure AI Vision を使用したComputer Vision Solutionの開発'
---

# Azure AI Vision を使用して画像を分析する

Azure AI Vision は、画像を分析することで視覚入力を解釈する人工知能機能です。Microsoft Azure では、**Vision** Azure AI サービスが一般的なコンピュータ ビジョン タスク用の事前構築モデルを提供しており、画像のキャプションやタグの提案、一般的なオブジェクトや人物の検出などが含まれます。

## このコースのリポジトリをクローンする

まだ **Azure AI Vision** コード リポジトリを作業環境にクローンしていない場合は、次の手順に従ってクローンします。すでにクローンしている場合は、Visual Studio Code でクローンしたフォルダーを開きます。

1. Visual Studio Code を起動します。
2. パレット (SHIFT+CTRL+P) を開き、**Git: Clone** コマンドを実行して `https://github.com/MicrosoftLearning/mslearn-ai-vision` リポジトリをローカル フォルダーにクローンします (フォルダーはどこでも構いません)。
3. リポジトリがクローンされたら、Visual Studio Code でフォルダーを開きます。
4. リポジトリ内の C# コード プロジェクトをサポートするための追加ファイルがインストールされるのを待ちます。
 > **注**: ビルドおよびデバッグに必要なアセットを追加するように求められた場合は、**Not Now** を選択します。*Detected an Azure Function Project in folder* というメッセージが表示された場合は、そのメッセージを閉じても問題ありません。

## Azure AI サービス リソースをプロビジョニングする

サブスクリプションにまだリソースがない場合は、**Azure AI サービス** リソースをプロビジョニングする必要があります。

1. `https://portal.azure.com` で Azure ポータルを開き、Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。
2. **リソースの作成** を選択します。
3. 検索バーで *Azure AI services* を検索し、**Azure AI Services** を選択して、次の設定で Azure AI サービス マルチサービス アカウント リソースを作成します。
    - **サブスクリプション**: *Azure サブスクリプション*
    - **リソース グループ**: *リソース グループを選択または作成 (制限付きサブスクリプションを使用している場合は、新しいリソース グループを作成する権限がない場合があります - 提供されたものを使用します)*
    - **リージョン**: *East US、France Central、Korea Central、North Europe、Southeast Asia、West Europe、West US、または East Asia から選択\**
    - **名前**: *一意の名前を入力*
    - **価格レベル**: Standard S0

    \*Azure AI Vision 4.0 の機能は現在これらのリージョンでのみ利用可能です。

4. 必要なチェックボックスを選択してリソースを作成します。
5. デプロイメントが完了するのを待ち、デプロイメントの詳細を表示します。
6. リソースがデプロイされたら、その **キーとエンドポイント** ページを表示します。次の手順でエンドポイントとキーのいずれかが必要になります。

## Azure AI Vision SDK を使用する準備をする

この演習では、Azure AI Vision SDK を使用して画像を分析する部分的に実装されたクライアント アプリケーションを完成させます。

> **注**: **C#** または **Python** のいずれかの SDK を使用することができます。以下の手順では、希望する言語に適した操作を行います。

1. Visual Studio Code の **Explorer** ペインで、**Labfiles/01-analyze-images** フォルダーに移動し、希望する言語に応じて **C-Sharp** または **Python** フォルダーを展開します。
2. **image-analysis** フォルダーを右クリックして統合ターミナルを開き、希望する言語に応じて次のコマンドを実行して Azure AI Vision SDK パッケージをインストールします:

    **C#**

    ```
    dotnet add package Azure.AI.Vision.ImageAnalysis -v 1.0.0-beta.3
    ```

    > **注**: 開発キット拡張機能のインストールを求められた場合は、そのメッセージを閉じても問題ありません。

     **Python**
    
    ```
    pip install azure-ai-vision-imageanalysis==1.0.0b3
    ```
   
    > **ヒント**: このラボを自分のマシンで行っている場合は、`matplotlib` と `pillow` もインストールする必要があります。

3. **image-analysis** フォルダーの内容を表示し、設定ファイルが含まれていることを確認します:
    - **C#**: appsettings.json
    - **Python**: .env
    設定ファイルを開き、Azure AI サービス リソースの **エンドポイント** と認証 **キー** を反映するように設定値を更新します。変更を保存します。

4. **image-analysis** フォルダーにはクライアント アプリケーションのコード ファイルが含まれていることを確認します:
    - **C#**: Program.cs
    - **Python**: image-analysis.py
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

## 分析する画像を表示する

この演習では、Azure AI Vision サービスを使用して複数の画像を分析します。

1. Visual Studio Code で **image-analysis** フォルダーとその中にある **images** フォルダーを展開します。
2. 各画像ファイルを順に選択して、Visual Studio Code で表示します。

## 画像を分析してキャプションを提案する

SDK を使用して Vision サービスを呼び出し、画像を分析する準備が整いました。

1. クライアント アプリケーションのコード ファイル (**Program.cs** または **image-analysis.py**) の **Main** 関数で、設定を読み込むコードが提供されていることを確認します。次に、**Authenticate Azure AI Vision client** コメントの下に、Azure AI Vision クライアント オブジェクトを作成して認証するための次の言語固有のコードを追加します:

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

2. **Main** 関数で、追加したコードの下に、画像ファイルのパスを指定し、その画像パスを他の関数 (**AnalyzeImage**) に渡すコードがあることを確認します。これらの関数はまだ完全には実装されていません。

3. **AnalyzeImage** 関数で、**Get result with specify features to be retrieved** コメントの下に、次のコードを追加します:

**C#**

```C#
// Get result with specified features to be retrieved
ImageAnalysisResult result = client.Analyze(
    BinaryData.FromStream(stream),
    VisualFeatures.Caption | 
    VisualFeatures.DenseCaptions |
    VisualFeatures.Objects |
    VisualFeatures.Tags |
    VisualFeatures.People);
```

**Python**

```Python
# Get result with specified features to be retrieved
result = cv_client.analyze(
    image_data=image_data,
    visual_features=[
        VisualFeatures.CAPTION,
        VisualFeatures.DENSE_CAPTIONS,
        VisualFeatures.TAGS,
        VisualFeatures.OBJECTS,
        VisualFeatures.PEOPLE],
)
```
    
4. **AnalyzeImage** 関数で、**Display analysis results** コメントの下に、次のコードを追加します (後で追加するコードを示すコメントも含みます):

**C#**

```C#
// Display analysis results
// Get image captions
if (result.Caption.Text != null)
{
    Console.WriteLine(" Caption:");
    Console.WriteLine($"   \"{result.Caption.Text}\", Confidence {result.Caption.Confidence:0.00}\n");
}

// Get image dense captions
Console.WriteLine(" Dense Captions:");
foreach (DenseCaption denseCaption in result.DenseCaptions.Values)
{
    Console.WriteLine($"   Caption: '{denseCaption.Text}', Confidence: {denseCaption.Confidence:0.00}");
}

// Get image tags


// Get objects in the image


// Get people in the image
```

**Python**

```Python
# Display analysis results
# Get image captions
if result.caption is not None:
    print("\nCaption:")
    print(" Caption: '{}' (confidence: {:.2f}%)".format(result.caption.text, result.caption.confidence * 100))

# Get image dense captions
if result.dense_captions is not None:
    print("\nDense Captions:")
    for caption in result.dense_captions.list:
        print(" Caption: '{}' (confidence: {:.2f}%)".format(caption.text, caption.confidence * 100))

# Get image tags


# Get objects in the image


# Get people in the image

```

5. 変更を保存し、**image-analysis** フォルダーの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します (引数として **images/street.jpg** を指定します):

**C#**

```
dotnet run images/street.jpg
```

**Python**

```
python image-analysis.py images/street.jpg
```

6. 出力を確認し、**street.jpg** 画像に対する提案されたキャプションが含まれていることを確認します。
7. プログラムを再度実行し、今度は引数として **images/building.jpg** を指定して、**building.jpg** 画像に対して生成されたキャプションを確認します。
8. 前のステップを繰り返して、**images/person.jpg** ファイルのキャプションを生成します。

## 画像に対する提案されたタグを取得する

画像の内容に関する手がかりを提供する関連する *タグ* を識別することが有用な場合があります。

1. **AnalyzeImage** 関数で、**Get image tags** コメントの下に次のコードを追加します:

**C#**

```C#
// Get image tags
if (result.Tags.Values.Count > 0)
{
    Console.WriteLine($"\n Tags:");
    foreach (DetectedTag tag in result.Tags.Values)
    {
        Console.WriteLine($"   '{tag.Name}', Confidence: {tag.Confidence:F2}");
    }
}
```

**Python**

```Python
# Get image tags
if result.tags is not None:
    print("\nTags:")
    for tag in result.tags.list:
        print(" Tag: '{}' (confidence: {:.2f}%)".format(tag.name, tag.confidence * 100))
```

2. 変更を保存し、**images** フォルダー内の各画像ファイルに対してプログラムを実行し、画像キャプションに加えて提案されたタグのリストが表示されることを確認します。

## 画像内のオブジェクトを検出して位置を特定する

*オブジェクト検出* は、画像内の個々のオブジェクトを識別し、その位置をバウンディング ボックスで示す特定の形式のコンピュータ ビジョンです。

1. **AnalyzeImage** 関数で、**Get objects in the image** コメントの下に次のコードを追加します:

**C#**

```C#
// Get objects in the image
if (result.Objects.Values.Count > 0)
{
    Console.WriteLine(" Objects:");

    // Prepare image for drawing
    stream.Close();
    System.Drawing.Image image = System.Drawing.Image.FromFile(imageFile);
    Graphics graphics = Graphics.FromImage(image);
    Pen pen = new Pen(Color.Cyan, 3);
    Font font = new Font("Arial", 16);
    SolidBrush brush = new SolidBrush(Color.WhiteSmoke);

    foreach (DetectedObject detectedObject in result.Objects.Values)
    {
        Console.WriteLine($"   \"{detectedObject.Tags[0].Name}\"");

        // Draw object bounding box
        var r = detectedObject.BoundingBox;
        Rectangle rect = new Rectangle(r.X, r.Y, r.Width, r.Height);
        graphics.DrawRectangle(pen, rect);
        graphics.DrawString(detectedObject.Tags[0].Name,font,brush,(float)r.X, (float)r.Y);
    }

    // Save annotated image
    String output_file = "objects.jpg";
    image.Save(output_file);
    Console.WriteLine("  Results saved in " + output_file + "\n");
}
```

**Python**

```Python
# Get objects in the image
if result.objects is not None:
    print("\nObjects in image:")

    # Prepare image for drawing
    image = Image.open(image_filename)
    fig = plt.figure(figsize=(image.width/100, image.height/100))
    plt.axis('off')
    draw = ImageDraw.Draw(image)
    color = 'cyan'

    for detected_object in result.objects.list:
        # Print object name
        print(" {} (confidence: {:.2f}%)".format(detected_object.tags[0].name, detected_object.tags[0].confidence * 100))
        
        # Draw object bounding box
        r = detected_object.bounding_box
        bounding_box = ((r.x, r.y), (r.x + r.width, r.y + r.height)) 
        draw.rectangle(bounding_box, outline=color, width=3)
        plt.annotate(detected_object.tags[0].name,(r.x, r.y), backgroundcolor=color)

    # Save annotated image
    plt.imshow(image)
    plt.tight_layout(pad=0)
    outputfile = 'objects.jpg'
    fig.savefig(outputfile)
    print('  Results saved in', outputfile)
```

2. 変更を保存し、**images** フォルダー内の各画像ファイルに対してプログラムを実行し、検出されたオブジェクトを確認します。各実行後、コード ファイルと同じフォルダーに生成される **objects.jpg** ファイルを表示して、注釈付きオブジェクトを確認します。

## 画像内の人物を検出して位置を特定する

*人物検出* は、画像内の個々の人物を識別し、その位置をバウンディング ボックスで示す特定の形式のコンピュータ ビジョンです。

1. **AnalyzeImage** 関数で、**Get people in the image** コメントの下に次のコードを追加します:

**C#**

```C#
// Get people in the image
if (result.People.Values.Count > 0)
{
    Console.WriteLine($" People:");

    // Prepare image for drawing
    System.Drawing.Image image = System.Drawing.Image.FromFile(imageFile);
    Graphics graphics = Graphics.FromImage(image);
    Pen pen = new Pen(Color.Cyan, 3);
    Font font = new Font("Arial", 16);
    SolidBrush brush = new SolidBrush(Color.WhiteSmoke);

    foreach (DetectedPerson person in result.People.Values)
    {
        // Draw object bounding box
        var r = person.BoundingBox;
        Rectangle rect = new Rectangle(r.X, r.Y, r.Width, r.Height);
        graphics.DrawRectangle(pen, rect);
        
        // Return the confidence of the person detected
        //Console.WriteLine($"   Bounding box {person.BoundingBox.ToString()}, Confidence: {person.Confidence:F2}");
    }

    // Save annotated image
    String output_file = "persons.jpg";
    image.Save(output_file);
    Console.WriteLine("  Results saved in " + output_file + "\n");
}
```

**Python**

```Python
# Get people in the image
if result.people is not None:
    print("\nPeople in image:")

    # Prepare image for drawing
    image = Image.open(image_filename)
    fig = plt.figure(figsize=(image.width/100, image.height/100))
    plt.axis('off')
    draw = ImageDraw.Draw(image)
    color = 'cyan'

    for detected_people in result.people.list:
        # Draw object bounding box
        r = detected_people.bounding_box
        bounding_box = ((r.x, r.y), (r.x + r.width, r.y + r.height))
        draw.rectangle(bounding_box, outline=color, width=3)

        # Return the confidence of the person detected
        #print(" {} (confidence: {:.2f}%)".format(detected_people.bounding_box, detected_people.confidence * 100))
        
    # Save annotated image
    plt.imshow(image)
    plt.tight_layout(pad=0)
    outputfile = 'people.jpg'
    fig.savefig(outputfile)
    print('  Results saved in', outputfile)
```

2. (オプション) **Return the confidence of the person detected** セクションの **Console.Writeline** コマンドのコメントを解除して、特定の位置で人物が検出された信頼度を確認します。
3. 変更を保存し、**images** フォルダー内の各画像ファイルに対してプログラムを実行し、検出された人物を確認します。各実行後、コード ファイルと同じフォルダーに生成される **people.jpg** ファイルを表示して、注釈付きの人物を確認します。

> **注**: 前述のタスクでは、単一のメソッドを使用して画像を分析し、結果を解析して表示するコードを段階的に追加しました。SDK には、キャプションの提案、タグの識別、オブジェクトの検出などの個別のメソッドも用意されており、必要な情報だけを返す最適なメソッドを使用することで、返されるデータ ペイロードのサイズを削減できます。詳細については、.[NET SDK ドキュメント](https://learn.microsoft.com/dotnet/api/overview/azure/cognitiveservices/computervision?view=azure-dotnet) または [Python SDK ドキュメント](https://learn.microsoft.com/python/api/azure-cognitiveservices-vision-computervision/azure.cognitiveservices.vision.computervision) を参照してください。

## リソースのクリーンアップ

このラボで作成した Azure リソースを他のトレーニング モジュールに使用しない場合は、追加の料金が発生しないように削除できます。以下の手順に従ってください。

1. `https://portal.azure.com` で Azure ポータルを開き、Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。
2. 上部の検索バーで *Azure AI services multi-service account* を検索し、このラボで作成した Azure AI サービス マルチサービス アカウント リソースを選択します。
3. リソース ページで **削除** を選択し、リソースを削除する手順に従います。

## 詳細情報

この演習では、Azure AI Vision サービスの画像分析および操作機能の一部を探索しました。このサービスには、オブジェクトや人物の検出、その他のコンピュータ ビジョン タスクの機能も含まれています。

**Azure AI Vision** サービスの使用に関する詳細については、[Azure AI Vision ドキュメント](https://learn.microsoft.com/azure/ai-services/computer-vision/)を参照してください。
