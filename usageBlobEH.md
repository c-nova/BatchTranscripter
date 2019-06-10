## 使い方（Azure Storage BLOB Event Hub Trigger 版）

1. （BLOB が無い場合）事前準備として Azure Portal から Azure Storage Account を作成し、音声ファイルを格納する BLOB コンテナを作成します。コンテナの作成までの流れは、以下の URL が参考になります。
   
   [Windows Azure ストレージ オブジェクトを作成します](https://docs.microsoft.com/ja-jp/sql/tutorials/lesson-1-create-windows-azure-storage-objects?view=sql-server-2014)

2. （BLOB が無い場合）同じ方法で「認識結果ファイルを格納する」コンテナを作成します。

3. 以降に BLOB Sorage から直接 音声 BLOB を Cognitive Services に送信する手順がありますが、Azure Portal からは BLOB 単位でのみ SAS (Signed Access Signature) を作成することができ、コンテナ単位で設定することはできません。コンテナに SAS を設定するには「Microsoft Azure Storage Explorer」(以下 Storage Explorer)というソフトウェアが必要になります。もし Storage Explorer をお持ちでない方は、以下のリンクから導入を行ってください（導入に当たって複雑な部分はありませんので手順は省きます）。
   
   [Microsoft Azure Storage Explorer](https://azure.microsoft.com/ja-jp/features/storage-explorer/)
   
4. Storage Explorer の導入が完了しましたら、スタートメニューより Storage Explorer を起動します。
   <img src="img/mase001.png" width=480><br>

5. Storage Explorer は初期状態では Azure に接続されていません。Azure アカウント、サブスクリプションに未接続の方は、[この手順](https://docs.microsoft.com/ja-jp/azure/vs-azure-tools-storage-manage-with-storage-explorer?tabs=windows#connect-to-an-azure-subscription)を参考に先ほど作成した BLOB Storage が所属するサブスクリプションへの接続を行ってください。接続が完了すると以下のような画面になります。
   <img src="img/mase002.png" width=480><br>

6. サブスクリプションのリストから音源ファイルを配置する BLOB コンテナ までナビゲートし、右クリックします。ポップアップするメニューから「Get Shared Access Signature」をクリックします。
   <img src="img/mase003.png" width=480><br>

7. Shared Access Signature の設定画面が開くため、「Permission」で全ての権限にチェックを入れ、「Expire time」で適切な有効期限日時を設定し、「Create」をクリックします。
   
   <img src="img/mase004.png" width=240><br>

8. SAS が作成されるため、「URL」欄の右端にある「Copy」をクリックして URL を保存します。この URL は後ほど利用するためメモを保存します。この SAS は漏洩すると全権限を持った操作が行われるため、秘匿性の高い場所に保管を行います。保存が完了した場合には、Storage Explorer を閉じて終了します。     
   <img src="img/mase005.png" width=240><br>

9.  Web ブラウザを起動し、Azure Portal から Logic Apps を作成します。
   <img src="img/inst001.png" width=480><br>
   <img src="img/inst002.png" width=240><br>
   <img src="img/inst003.png" width=240>

10. デプロイされた Logic Apps リソースを開き、「空のロジック アプリ」を作成します。   
   <img src="img/inst004.png" width=480><br>
   <img src="img/inst005.png" width=480>

11. 「ロジック アプリ デザイナー」が開くので「コネクタとトリガーを検索する」欄で「Azure Event Grid」と入力し、検索結果から「Azure Event Grid」を選択します。
   <br><img src="img/inst101.png" width=480>

12. トリガーの検索結果から「When a resource event occurs (プレビュー)」を選択します（環境によっては稀に Event Hub トリガーが動作しない場合があります。その場合には、[Blob トリガーを利用する場合](usageBlob.md)の手順に従って作業を進めてください。このトリガーはリアルタイムに検出可能であるのに対して、BLOB トリガーは一定時間間隔で検出可能となります）。
   <br><img src="img/inst102.png" width=480>

13. 初めて Event Grid 接続を行う場合には、以下のような画面が表示されることがあります。適切なテナント名を選択し、「サインイン」をクリックします。
   <br><img src="img/inst103.png" width=480>

14. サインイン画面がポップアップされるので、適切なアカウントでサインインします。
   <br><img src="img/inst104.png" width=240>

15. 下記のような画面が表示されるので「Allow access」をクリックします。
   <br><img src="img/inst105.png" width=240>

16. これで Event Grid トリガーが作成されるため、名前を「Speech BLOBフォルダに新規ファイルが作成された時」にし、「Subscription」の項目で適切な Azure サブスクリプションを選択し、「Resource Type」から「Microsoft.Storage.StorageAccount」を選択します。「Rsource Name」で利用する Azure Storage Account の名称を選択し、「Event Type 項目 - 1」は「Microsoft.Storage.BlobCreated」を選択します。「Prefix Filter」では Event Grid で認識可能な呼び出し名の「プレフィックス」を設定しますが、例えば「/blobServices/default/containers/<音声格納コンテナ名>/blobs/」を選択します。この部分が選択されていない場合、同じ Storage Account の異なる Blob にオブジェクトが作成される度にトリガーが発行されるため、ご注意ください。
   <br><img src="img/inst106.png" width=480>
    
17. 続けて Blob への接続情報を作成するために、一時的に後続のステップを追加します。トリガーの下部にある「＋新しいステップ」をクリックします。
   <br><img src="img/inst107.png" width=480>

18. 今回は接続情報を作成するために一時作成するので、「コネクタとアクションを検索する」欄に「Blob」と入力します。検索が完了すると画面下部の「アクション」欄に結果が表示されるため、「BLOB の一覧表示」を選択します。
   <br><img src="img/inst108.png" width=480>

19. 接続する BLOB の選択画面が表示されますので、接続したい BLOB を選択し、「作成」をクリックします。「接続名」は任意なのですが、後続作業でペーストするコードでの記述が「azureblob」となっているので、ここではそのように名前を付けます。他の名前に変更する場合には、後続の作業で適宜変更を行ってください。
   <br><img src="img/inst109.png" width=480>

20. 以降の作業に必要になる接続情報を取得するために、「コードビュー」をクリックします。
   <br><img src="img/inst110.png" width=480>

21. 先ほど作成した接続情報（ここでは azueblob）の接続情報をコピーし、メモ帳等で保管します。
   <br><img src="img/inst111.png" width=480>

22. これで接続設定の作成と保管が完了しました。このアクションはもう必要ありませんので、「デザイナー」ボタンでデザイン画面に戻り、ボックス右上の「・・・」をクリックし、「削除」をクリックしてアクションを削除します。
   <br><img src="img/inst112.png" width=480>

23. 「保存」をクリックし、一旦内容を保存した後、「コードビュー」をクリックします。
   <br><img src="img/inst113.png" width=480>

24. 開いたコードの `definition` のサブアイテム `$schema`から `parameters` まで(triggersの直前まで)の内容を選択し、[BatchTranscriptforBlobEH.json](BatchTranscriptforBlobEH.json) のファイルの内容で全て上書きコピーします（triggers は消さないように注意してください）。コピー完了後、先ほど保存した接続情報を ` $connections` のサブアイテムの `values` に追加します。
   <br><img src="img/inst114.png" width=480>

25. 変更後、「保存」をクリックします。その後「デザイナー」ボタンをクリックし、デザイナー画面を開きます。
   <br><img src="img/inst115.png" width=480>

26. 次に「speechSubKey 変数を初期化する」を開き、「値」の欄に Speech Services のサブスクリプション キーを入力します（サービス未作成の場合は別途作成してください）。このシステムでは REST の呼び出し先が「japaneast（東日本）」となっていますので、東日本でサービスを作成するか、Speech Services を呼び出す箇所のURLの変更を行ってください。
   <br><img src="img/inst207.png" width=480>

27. 次に「バッチ音声認識用 JSON の作成」を開き、「recordingUrl」のプロパティの変更を行います。"https://<Blob Host Name>.blob.core.windows.net@{uriPath(triggerBody()?['data'].url)}?<Blob SAS Parameters>" と なっている箇所を、例えば blobstore という名前のストレージアカウントで作成した場合には「blobstore.blob.core.windows.net」と、「?」以降の部分は先ほど保存したURLの「?」以降の部分で置き換えます。
    <br><img src="img/inst209.png" width=480>

28. 結果のテキストファイルを配置する BLOB のフォルダを指定します。
    <br><img src="img/inst116.png" width=480>

29. 最後に「保存」、「実行」ボタンを押し、トリガーを実行します（この時点では何も動きません）。

30. 10項で指定した BLOB コンテナに音声ファイルを配置します。ここでは例として Azure Portal を使用してファイルを配置していますが、Web アプリケーションや Storage Explorer で配置しても問題ありません。
    <br><img src="img/inst117.png" width=480>

31. Logic Apps の「概要」をクリックし、「実行の履歴」に音声認識処理が追加されるのを待ちます（Event Grid トリガーを使用している場合には即時実行されます）。
    <br><img src="img/inst022.png" width=480>

32. 追加された処理をクリックすると、現在の実行状況が表示されます。最後の処理が完了するまで待機します。
    <br><img src="img/inst023.png" width=480>

33. 全ての処理が完了後、元の音声ファイルが削除され、元のファイル名+.txt ファイルの音声認識結果ファイルが作成されていれば完了です。
    <br><img src="img/inst118.png" width=240>

    <br><img src="img/inst119.png" width=480>
