# 第1章 メンタルモデルの構築
## 1.1 LLM
### 1.1.1  LLMの仕組み
生成AIは文字どおり何かを生成する能力を有する人工知能を表す言葉です。生成AIを実現する人工知能技術の一つがLLMです。せっかくなので生成AI自身にLLMについて説明してもらいましょう。

> LLM（Large Language Model）は、大量のテキストデータを学習して言語を理解・生成できる大規模な人工知能モデルです。このモデルは、文章生成、翻訳、質問応答など多様な言語タスクをこなすことができ、長い文脈を理解して適切な応答を生成する能力を持っています。LLMの特徴的な点は、特定の分野に限らず幅広いトピックに対応できる汎用性と、人間らしい自然な対話ができることです。

また、多くのLLMが中核的機構として採用するTransformerについても説明してもらいました。

> Transformerは、自然言語処理タスクに革新をもたらした機械学習モデルのアーキテクチャです。このモデルは、注意機構（Attention mechanism）を中心に設計されており、入力シーケンス内の要素間の関係性を効率的に捉えることができます。Transformerの登場により、長文の理解や生成が向上し、機械翻訳や文章要約など、様々な言語処理タスクでの性能が大幅に改善されました。

Attention機構は、2017年にGoogle Researchによる論文『Attention Is All You Need』[2]で発表されました。理解するには数学やニューラルネットワークの知識が必要なので、生成AIの利用者の立場では論文を読む必要はありません。ですが、ざっくりとしたイメージを持つことは役に立つので、興味がある方は3Blue1Brown（スリーブルーワンブラウン）というYouTubeチャネルで公開されている解説動画を観てください。『GPTとは何か Transformerの視覚化 Chapter 5, Deep Learning』は日本語翻訳[3]されています。また、LLMの内部の計算過程を可視化する『Transfer Explainer』というWebサイト[4]もあります。

ざっくりと捉えると、Attention機能は文章の特定の部分に注意を向けて学習させる仕組みです。人間が文章を読む際に、重要な箇所に注目して理解するのと似たような処理を行います。これにより、文脈や関連性を考慮しながら、より適切な言語処理が可能になります。

### 1.1.2 LLMのメンタルモデル
LLMとは要は何であるかを一文で表現するなら、「大規模な学習データを用いて、文脈を考慮した上で確率的にもっともらしい文章や画像を予測して完成させるマシン」です。「文章を何が何でも完成させないと気がすまないマン」と擬人化して捉えることもできるでしょう。

たとえば以下のようにプロンプトを入力すると、

```
パンはパンでも
```

LLMは文章を完成させたくて仕方ないので、以下のように回答してくれます。

```
食べられないパンは？

これは日本語の言葉遊びですね。「パンはパンでも」という表現は、通常「パン」という言葉で
始まり、別の意味を持つ「パン」で終わる謎かけの導入部分です。

正解は「フライパン」です。

この言葉遊びの面白さは以下の点にあります：

1. 「パン」という言葉が二つの異なる意味で使われています：
   - 最初の「パン」は食べ物のパンを指します。
   - 二つ目の「パン」は「フライパン」の一部で、調理器具を指します。

2. 「食べられないパン」という矛盾した表現が使われています。通常、パンは食べるものですが、
フライパンは食べられません。

このような言葉遊びは、日本語の豊かな表現力と、同音異義語を活用した言葉の面白さを示しています。

他にも似たような言葉遊びがありますが、もし興味があれば、いくつか例を挙げることもできます。
```

さて、本書はITエンジニアを対象としていますので、もう少しモデルらしいものを考えてみましょう。LLMを大きなブラックボックスとして見ると、文章（一連のトークン）を引数にとり、後続の文章（一連のトークン）を予測して返す関数と捉えることができます。

![645188e75661-20240817](https://github.com/user-attachments/assets/0cde9310-c4ee-45e6-843c-d52d9615f98b)

Pythonのコードでこの関数を定義するなら、以下のようになります。

``` python
def llm(input_str: list[str]) -> list[str]:
    pass
```
もう少し解像度を上げて見ると、LLMによる生成は一回の処理で次に来るトークンを予測し、結果として得られたトークンを入力の末尾に加えて、さらに次に来るトークンを予測するので、以下のような繰り返し処理となります。

![27af64749656-20240817](https://github.com/user-attachments/assets/e076267c-d304-4186-ba33-b6555fe38769)

``` python
def llm(input_str: list[str]) -> str:
    pass
```

LLMをこのような関数として捉えたとき、この関数には次のような特徴があります。

可変長引数：任意の長さの入力を受け付ける
非決定的：同じ入力でも異なる出力を生成する可能性がある
ステートレス：各呼び出しは独立しており、以前の呼び出しの影響を受けない
引数は互いに非独立：各々の引数は相互に影響し合い、それにより振る舞いが決まる
二つ目の非決定的であるとは、すなわちランダムであることを意味しますが、このランダムさの度合いを表すのがTemperatureです。Temparatureが大きいほどランダム性が増します。0が最小で、その場合には確率が最も高いトークンが次のトークンとして出力されます。

四つ目の項目は、LLMは文脈や単語の関係性を考慮することを表しています。たとえば、次の図は前述のTransformer Explainerを用いて、"A boy is walking with his"に続く単語の候補を確率順に並べたものです。

![97bafe0541df-20240817](https://github.com/user-attachments/assets/649eed61-329a-4f60-8c10-1969d3fedc4e)

ここで入力のwalkingをplayingに変えて"A boy is playing with his"とすると、以下の図のようになります。

![e3f51b2b9adb-20240817](https://github.com/user-attachments/assets/b1a0be02-a57c-4b62-8546-120926b755a4)

walkingのときには候補に存在しなかったtoyが出現していますね。
つまり、

```
A boy is playing    with      his          ____
         (動詞)  + (前置詞) ＋ (所有代名詞) + (名詞)
```

という文法的な観点からの確率だけでなく、playする（遊ぶ）対象という意味も加味しての出現確率となっていることがわかります（walkingのときはmotherが圧倒的だったのに、playingだとfatherも善戦しているのも興味深いですね）。

## 1.2 プロンプトエンジニアリング
### 1.2.1 プロンプトエンジニアリングとは

プロンプトエンジニアリングについて、生成AIによる説明は以下のとおりです。

> プロンプトエンジニアリングは、AI言語モデルから望ましい出力を得るために入力（プロンプト）を最適化する技術です。適切な指示、文脈、例示を含むプロンプトを設計することで、AIの応答の質と関連性を向上させることができます。この技術は、AIモデルの能力を最大限に引き出し、特定のタスクや目的に合わせてAIの出力を調整するために不可欠です。

これまで述べたとおり、LLMは入力された文章の続きを予測・補完し、まとまった文章として完成させようとします。そしてその振る舞いはランダム性を持っています。ですので、雑な指示を与えても何かしらの回答を得ることができます。暇つぶしにAIチャットボットと雑談をするのならそれで構わないのですが、特定のタスクを完遂するにはより明確で具体的な指示が必要です。プロンプトエンジニアリングとは、そのための実践的な技法のことです。

### 1.2.2 プロンプトエンジニアリングのメンタルモデル
前節では、LLMを関数に見立てるメンタルモデルについて説明しました。この関数はランダム性を持つため、特定の課題に対してユーザーが期待する回答を導くためには、実行時に何らかの条件をつけて可能性を狭める必要があります。これは、無数に分岐する可能性の中から、望ましい結果へ誘導するイメージです。

![1dbf78817577-20240817](https://github.com/user-attachments/assets/b679de3e-6510-42dc-adac-12d54f2b612f)

この図は、LLMの出力可能性が枝分かれしていく様子を示しています。プロンプトエンジニアリングは、たくさんの分岐の中から望ましい経路を選択するためのテクニックと言えます。

LLM関数のメンタルモデルにおいて、可変長引数として入力されるトークンは互いに影響を与え合い、全体の振る舞いを決定します。こレはつまり、プロンプトの書き方を工夫することで、生成される文章をある程度コントロールできるということです。

このLLMの特性を利用して、ChatGPTやClaudeのような生成AIチャットサービスはユーザーとの対話を実現しています。その仕組みを、ざっくりと模式図に表すと以下のようになります。

![01ab23b8ff37-20240817](https://github.com/user-attachments/assets/8fa9c714-5b0c-4559-90c8-e726d2cdbbeb)

この図は、AIアシスタントの役割設定（システムプロンプト）とユーザー入力（ユーザープロンプト）を合わせてLLMに入力すると、LLMがそれに基づいた適切な回答を予測して出力する流れを示しています。システムプロンプトはサービス提供者によって固定されていますが、ユーザープロンプトはユーザー自身が自由に記述できます。

Claudeのシステムプロンプトは公表されている[5]ので一度見てみるとよいでしょう。

プロンプトエンジニアリングとは、このユーザープロンプトの書き方を工夫し、LLMから望ましい結果を引き出す技術です。例えば、単に「猫について教えて」と聞くのではなく、「猫の生態、歴史、人間との関係について、それぞれ100字程度で簡潔に説明してください」とより具体的に指示することで、構造化された回答を得られる可能性が高まります。
