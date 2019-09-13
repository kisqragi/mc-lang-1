# TODO 1.1
これはまだまだ終わらない。  
ミニキャン中はもちろん、ミニキャン後もこの課題だけは取り組んでいくつもりです。

# TODO 1.2
ここでまずかなり躓いた。  
けれどもkaitoさんのおかげで解決。  
まずはLLVM/CLANGのインストールをする。  
kaitoさんに教えてもらったサイトを参考にし、実行(https://apt.llvm.org/)
Install(stable branch)  
```
wget -O - https://apt.llvm.org/llvm-snapshot.gpg.key|sudo apt-key add -
```
バージョン8出ないとエラーが出るという事前情報スレッドにあったのでver8用にインストールをかます。  
```
apt-get install clang-8 lldb-8 lld-8
```
この後に
```
git clone https://github.com/yamaguchi1024/mc-lang-1.git
```
でローカルリポジトリの作成。  
リポジトリ上でmakeと打つと完成。  
かと思いきやllvm-configでエラーが出るのでllvm-config-8をリンクでllvm-configにする
```
sudo ln -s /usr/bin/llvm-config-8 /usr/bin/llvm-config
```
```
make
```
完了。  

なのですが、この後はもうソースを書いていくのですがここでもつまずきました。（初心者なので）  

### 出くわした問題
そもそもどういった流れで開発をしていけば良いかわからなかった。  
コメントを読んで実行`./mc test/test1.mc`ですがエラーばかり。  
何度見ても間違いはなさそう。どうすれば...
ここで気づきました。makeしてないじゃん。  
```
make
```
これしないと文法チェックとかしてくれないんですね。  
初心者すぎてわかりませんでした。(恥ずかしい)  
なので開発の流れは簡単で、
```
-> コードを書く
-> makeコマンドを使う
-> ./mc test/test1.mc
```
という感じですかね。makeしてない事に気づくまで意外とかかったので書いておきます。  

# TODO 1.3
ここからは簡単です。(これは嘘)  
```
if (isdigit(lastChar)) {
    std::string str = "";
    str += lastChar; 
    while (isdigit(lastChar = getNextChar(iFile))) {
        str += lastChar; 
    }
    const char* cstr = str.c_str();
    setnumVal(strtod(cstr, nullptr));
    return tok_number;
}
```
strtodの使用だけ少し面倒でchar型にしないといけなかったのでc_str()を使ってstring型からcharに変えました。  
strtodの第二引数は今回は特にいらないのでnullptrにしておきました。  

# TODO 1.4
```
if (lastChar == '#') {
    do {
        lastChar = getNextChar(iFile);
    } while(lastChar != EOF && lastChar != '\n');
    if (lastChar != EOF) {
        return gettok();
    }
}
```
コメント通り実装しただけです。  

# TODO 1.5
```
getNextToken();
    auto result = ParseExpression();
    if (CurTok !=  ')') {
        return LogError("expected ')'");
    }
    getNextToken();
    return std::move(result);
```
c++は使った事なかったのでよくわからなかったのですが、autoという便利なものがソース中にあったので真似して使いました。  
で、ここからが理解があまり深まってないのですがParseExpressionがunique_ptrなので値の移動は所有権ごと行う必要があります。メモリ解放の関係。  
なのでresultに結果を受け取って、それをreturnするときはstd::moveで所有権ごと値を渡しています。  

# TODO 1.6
```
int tokprec = GetTokPrecedence();

// 2. もし呼び出し元の演算子(CallerPrec)よりも結合度が低ければ、ここでは演算をせずにLHSを返す。
// 例えば、「4*2+3」ではここで'2'が返るはずで、「4+2*3」ではここでは返らずに処理が進み、
// '2*3'がパースされた後に返る。
if (tokprec < CallerPrec) {
    return LHS;
}

// 3. 二項演算子をセットする。e.g. int BinOp = CurTok;
int BinOp = CurTok;

// 4. 次のトークン(二項演算子の右のexpression)に進む。
getNextToken();

// 5. 二項演算子の右のexpressionをパースする。 e.g. auto RHS = ParsePrimary();
auto RHS = ParsePrimary();

// GetTokPrecedence()を呼んで、もし次のトークンも二項演算子だった場合を考える。
// もし次の二項演算子の結合度が今の演算子の結合度よりも強かった場合、ParseBinOpRHSを再帰的に
// 呼んで先に次の二項演算子をパースする。
int NextPrec = GetTokPrecedence();
if (tokprec < NextPrec) {
    RHS = ParseBinOpRHS(tokprec + 1, std::move(RHS));
    if (!RHS)
        return nullptr;
}

// LHS, RHSをBinaryASTにしてLHSに代入する。
LHS = llvm::make_unique<BinaryAST>(BinOp, std::move(LHS), std::move(RHS));
```
すみません。コメント頼りに実装しただけです。

# TODO 1.7
```
<codegen.h>
  switch (Op) {
  case '+':
    return Builder.CreateAdd(L, R, "addtmp");

  case '-':
    return Builder.CreateSub(L, R, "addtmp");

  case '*':
    return Builder.CreateMul(L, R, "addtmp");

  case '/':
    return Builder.CreateUDiv(L, R, "addtmp");

  default:
    return LogErrorV("invalid binary operator");
  }
```
```
<mc.cpp>
(追加)
BinopPrecedence['/'] = 40;
```

調子に乗って除算を実装しました。まあ、追加するだけだったのですが。  
除算は乗算と結合度が同じなので値を同じにしました。  


#### メモ
LHS = Left-Hand-Side    // 左辺  
RHS = Right-Hand-Side   // 右辺
