---
lab:
 title: '音声の認識と合成'
 module: 'モジュール 4 - Azure AI サービスを使用して音声対応アプリを作成する'
---

# 音声の認識と合成

**Azure AI Speech** は、音声に関連する機能を提供するサービスで、以下を含みます：

- *音声からテキストへの変換* API：音声認識（聞こえる話し言葉をテキストに変換）を実装できます。
- *テキストから音声への変換* API：音声合成（テキストを聞こえる音声に変換）を実装できます。

この演習では、これらの API の両方を使用して、話す時計アプリケーションを実装します。

> **注意**
> この演習には、スピーカー/ヘッドフォンを備えたコンピューターが必要です。最良の体験を得るには、マイクも必要です。一部のホストされた仮想環境では、ローカルマイクからの音声をキャプチャできる場合がありますが、これが機能しない場合（またはマイクがまったくない場合）は、提供された音声ファイルを使用して音声入力を行うことができます。指示に従って、マイクを使用するか音声ファイルを使用するかに応じて異なるオプションを選択してください。

## *Azure AI Speech* リソースのプロビジョニング

サブスクリプションにまだない場合は、**Azure AI Speech** リソースをプロビジョニングする必要があります。

1. `https://portal.azure.com` で Azure ポータルを開き、Azure サブスクリプションに関連付けられた Microsoft アカウントを使用してサインインします。
1. 上部の検索フィールドで **Azure AI サービス** を検索し、**Enter** キーを押して、結果で **Speech サービス** の下の **作成** を選択します。
1. 次の設定を使用してリソースを作成します：

    - **サブスクリプション**: *Azure サブスクリプション*
    - **リソースグループ**: *リソースグループを選択または作成*
    - **リージョン**: *利用可能なリージョンを選択*
    - **名前**: *一意の名前を入力*
    - **価格レベル**: **F0**（*無料*）または **S**（*標準*）を選択します（F が利用できない場合）。
    - **責任ある AI 通知**: 同意します。
1. **確認と作成** を選択し、次に **作成** を選択してリソースをプロビジョニングします。
1. デプロイメントが完了するのを待ち、デプロイされたリソースに移動します。
1. **キーとエンドポイント** ページを表示します。このページの情報は後の演習で必要になります。

## Visual Studio Code でアプリを開発する準備

Visual Studio Code を使用して音声アプリを開発します。アプリのコードファイルは GitHub リポジトリに提供されています。

> **ヒント**: すでに **mslearn-ai-language** リポジトリをクローンしている場合は、Visual Studio Code で開きます。そうでない場合は、次の手順に従って開発環境にクローンします。

1. Visual Studio Code を起動します。
1. パレット（SHIFT+CTRL+P）を開き、**Git: Clone** コマンドを実行して `https://github.com/MicrosoftLearning/mslearn-ai-language` リポジトリをローカルフォルダにクローンします（フォルダはどこでも構いません）。
1. リポジトリがクローンされたら、Visual Studio Code でフォルダを開きます。

    > **注意**: Visual Studio Code がコードの信頼を求めるポップアップメッセージを表示した場合は、**はい、著者を信頼します** オプションをクリックします。

1. リポジトリ内の C# コードプロジェクトをサポートするための追加ファイルがインストールされるのを待ちます。

    > **注意**: ビルドとデバッグに必要なアセットを追加するように求められた場合は、**今は追加しない** を選択します。

## アプリケーションの構成

C# と Python の両方のアプリケーションが提供されており、同じ機能を備えています。まず、Azure AI Speech リソースを使用できるようにアプリケーションのいくつかの重要な部分を完了します。

1. Visual Studio Code の **エクスプローラー** ペインで **Labfiles/07-speech** フォルダに移動し、言語の好みに応じて **CSharp** または **Python** フォルダを展開し、その中にある **speaking-clock** フォルダを展開します。各フォルダには、Azure AI Speech 機能を統合するアプリの言語固有のファイルが含まれています。
1. コードファイルを含む **speaking-clock** フォルダを右クリックし、統合ターミナルを開きます。次に、言語の好みに応じて適切なコマンドを実行して Azure AI Speech SDK パッケージをインストールします：

    **C#**

    ```
    dotnet add package Microsoft.CognitiveServices.Speech --version 1.30.0
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-speech==1.30.0
    ```

1. **エクスプローラー** ペインで **speaking-clock** フォルダ内の構成ファイルを開きます：

    - **C#**: appsettings.json
    - **Python**: .env

1. 構成ファイルの値を更新して、作成した Azure AI Speech リソースの **リージョン** と **キー** を含めます（Azure ポータルの Azure AI Speech リソースの **キーとエンドポイント** ページで利用可能）。

    > **注意**: リソースの *エンドポイント* ではなく、*リージョン* を追加してください！

1. 構成ファイルを保存します。

## Azure AI Speech SDK を使用するコードの追加

1. **speaking-clock** フォルダにはクライアントアプリケーションのコードファイルが含まれています：

    - **C#**: Program.cs
    - **Python**: speaking-clock.py

    コードファイルを開き、既存の名前空間参照の下にある **Import namespaces** コメントを見つけます。次に、このコメントの下に、Azure AI Speech SDK を使用するために必要な名前空間をインポートするための言語固有のコードを追加します。

    **C#**: Program.cs

    ```csharp
    // Import namespaces
    using Microsoft.CognitiveServices.Speech;
    using Microsoft.CognitiveServices.Speech.Audio;
    ```

    **Python**: speaking-clock.py

    ```python
    # Import namespaces
    import azure.cognitiveservices.speech as speech_sdk
    ```

1. **Main** 関数で、構成ファイルからサービスキーとリージョンを読み込むコードがすでに提供されていることを確認します。次に、**Configure speech service** コメントの下に、Azure AI Speech リソースの **SpeechConfig** を作成するための次のコードを追加します。


    **C#**: Program.cs

    ```csharp
    // Configure speech service
    speechConfig = SpeechConfig.FromSubscription(aiSvcKey, aiSvcRegion);
    Console.WriteLine("Ready to use speech service in " + speechConfig.Region);
    
    // Configure voice
    speechConfig.SpeechSynthesisVoiceName = "en-US-AriaNeural";
    ```

    **Python**: speaking-clock.py

    ```python
    # Configure speech service
    speech_config = speech_sdk.SpeechConfig(ai_key, ai_region)
    print('Ready to use speech service in:', speech_config.region)
    ```

1. 変更を保存し、**speaking-clock** フォルダの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します：

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock.py
    ```

1. C# を使用している場合、非同期メソッドで **await** 演算子を使用することに関する警告を無視できます - 後で修正します。コードは、アプリケーションが使用する音声サービスリソースのリージョンを表示するはずです。

## 音声認識コードの追加

Azure AI Speech リソースの音声サービス用に **SpeechConfig** を作成したので、**音声からテキスト** API を使用して音声を認識し、テキストに変換できます。

> **重要**: このセクションには、2つの代替手順の指示が含まれています。動作するマイクがある場合は最初の手順に従ってください。音声入力を音声ファイルを使用してシミュレートする場合は、2番目の手順に従ってください。

### 動作するマイクがある場合

1. プログラムの **Main** 関数で、コードが音声入力を受け入れるために **TranscribeCommand** 関数を使用していることを確認します。
1. **TranscribeCommand** 関数で、**Configure speech recognition** コメントの下に、デフォルトのシステムマイクを使用して音声を認識し、テキストに変換するための **SpeechRecognizer** クライアントを作成するための適切なコードを追加します：

    **C#**

    ```csharp
    // Configure speech recognition
    using AudioConfig audioConfig = AudioConfig.FromDefaultMicrophoneInput();
    using SpeechRecognizer speechRecognizer = new SpeechRecognizer(speechConfig, audioConfig);
    Console.WriteLine("Speak now...");
    ```

    **Python**

    ```python
    # Configure speech recognition
    audio_config = speech_sdk.AudioConfig(use_default_microphone=True)
    speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
    print('Speak now...')
    ```

1. 次に、以下の **音声認識コマンドを処理するコードの追加** セクションに進みます。

---

### 代わりに音声ファイルからの音声入力を使用する場合

1. ターミナルウィンドウで、音声ファイルを再生するために使用できるライブラリをインストールするために次のコマンドを入力します：

    **C#**

    ```
    dotnet add package System.Windows.Extensions --version 4.6.0 
    ```

    **Python**

    ```
    pip install playsound==1.2.2
    ```

1. プログラムのコードファイルで、既存の名前空間インポートの下に、インストールしたライブラリをインポートするための次のコードを追加します：

    **C#**: Program.cs

    ```csharp
    using System.Media;
    ```

    **Python**: speaking-clock.py

    ```python
    from playsound import playsound
    ```

1. **Main** 関数で、コードが音声入力を受け入れるために **TranscribeCommand** 関数を使用していることを確認します。次に、**TranscribeCommand** 関数で、**Configure speech recognition** コメントの下に、音声ファイルから音声を認識し、テキストに変換するための **SpeechRecognizer** クライアントを作成するための適切なコードを追加します：


    **C#**: Program.cs

    ```csharp
    // Configure speech recognition
    string audioFile = "time.wav";
    SoundPlayer wavPlayer = new SoundPlayer(audioFile);
    wavPlayer.Play();
    using AudioConfig audioConfig = AudioConfig.FromWavFileInput(audioFile);
    using SpeechRecognizer speechRecognizer = new SpeechRecognizer(speechConfig, audioConfig);
    ```

    **Python**: speaking-clock.py

    ```python
    # Configure speech recognition
    current_dir = os.getcwd()
    audioFile = current_dir + '\\time.wav'
    playsound(audioFile)
    audio_config = speech_sdk.AudioConfig(filename=audioFile)
    speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
    ```

---

### 音声認識コマンドを処理するコードの追加

1. **TranscribeCommand** 関数で、**Process speech input** コメントの下に、音声入力をリッスンするための次のコードを追加します。関数の最後にあるコマンドを返すコードを置き換えないように注意してください：


    **C#**: Program.cs

    ```csharp
    // Process speech input
    SpeechRecognitionResult speech = await speechRecognizer.RecognizeOnceAsync();
    if (speech.Reason == ResultReason.RecognizedSpeech)
    {
        command = speech.Text;
        Console.WriteLine(command);
    }
    else
    {
        Console.WriteLine(speech.Reason);
        if (speech.Reason == ResultReason.Canceled)
        {
            var cancellation = CancellationDetails.FromResult(speech);
            Console.WriteLine(cancellation.Reason);
            Console.WriteLine(cancellation.ErrorDetails);
        }
    }
    ```

    **Python**: speaking-clock.py

    ```python
    # Process speech input
    speech = speech_recognizer.recognize_once_async().get()
    if speech.reason == speech_sdk.ResultReason.RecognizedSpeech:
        command = speech.text
        print(command)
    else:
        print(speech.reason)
        if speech.reason == speech_sdk.ResultReason.Canceled:
            cancellation = speech.cancellation_details
            print(cancellation.reason)
            print(cancellation.error_details)
    ```

1. 変更を保存し、**speaking-clock** フォルダの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します：

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock.py
    ```

1. マイクを使用している場合は、はっきりと「what time is it?」と言ってください。プログラムは音声入力をテキストに変換し、時間を表示するはずです（コードが実行されているコンピューターのローカル時間に基づいていますが、あなたの場所の正しい時間ではないかもしれません）。

SpeechRecognizer は約5秒間話す時間を与えます。音声入力が検出されない場合、「No match」結果を生成します。SpeechRecognizer がエラーに遭遇した場合、「Cancelled」結果を生成します。アプリケーションのコードはエラーメッセージを表示します。最も可能性の高い原因は、構成ファイルのキーまたはリージョンが間違っていることです。

## 音声合成の追加

話す時計アプリケーションは音声入力を受け入れますが、実際には話しません！音声を合成するコードを追加して修正しましょう。

1. プログラムの **Main** 関数で、コードが現在の時間をユーザーに伝えるために **TellTime** 関数を使用していることを確認します。
1. **TellTime** 関数で、**Configure speech synthesis** コメントの下に、音声出力を生成するための **SpeechSynthesizer** クライアントを作成するための次のコードを追加します：


    **C#**: Program.cs

    ```csharp
    // Configure speech synthesis
    speechConfig.SpeechSynthesisVoiceName = "en-GB-RyanNeural";
    using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig);
    ```

    **Python**: speaking-clock.py

    ```python
    # Configure speech synthesis
    speech_config.speech_synthesis_voice_name = "en-GB-RyanNeural"
    speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config)
    ```

    > **注意**: デフォルトのオーディオ構成は、出力用のデフォルトのシステムオーディオデバイスを使用するため、明示的に **AudioConfig** を提供する必要はありません。オーディオ出力をファイルにリダイレクトする必要がある場合は、ファイルパスを持つ **AudioConfig** を使用できます。

1. **TellTime** 関数で、**Synthesize spoken output** コメントの下に、音声出力を生成するための次のコードを追加します。関数の最後にある応答を印刷するコードを置き換えないように注意してください：

    **C#**: Program.cs

    ```csharp
    // Synthesize spoken output
    SpeechSynthesisResult speak = await speechSynthesizer.SpeakTextAsync(responseText);
    if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
    {
        Console.WriteLine(speak.Reason);
    }
    ```

    **Python**: speaking-clock.py

    ```python
    # Synthesize spoken output
    speak = speech_synthesizer.speak_text_async(response_text).get()
    if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
    ```

1. 変更を保存し、**speaking-clock** フォルダの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します：

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock.py
    ```

1. プロンプトが表示されたら、マイクに向かってはっきりと「what time is it?」と言ってください。プログラムは時間を伝えるはずです。

## 別の音声を使用する

話す時計アプリケーションはデフォルトの音声を使用していますが、これを変更することができます。音声サービスは、*標準* 音声やより人間らしい *ニューラル* 音声の範囲をサポートしています。また、*カスタム* 音声を作成することもできます。

> **注意**: ニューラル音声と標準音声のリストについては、Speech Studio の Voice Gallery を参照してください。

1. **TellTime** 関数で、**Configure speech synthesis** コメントの下にあるコードを次のように変更して、**SpeechSynthesizer** クライアントを作成する前に別の音声を指定します：

   **C#**: Program.cs

    ```csharp
    // Configure speech synthesis
    speechConfig.SpeechSynthesisVoiceName = "en-GB-LibbyNeural"; // change this
    using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig);
    ```

    **Python**: speaking-clock.py

    ```python
    # Configure speech synthesis
    speech_config.speech_synthesis_voice_name = 'en-GB-LibbyNeural' # change this
    speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config)
    ```

1. 変更を保存し、**speaking-clock** フォルダの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します：

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock.py
    ```

1. プロンプトが表示されたら、マイクに向かってはっきりと「what time is it?」と言ってください。プログラムは指定された音声で時間を伝えるはずです。

## 音声合成マークアップ言語の使用

音声合成マークアップ言語（SSML）を使用すると、XML ベースの形式を使用して音声の合成方法をカスタマイズできます。

1. **TellTime** 関数で、**Synthesize spoken output** コメントの下にある現在のコードをすべて次のコードに置き換えます（**Print the response** コメントの下のコードはそのままにします）：

   **C#**: Program.cs

    ```csharp
    // Synthesize spoken output
    string responseSsml = $@"
        <speak version='1.0' xmlns='http://www.w3.org/2001/10/synthesis' xml:lang='en-US'>
            <voice name='en-GB-LibbyNeural'>
                {responseText}
                <break strength='weak'/>
                Time to end this lab!
            </voice>
        </speak>";
    SpeechSynthesisResult speak = await speechSynthesizer.SpeakSsmlAsync(responseSsml);
    if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
    {
        Console.WriteLine(speak.Reason);
    }
    ```

    **Python**: speaking-clock.py

    ```python
    # Synthesize spoken output
    responseSsml = " \
        <speak version='1.0' xmlns='http://www.w3.org/2001/10/synthesis' xml:lang='en-US'> \
            <voice name='en-GB-LibbyNeural'> \
                {} \
                <break strength='weak'/> \
                Time to end this lab! \
            </voice> \
        </speak>".format(response_text)
    speak = speech_synthesizer.speak_ssml_async(responseSsml).get()
    if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
    ```

1. 変更を保存し、**speaking-clock** フォルダの統合ターミナルに戻り、次のコマンドを入力してプログラムを実行します：

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock.py
    ```

1. プロンプトが表示されたら、マイクに向かってはっきりと「what time is it?」と言ってください。プログラムは SSML で指定された音声で時間を伝え、その後に「Time to end this lab!」と言うはずです。

## さらなる情報

**音声からテキスト** および **テキストから音声** API の使用について詳しく知るには、[音声からテキストのドキュメント](https://learn.microsoft.com/azure/ai-services/speech-service/index-speech-to-text)および[テキストから音声のドキュメント](https://learn.microsoft.com/azure/ai-services/speech-service/index-text-to-speech)を参照してください。
