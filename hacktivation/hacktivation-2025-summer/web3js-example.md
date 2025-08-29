# Web3.jsのデモ
ここでの目標は、[Web3.js]( https://docs.web3js.org/ )というライブラリを使って、「JavaScriptから」以下3点あたりをやることです。
1. アカウントを作ってなんとなく触る
2. ETHの残高確認や送金を体験してみる
3. コントラクト（Ethereum上のプログラム）の関数を呼び出してみる

※この資料ではあくまでずっとテストネット（開発環境）を使うので、**メインネット（本番環境）は使いません**。<br>
※**秘密鍵**とか**リカバリーパスフレーズ**とかは、**公開しない**でください。<br>
※あくまで作成時点での情報を元に作成しているので、リンク先とか挙動とか諸々はご注意ください。<br>
※この資料に限らずですが、リンク先のURLが正しいかとかはよく確認して、詐欺に注意してください。

## SSHでAzureのテスト環境に接続（ここだけこのイベント特有の内容）
おまじない：
1. コマンドプロンプトとかターミナルとかを開く
2. `ssh <メールで配布したアカウント名>@<メールで配布したドメイン名>` を実行
3. パスワードを聞かれるので、メールで配布したパスワードを入れてEnter
4. 最後に何か聞かれたらyes

これでLinuxのプロンプトが出てきたらOK。
以後、コマンド実行はここでやります。

※このAzure環境では、[Node.js]( https://nodejs.org/ja/ )、[Web3.js]( https://docs.web3js.org/ )、[Foundry]( https://getfoundry.sh/ )をすでにインストールして、nodeにWeb3.jsのパスも通しています。（なので、ご自身のローカル端末のターミナルで以下をやりたい場合は、このへんの環境構築が必要なのをご注意ください。この資料の内容だけなら、Foundryは無くて大丈夫で、Node.jsとWeb3.jsのインストールと、必要に応じてパスを通すとかです）

## Node.jsの立ち上げ
`node` コマンドを実行すると、以下のようになる（もしかするとバージョンは違うかも）。
```shell
$ node
Welcome to Node.js v22.14.0.
Type ".help" for more information.
> 
```

## RPCノードの準備
ブロックチェーン（イーサリアム）ネットワークと通信したいので、通信先となる、ブロックチェーン（イーサリアム）ネットワークを構成している（公開）ノードを準備します。
1. [ChainList]( https://chainlist.org/ ) にアクセス
2. 「Include Testnets」にチェックを入れて、イーサリアムのテストネット（今回は「Holesky」）を検索
3. 「ChainID」が「17000 (0x4268)」と書いてある、「Holesky」の下矢印みたいなのを押して展開する
4. どれかを選んで（`1rpc.io`のやつは上手くいかないかも）手元にURLをコピーしておく

このURLを後ほど使うので、メモ帳とかにコピペしといてください。

## Web3.jsを使う
[Web3.js]( https://docs.web3js.org/ )は、フロントエンドからイーサリアム上のデータとやり取りするためのライブラリです。
これを使って、イーサリアムのネットワーク上でアカウントを作ったり、ETHを送金したり、スマートコントラクトの関数を呼び出したりできます。

ライブラリは他にもありますが（ぜひググったりして他も使ってみてください）、お試し的に体感いただくのにお手軽なので、今回はこちらを使います。

なお、時間が経つと、バージョンアップによってここの記載だとうまく動かなくなることがあるかもなのでご注意ください。

### 準備
Web3.jsのインスタンスを作成。Node.jsを開き直したりしたら、**毎回**これが必要。
```javascript
// Web3.jsの準備
const Web3 = require('web3').default; // 忘れがちになるかもだけど、Node.jsを開き直すとここも毎回必要
const web3 = new Web3("https://XXXXXXXXXXXXXXXXX"); // https://XXXXXXXXXXXXXXXXXはChainListで準備したURL
```

※今回のAzure環境ではインストール済みですが、本来は`npm install web3`とかでインストールしておく必要があります。

### アカウント作成
後で送金テストをしたいので、アカウントを2つ作っておきましょう。
```javascript
// 以下でアカウント作成（1個目のアカウント）
const account1 = web3.eth.accounts.create(); // 2個目以降のアカウントは、左辺の変数名を変える必要がある
console.log(account1); // 作成したアカウントを確認

// 以下でアカウント作成（2個目のアカウント）
const account2 = web3.eth.accounts.create(); // 2個目以降のアカウントは、左辺の変数名を変える必要がある
console.log(account2); // 作成したアカウントを確認
```

`console.log`で表示した内容を手元にメモっておいてください（特に`address`と`privateKey`の項目のところ）。

※ここで作成したアカウントを他でも使う場合（別のアプリを作ったときとかにこのアカウントに送金するとか）、少なくとも同じ人、関係する人が使っているなど、アカウント同士の関連が推測される可能性がある、というのは認識してお使いください。（毎回別のアカウントを作って、新しいアカウントたちでテストを完結させる方がより安全ではある）

### アカウントのETH残高を確認
まだ何もやってないので、このアカウントの残高は 0 ETH のはず。（`BigInt`型で末尾に`n`がついて`0n`とか表示されるかも。`BigInt`型は大きな整数を扱うための型）

#### コマンドで確認
```javascript
// then以降が無いとPromise（非同期処理の結果を扱うためのオブジェクト）が解決されなくて、うまく残高が表示されない
web3.eth.getBalance("0xXXXXXXXXXXXXX").then(console.log); // 0xXXXXXXXXXXXXXのところは、アカウント作成時にコピペしたアカウントのアドレスを入れる

// ETH単位にしたい場合（デフォルトはwei単位）
web3.eth.getBalance("0xXXXXXXXXXXXXX").then(balance => console.log(web3.utils.fromWei(balance, "ether") + " ETH")); // 0xXXXXXXXXXXXXXのところは、アカウント作成時にコピペしたアカウントのアドレスを入れる
```

#### ブロックチェーンエクスプローラーで確認
Etherscanでもアカウントの残高を確認。
今回はHoleskyテストネットを使うので、「etherscan holesky」あたりで検索すると[Holesky Testnet Explorer]( https://holesky.etherscan.io/ )というのが出てくるので、ここでアカウントアドレスを入れて検索します。

### FaucetでETHを取得
Faucet（蛇口）から、テスト用にETHを取得します。送金テストをこの後にやりたいので、まずは片方のアカウントにテスト用の残高を取得しましょう。

「Faucet　<ネットワーク名>」みたいな感じでググると出てきたりします。
今回は[Google Cloud]( https://cloud.google.com/application/web3/faucet/ethereum/holesky )のものを使ってみましょう。

1. 「Select network」で「Ethereum Holešky」を選択（↑のリンクから行くと、たぶんすでにこれが選択されている）
2. 「Wallet address or ENS name」に、アカウントアドレスを入れる（2つ作ったうちの1つだけでOK）
3. 「Recieve 1 Holešky ETH」を押す
4. しばらく待つと、「Drip complete」と出る：
   ```
   Drip complete
   Confirmation
   Testnet tokens sent! Check your wallet address.
   Network
   Ethereum Holešky
   Recipient
   0x...
   Transaction hash 
   0x...
   ```
   みたい感じ


### アカウントのETH残高を確認（再）
FaucetからETHを取得したアカウントは残高があって、もう1つのアカウント（FaucetからETHを取得していない方）は 0 ETH のはず。（コマンドだと`BigInt`型で末尾に`n`がついて`0n`とか表示されるかも。`BigInt`型は大きな整数を扱うための型）

#### コマンドで確認（再）
```javascript
// then以降が無いとPromise（非同期処理の結果を扱うためのオブジェクト）が解決されなくて、うまく残高が表示されない
web3.eth.getBalance("0xXXXXXXXXXXXXX").then(console.log); // 0xXXXXXXXXXXXXXのところは、アカウント作成時にコピペしたアカウントのアドレスを入れる

// ETH単位にしたい場合（デフォルトはwei単位）
web3.eth.getBalance("0xXXXXXXXXXXXXX").then(balance => console.log(web3.utils.fromWei(balance, "ether") + " ETH")); // 0xXXXXXXXXXXXXXのところは、アカウント作成時にコピペしたアカウントのアドレスを入れる
```

#### ブロックチェーンエクスプローラーで確認（再）
Etherscanでもアカウントの残高を確認。
今回はHoleskyテストネットを使うので、「etherscan holesky」あたりで検索すると[Holesky Testnet Explorer]( https://holesky.etherscan.io/ )というのが出てくるので、ここでアカウントアドレスを入れて検索します。


### ETHを送金してみる
一方のアカウントから別のアカウントへETHを送金してみましょう。
ここで、先ほどコピペしておいた`privateKey`も使います。

`<FROMADDR>`に送り元のアドレス（FaucetからETHを取得 **した** 方のアカウントアドレス）、`<TOADDR>`に送り先のアドレス（FaucetからETHを取得 **してない** 方のアカウントアドレス）、`<FROMPRIVKEY>`に送り元の`privateKey`を入れて、以下を実行します。
```javascript
web3.eth.accounts.signTransaction({
  from: "<FROMADDR>",
  to: "<TOADDR>",
  value: web3.utils.toWei("0.001", "ether"),
  gas: 21000,
  maxFeePerGas: web3.utils.toWei("50", "gwei"),
  maxPriorityFeePerGas: web3.utils.toWei("2", "gwei"),
  type: 2
}, "<FROMPRIVKEY>").then(s => web3.eth.sendSignedTransaction(s.rawTransaction));
```

※`value`のところで送金するETHの量を調整できます。<br>
※`gas`は送金手数料です。<br>
※**秘密鍵の直書きはデモ用**です。サービスにするときとかは、パスワードとかAPIキーみたいに`.env`を読み込む、みたいなやり方をしてください。このあたりはブロックチェーンとかSolidity固有ではないので、必要に応じて調べてみてください。

### アカウントのETH残高を確認（再々）
送金元の残高が減って、送金先の残高が増えているはずです。

#### コマンドで確認（再々）
```javascript
// then以降が無いとPromise（非同期処理の結果を扱うためのオブジェクト）が解決されなくて、うまく残高が表示されない
web3.eth.getBalance("0xXXXXXXXXXXXXX").then(console.log); // 0xXXXXXXXXXXXXXのところは、アカウント作成時にコピペしたアカウントのアドレスを入れる

// ETH単位にしたい場合（デフォルトはwei単位）
web3.eth.getBalance("0xXXXXXXXXXXXXX").then(balance => console.log(web3.utils.fromWei(balance, "ether") + " ETH")); // 0xXXXXXXXXXXXXXのところは、アカウント作成時にコピペしたアカウントのアドレスを入れる
```

#### ブロックチェーンエクスプローラーで確認（再々）
Etherscanでもアカウントの残高を確認。
今回はHoleskyテストネットを使うので、「etherscan holesky」あたりで検索すると[Holesky Testnet Explorer]( https://holesky.etherscan.io/ )というのが出てくるので、ここでアカウントアドレスを入れて検索します。


## コントラクトにアクセスしてみる
### ABIとは（ざっくり）
[ABI（Application Binary Interface）]( https://docs.soliditylang.org/ja/latest/abi-spec.html )は、コントラクト（Ethereum上にあるプログラム）を使うのに必要な情報です。詳しくはリンク先をご参照ください。

### USDCのABIを触ってみる（Sepoliaテストネット）
ステーブルコインとしてよく知られている、[USDC]( https://www.usdc.com/ )のコントラクトを使ってみましょう。

[USDCのデプロイアドレス一覧]( https://developers.circle.com/stablecoins/usdc-contract-addresses#testnet )から、Ethereumテストネット（引き続きテスト環境で実験しましょう）のデプロイアドレスを探してみます。
「Ethereum Sepolia」のアドレスが`0x1c7D4B196Cb0C7B01d743Fbc6116a902379C7238`だとわかります。
テストネットはいくつかあって、さっきまでは「Holesky」を使っていましたが、今回は（USDCのリストにHoleskyが無かったので）「Sepolia」の方を使ってみることにします。

[リンク先]( https://sepolia.etherscan.io/address/0x1c7D4B196Cb0C7B01d743Fbc6116a902379C7238 )に飛んでみると、このコントラクトに対応するEtherscanのページが表示されます。これがSepoliaテストネット上にあるUSDCのコントラクトです。

#### 準備
さっきまでと違うネットワークなので、一度`Ctrl+D`なんかでNode.jsを終了して、再度`node`コマンドでNode.jsを立ち上げましょう。

Holeskyのときと同様に、[ChainList]( https://chainlist.org/ ) でRPCノードを準備しましょう。
**「Include Testnets」へのチェックを入れるのを忘れない**ようにしつつ、今回は「sepolia」を検索します。ChainIDが「11155111」の「Ethereum Sepolia」というやつです（ChainIDで検索した方が良いかもしれない）。

RPCノードのURLを準備したら、再び以下でWeb3.jsのインスタンスを作成しましょう。
```javascript
// Web3.jsの準備
const Web3 = require('web3').default;
const web3 = new Web3("https://XXXXXXXXXXXXXXXXX"); // https://XXXXXXXXXXXXXXXXXはChainListで準備したURL
```

#### コントラクト情報を取得
Etherscanのページで、Contract ABIのところを、abi情報として設定してWeb3.jsの機能を呼び出します。
ただし、今回は天下りですが、[こちら]( https://sepolia.etherscan.io/address/0xda317c1d3e835dd5f1be459006471acaa1289068#code )のコントラクトからABIを取得します。
コントラクトのページで、Contract＞Codeに「Contract ABI」という項目があるので、これを全部コピーか、必要箇所のみコピーします。
今回は説明用に「name」がUSDCであることを見ようと思うので、「name」に相当するところだけをコピーしておきましょう。
```javascript
const abi = [{"inputs":[],"name":"name","outputs":[{"internalType":"string","name":"","type":"string"}],"stateMutability":"view","type":"function"}]; // 「Contract ABI」のうち、nameの情報がある部分
const usdc = new web3.eth.Contract(abi, '0x1c7D4B196Cb0C7B01d743Fbc6116a902379C7238'); // 0xXXXXXXXXXXXXX にはUSDCの"Imple"コントラクトのデプロイアドレス
```

※USDCがProxyパターンという作りをしているため、今回は特殊でした。
Proxyパターンという作りをしていない「普通の」コントラクトであれば、普通に元のコントラクトのところにあるABIを使えばOKです。
Proxyパターンとかは、少しアドバンスドな内容で今回は触れないことにします。ご興味があればぜひ調べてみてください。

#### USDCコントラクトの「name」情報を出力してみる
[Web3.jsのメソッドを呼び出す機能]( https://docs.web3js.org/libdocs/Contract/#generated-methods )を使って、ABIで取得したコントラクト情報にある「name」を取得する部分を`call()`してみましょう。
```javascript
usdc.methods.name().call().then(console.log);
```

これで、[USDCコントラクトのページ]( https://sepolia.etherscan.io/address/0x1c7D4B196Cb0C7B01d743Fbc6116a902379C7238#readProxyContract )でContract＞Read as Proxyにある「name」を押したときに出る内容（「USDC」）が出るはず。

ざっとではありますが、Ethereum上のコントラクトの情報をフロントエンドから呼び出す、というのは基本はここまでの内容です。
あとはググったり生成AIに聞いてみたり、各自で調べたり試したりしつつ応用していただく、と言う感じです。
Web3.js以外にも、Reactの場合は[Wagmi]( https://wagmi.sh/ )とかを使ってももちろんOKです。

この資料では、フロントエンドとバックエンド（ブロックチェーン）のつなぎ部分の基本的な流れが、ここでご紹介した内容であることを把握いただければOKです。

# MetaMaskを触ってみる
実際にユーザがアプリを触るときには、MetaMask経由でトランザクションの承認をしたりします。アプリの作りによりますが、MetaMaskとの連携が必須というわけではないです。

ここでの目標は、以下2点です。
1. 何となくMetaMaskのUIに触れてみる
2. フロントエンドからMetaMaskをどんな感じで呼ぶかのイメージをつかむ

※ここでも引き続きテストネット（開発環境）を使うので、**メインネット（本番環境）のアカウント（秘密鍵）は使いません**。<br>
※↑の方にある「アカウント作成」のところでも記載した通り、アカウント同士の関連性が推測される可能性があるので、MetaMaskアカウントも、自分で別の用途で使う場合は、念のため別のアカウントを作るとかが安パイだと思います。（このまま使っても基本的には大丈夫とは思いますが、こういうリスクあるよ、くらいの認識はしつつ使うのが賢明です）


## MetaMaskの準備
1. [MetaMask公式ページ]( https://metamask.io/ja/download )から、ブラウザ拡張機能のMetaMaskをインストールしてアカウント作成しておきます。
2. ブラウザ拡張機能のMetaMaskを起動して、アカウント追加するメニューから、アカウント作成時にメモっておいた秘密鍵をインポートします。
3. ネットワークを適宜設定します。（今回すでに作成しているアカウントはHoleskyテストネットのETHを持っていると思うので、Holeskyテストネットにするのが良いと思います）

ここまでで、作ったアカウントをMetaMaskで扱えるようになります。

### MetaMaskのUIお試し
Etherscanから[WETH]( https://holesky.etherscan.io/address/0x94373a4919b3240d86ea41593d5eba789fef3848 )（Wrapped ETH）のコントラクトに接続して、MetaMaskを連携させつつ関数を呼び出してみましょう。

Wrapped ETHは、ここでは「こういうトークンがあるんだな」くらいに思っていただくくらいでOKです。単に、MetaMaskのUIの簡単なお試しをするのに程よいコントラクトがこれかな、くらいで選んだだけです。MetaMaskのUIお試しという本筋からは外れるので詳細には触れないですが、ご興味ある方はぜひググってみてください。

以下の手順で、MetaMaskを使ってETHをWETHに変換してみます。

#### MetaMaskとアカウントの準備
1. MetaMaskで、HoleskyのETH残高が入っているアカウントを選んでおく

#### WETH残高の確認
1. Contract＞Read Contractを開いて「balanceOf」関数（WETH残高の確認）を選んで、MetaMaskで選んだ（これから使おうとしている）アドレスを入れて「Query」（WETH残高は`0`のはず）

#### ETHをWETHに変換
1. Contract＞Write Contractを開いて「Connect to Web3」
2. 内容をちゃんと読んで確認しつつ、OKとか接続とかでMetaMaskを接続する（「Connected - Web3 [`<アドレス>`]」になってたら接続できてる）
3. 「deposit」（入金）関数を選んで、自分が持っているETH残高の範囲の値を入れて「Write」
4. 【**注意：これをやると、HoleskyのETH残高が減ります。**やりたくない場合はやらなくても良いです】MetaMaskが呼び出されるので、ちゃんと受け入れられる内容になってたら「確認」
5. 少し待つと、トランザクションが「確認されました」とかになります
6. （不要なサイト連携は忘れる前に切っておく方が無難だと思うので）忘れる前に、MetaMaskの右上にあるメニューから「アクセス許可の管理」みたいなやつを探して、「接続解除」しておきましょう（これでページの再読み込みをすると、「Connected - Web3 [`<アドレス>`]」が「Connect to Web3」に戻ってるはず）

#### WETH残高の確認（再）
1. Contract＞Read Contractを開いて「balanceOf」関数を選んで、MetaMaskで選んでいる（今しがた接続していた）アドレスを入れて「Query」（WETH残高が増えているはず）


### 気が向いた人用
MetaMaskのロック解除用パスワード、シークレットリカバリーフレーズ、秘密鍵あたりは、何を消したらどうなるか、たとえば以下みたいなことをやって試すと挙動が分かりやすいかも。もちろんテストネットでやるのをオススメします。
1. テキトーなアカウント（と秘密鍵）を用意しておく（テストネットETHも入れておくと分かりやすいかも）
2. ブラウザAでMetaMask拡張機能をインストールしてアカウント作成
3. 秘密鍵をインポートして、MetaMaskにアカウントを追加する
4. シークレットリカバリフレーズをバックアップしておく
5. ブラウザBでMetaMask拡張機能をインストールして普通にアカウント作成
6. ブラウザAでもBでも良いので、MetaMask拡張機能を削除して再度インストールして、今度はシークレットリカバリフレーズでアカウント作成する


## MetaMaskをフロントエンドから呼ぶとき
MetaMaskとのやり取りは、（HTMLと）JavaScriptコードで制御します。ここでは概要だけ（挙動確認は省いて）ご紹介するので、詳しくはWeb3.js公式の「[Connecting to Metamask with Vanilla JS]( https://docs.web3js.org/guides/dapps/metamask-vanilla/ )」あたりを参考に動かしてみてください。

### めちゃくちゃ最低限やること
```javascript
// Web3.jsをimportしておく

// 1. `window.ethereum`でEthereum Provider(MetaMaskなど)の存在確認
if (window.ethereum) {

  // 2. Web3オブジェクトを定義
  const web3 = new Web3(window.ethereum);

  // 3. MetaMaskを使いたい場合、Ethereum ProviderがMetaMaskかどうかを確認
  if (window.ethereum.isMetaMask) {
    // Ethereum ProviderがMetaMaskだった場合の処理
  } else {
    // Ethereum ProviderがMetaMaskじゃない場合の処理
  }

  // Web3.js公式チュートリアルでは、ここでChainIdの取得などをしている
  // 詳細はWeb3.js公式チュートリアルを参照ください
  // ChainIDについては以下あたりを参照：
  //   https://eips.ethereum.org/EIPS/eip-155
  //   https://chainid.network/

  // 4. アカウント情報を取得したりしたい場合は `eth_requestAccounts` が必要
  // Web3.js公式チュートリアルでは、これをクリックイベントと紐づけている
  // awaitが分からない場合は、asyncとかthenとかと一緒に調べてみてください
  await window.ethereum.request({ method: "eth_requestAccounts" });
  const accounts = await web3.eth.getAccounts();

  // Web3.js公式チュートリアルでは、この後に署名の取り扱いなどをやっている

} else {
  // Ethereum Providerが見つからない場合の処理
}
```

`window.ethereum`については[MetaMask公式のAPIドキュメント]( https://docs.metamask.io/wallet/reference/provider-api/ )に記載があります。[かるでね氏のQiita記事]( https://qiita.com/cardene/items/9e796da49299f57bd5c7 )も参考になるかと思います。
