# はじめに

本研究では眼球運動に基づき文の読み時間を推定し，ヒトの文処理機構の解明を目指すとともに，
工学的な応用として文の読みやすさのモデル構築を行う．対象言語は日本語とする．

データとして\[subsec:bccwj-eyetrack\]節に示す『現代日本語書き言葉均衡コーパス』(BCCWJ)
\@xciteの読み時間データ BCCWJ-EyeTrack
\@xciteを用いる．\[subsec:prev\]節に示す通り，過去の研究は統語・意味・談話レベルのアノテーションを重ね合わせることにより，
コーパス中に出現する言語現象と読み時間の相関について検討してきた．
一方，Hale は，が文処理過程に影響を与えると言及し，漸進的な文処理
の困難さについて情報量基準に基づいたモデルをサプライザル (Surprisal
Theory) として定式化している@xcite．このサプライザル
に基づく日本語の読み時間の分析が求められている．

しかしながら，日本語においては，心理言語学で行われる読み時間を評価する単位と，
コーパス言語学で行われる頻度を計数する単位に齟齬があり，この分析を難しくしていた．
具体的には，前者においては一般的に統語的な基本単位である文節が用いられるが，
後者においては斉一な単位である短い語（国語研短単位など）が用いられる．

この齟齬を吸収するために，単語埋め込み@xciteの利用を提案する．
単語埋め込みは前後文脈に基づき構成することにより，
単語の置き換え可能性を低次元の実数値ベクトル表現によりモデル化する．
このうち skip-gram モデルは加法構成性を持つと言われ[^1]，
句を構成する単語のベクトルの線形和が，句の置き換え可能性をモデル化できる
\@xcite．

日本語の単語埋め込みとして，『国語研日本語ウェブコーパス』(NWJC)@xciteか
ら fastText \@xcite により構成した NWJC2vec \@xciteを用いた．
ベイジアン線形混合モデル@xciteに基づく統計分析[^2]の結果， skip-gram
モデルに基づく単語埋め込みのノルムと隣接文節間のコサイン類似度が，
読み時間を予測する因子となりうることが分かった． 前者のノルムがを，
後者の隣接文節間のコサイン類似度が隣接をモデル化すると考える．

以下，\[sec:related\]節に前提となる関連情報について示す．
\[sec:method\]節に分析手法について示す．
4節に結果と考察について示し，5節でまとめと今後の展開を示す．

# 前提

## BCCWJ-EyeTrack

BCCWJ-EyeTrack \@xciteは，BCCWJ の新聞記事サンプル 20記事に対して，
日本語母語話者24人分（女性19人, 未回答1人, 男性4人;
20--55歳）の読み時間を収集して，データベース化したものである．
自己ペース読文法 (SELF: Self-Paced Reading)
と視線走査法に基づく文節単位の5種類 の 読み時間 (FFT: First Fixation
Time, FPT: First Pass Time, SPT: Second Pass Time, RPT: Regression Path
Time, Total: Total Time) が視線停留オフセット値に基づいて算
出されている． 図 \[fig:eyetrack\] に読み時間のタイプの集計例を示す．

表 \[tbl:data\] に BCCWJ-EyeTrack のデータ形式について示す． `surface`
は単語の表層形である． 読み時間 (i.e., `time`) は対数に変換したデータ
(i.e., ` logtime`)も保持し，一般化線形混合モデル用に用いられる．
`measure` は読み時間のタイプ {SELF, FFT, FPT, RPT, SPT, TOTAL}を表す．
`sample, article, metadata_orig, metadata` は記事に関連する情報である．
`space` は文節境界に半角空白を入れたか否かを示す． `length`
は表層形の文字数である． `is_first,is_last,is_second_last`
はレイアウトに関する特徴量である．
`sessionN, articleN, screenN, lineN, segmentN`
は要素の呈示順に関する特徴量 である． `subj` は被験者のID
で統計処理においてランダム要因として用いる． `dependent`
は当該文節に係る文節の数を人手で付与したもの \@xciteである．

また，被験者が記事をきちんと読んでいるか確認するために，各記事を読んだ後に，
Yes/Noで解答できる簡単な内容理解課題を課した．視線走査法の内容理解課題の正解率
は99.2% (238/240)で，自己ペース読文法の内容理解課題の正解率77.9%
(187/240)より 有意に高かった(p@xmath00.001)[^3]．

## サプライザル

は文脈中に出現する言語的な事象 \@xmath1（音韻的特徴・単語・発話）が
伝達する情報を次式によりはかることができ，これをと呼んだ：
$${\tt Surprisal}(x) = \log_{2} \frac{1}{P(x|{\tt context})}$$ Surprisal
は \@xmath2 の（文脈
`context`による条件付き）確率が低い場合に大きい値をとり，確率が高い場合
に小さい値をとる．さらに単語を処理する認識努力(cognitive effort)はその
surprisal に比 例するとしている：
$${\tt Effort} \propto {\tt Surprisal}$$ Surprisal
は前方部分単語列に基づいて選好される parse 木を再考するコストと
ともに後方部分単語列を期待しうるか否かの困難さをモデル化する． Surprisal
は，確率的言語モデルに基づくもの[^4]・ N-gram Surprisal[^5]・ Parser
(PCFG) Surprisal[^6]などがあり， は Earley 法に基づく Parser
Surprisalを提案した． は，前方部分単語列に対する可能な parse
木の確率分布を反映する KL ダイバージェンスに基づく
surprisal[^7]を提案した．

また Pynte らは Latent Semantic Analysis (LSA) \@xciteによる分散表現を用
いた semantic surprisal \@xcite を提案し，英語の視線走査データである
Dundee Corpus のモデル化を行っている． 具体的には，300次元の LSA
モデルを用いて， -based surprisal （前隣接単語と注
目単語との意味的類似度）と -based surprisal
（従前に出現する前隣接単語以外の単語と注目単語の意味的類似度）
について一般化線形混合モデルにより調査した．wLSA-based surprisal
は読み時 間を予測する効果が確認されたが，sLSA-based surprisal
には効果が確認できなかった． Mitchell らは，Latent Direchret Allocation
(LDA) \@xcite による分散表現を用いた LDA-based surprisal
を提案した@xcite．

## 単語埋め込みと NWJC2vec

単語埋め込み@xcite単語を数百次元のベクトルで表現する技術である．
従来はその単語か否かを表す one-hot 表現が用いられていたため，
大規模語彙を表現するために高次元ベクトルになっていた．
学習の際のモデルとして，文脈から単語を推定する CBOW
モデルと単語から文脈を推定する skip-gram モデルが提案されている．

単語埋め込みにより，単語の入れ替え可能性を低次元のベクトルで表現できるようになったほか，
skip-gram モデルには加法構成性と呼ばれる句を構成する語ベクトルの和が，
句ベクトルとして利用できるという良い性質を持つ．
本研究ではこの性質を，日本語における語を計数する単位と読み時間を評価する単位の齟
齬の吸収に用いる．

NWJC2vec \@xcite は NWJC
258億語から訓練した日本語の単語埋め込みデータである． fastText
\@xciteを用いて訓練した 300次元の CBOW および skip-gram
モデル[^8]を用いる．
この学習した単語ベクトルを用いて，視線走査データの集計単位である文節単位のベクト
ルを合成する．合成には線形和を用いた．

## BCCWJ-EyeTrack の過去の分析

BCCWJ-EyeTrack
に対して，統語・意味・談話レベルのアノテーションを重ね合わせて，
様々な言語現象に対してヒトがどのような反応をするのかについて検討を進めてきた．
は被験者属性を対象とし，記憶力がある群は読む速度が速いが全読
み時間は記憶力がない群と変わらないこと，語彙力がある群が読み時間が長いことを明ら
かにした． 浅原ら[-@Asahara-2019d]は文節係り受けアノテーション
BCCWJ-DepPara
\@xciteと対照比較を行い，係り受けの数が多い文節ほど読み時間が短くなることを明らかにした．
は節情報アノテーション BCCWJ-ToriClause \@xcite
と対照比較を行い，節末の読み時間が短いことを明らかにした．
は分類語彙表番号アノテーション BCCWJ-WLSP \@xcite
と対照比較を行い，統語分類の「用の類」@xmath7「相の類」@xmath8「体の類」の順で読み時間が
長くなる傾向と，意味分類の「関係」が他の分類（「主体」「活動」「生産物」「自然
物」）と比べて読み時間が短くなる傾向を明らかにした．
は情報構造アノテーション BCCWJ-Infostr \@xcite
と対照比較を行い，共有性において旧情報(hearer-old)が新情報(hearer-new)よりも読み時間が短いことを明らかにした．
は述語項構造・共参照情報アノテーション BCCWJ-PAS \@xcite
と対照比較を行い，主語がゼロ代名詞の際に外界照応として二人称を指す場合の述語にお
いて，SPT が短くなることを明らかにした．
これらの分析には，サンプルと被験者をランダム要因とし，アノテーションを固定要因と
した対数時間に対する一般化線形混合モデルかベイジアン線形混合モデル
\@xciteに基づく方法を用いている．

# 分析手法

分析においては，いくつかの要因に基づく線形式に基づいて，読み時間をベイジアン線形
混合モデル@xciteにより推定し，その係数を見ることにより進める． 図
\[fig:formula\] に推定に用いた線形式を示す． 分析は，
分散表現のノルムと隣接文節類似度に基づくもの (@xmath9)，
頻度情報に基づく当該文節の出現確率のみに基づくもの (@xmath10)，
両方を用いたもの基づくもの (@xmath11)を対比する． 分散表現は
\[subsec:nwjc2vec\]節に示した fastText に基づく NWJC2vec 300次元のもの
を用い，CBOW モデルと skip-gram モデルの両方を比較した．
対象とする読み時間は，読み戻しが可能な視線走査法の FFT, FPT, RPT, TOTAL
とする． SPT については，\[sec:spt\] 節に述べる．

まず読み時間 `time` を対数正規分布 lognormal によりモデル化し， 期待値を
\@xmath12，分散を \@xmath13 とする． 式において \@xmath14
は基本的な要因を表し，@xmath15 を切片とする． \@xmath16
は文節の文字数に対する係数， \@xmath17
は文節間に半角空白を入れたか否かの係数である． \@xmath18, \@xmath19,
\@xmath20, \@xmath21, \@xmath22 が呈示順に対する係数， \@xmath23,
\@xmath24, \@xmath25 がレ イアウト情報に対する係数である．
その他，記事に対するランダム係数として \@xmath26を，
に対するランダム係数として \@xmath27を考慮する．

\@xmath30 は当該文節に係る文節の数に対する係数である．
先行研究においては，PCFG
の部分木により統語構造に基づく効果をモデル化していた．
日本語においては，比較的語順が自由な言語であるために句構造木ではなく，
文節係り受け木により評価する．
日本語は主辞後置言語であり，当該文節に係る文節は基本的に全て前置することから，
この係数によって実質的に統語構造に基づく効果がモデル化できると考える[^9]．

単語ベクトルから構成した文節ベクトルの情報の二つの情報を用いる．
一つは当該文節ベクトルのノルム \@xmath31である．
もう一つは当該文節ベクトルと左隣接ベクトルのコサイン類似度
\@xmath32である． この上で，単語埋め込みを考慮した期待値として
を検討する． \@xmath34
は単語埋め込みに基づく文節ベクトルのノルムに対する係数， \@xmath35
は単語埋め込みに基づく左隣接文節とのコサインに対する係数であり，これを評価する．
分散表現のモデルとして，CBOW と skip-gram の二つを評価する．

比較対照として，単語の頻度を考慮した期待値として \@xmath38を検討する．
\@xmath39 は文節内頻度に対する係数である．
単語の頻度に基づく手法については，文節間の連接を考慮しない．
文節内頻度は文節内の単語の頻度の相乗平均を評価する[^10]．
相乗平均を評価する際にゼロ頻度は 1 を乗じた．
なお，相加平均でも評価したがモデルが収束しなかった．

最後に，単語埋め込みと単語の頻度の両方を考慮した \@xmath40を検討する．

分析においては，[^11] を用いた． 500 iter の warm up のあと，5000 iter
を 4 chains 行った．

# 結果と考察

表 \[tbl:result\] に各モデルの分析結果を示す．詳細な結果については，
\[sec:detailed\] 節に示す．

推定される mean が 0.00 から@xmath41 SD 以上の差があるものに + もしくは
\@xmath42 を付与 する． 0 は mean が@xmath43 SD 以内のものである． +
はその値が大きければ，読み時間が長くなることを示す．
\@xmath44はその値が大きければ，読み時間が短くなることを示す．

まず， いずれの結果も で係り受
けが多ければ多いほど，読み時間が短くなった[^12]．
つまり，次に述べる結果は係り受けの効果を確認したうえでの付加的な効果である．
単語埋め込みに基づくモデルにおいては，
隣接文節間類似度が大きければ大きいほど
読み時間が短くなる傾向が見られた(@xmath47)． skip-gram
モデルにおいては，ベクトルのノルムが大きければ大きいほど
読み時間が長くなる傾向が見られた(@xmath48)． この傾向は CBOW
には見られなかった． 次に，頻度の相乗平均に基づくモデル (@xmath51)
においては， 高頻度のものが読み時間が短くなる傾向がみられた．
単語埋め込みと頻度の双方を考慮したモデル (@xmath52)では， skip-gram の
FFT 以外において，両者を個別にモデル化したものを合成したような結果
が得られた．

以下，結果について考察する．

まず，文節間の隣接ついては， wLSA-based surprisal \@xciteや LDA-based
surprisal \@xcite と同様に， fastText
による単語埋め込みに基づくモデルでもモデル化できることが確認された．
単語の頻度情報からは文節単位の隣接尤度の推定が困難であった．
加法構成性に基づき構成した文節単位のベクトルのコサイン類似度が，
適切に隣接尤度をモデル化できた．

文節内の単語の頻度の相乗平均は，その文節の生起確率を表す．
確率が高ければ高いほど読み時間が短くなることが適切にモデル化できている．
これは文節単位の unigram surprisal を適切にモデル化できていると考える．

skip-gram の FFT 以外においては，この unigram surprisal
と異なる文節単位の特徴と して， 単語埋め込みのノルムの効果が認められた
単語埋め込みのノルムは，他の単語の を表現していると考えられる．

いずれの結果も係り受けの効果とともに表れていることから，
が発見できたといえる．

# おわりに

本研究では，日本語の読み時間の推定のために単語埋め込みを用いることを提案した．
英語などで進められている surprisal の分析において，
単語の頻度に基づく確率が用いられている．
しかしながら，日本語においては頻度を計数する単位と読み時間を評価する単位との齟齬
があり， この分析を難しくしていた． 今回 skip-gram
の単語埋め込みを用いて，
ベクトルの線形和により文節ベクトルを構成することにより，この問題を解決した．
文節ベクトルのノルムが当該文節のをモデル化し，ノルムが大きければ大きい
ほど読み時間が長くなることを確認した．
さらにこれらの結果は統語的なモデルとともに導入 されるものであり，新しい
surprisal を発見したといえる．
また，先行研究と同様に，左隣接文節のベクトルと当該文節ベクトルのコサイン類似度が，
を適切にモデル化できることを確認した．

工学的な応用として重要な点として，これらの単語埋め込みに関する情報は
形態素解析器などで単語単位に割り当て可能であり，
線形和やコサイン類似度など比較的軽い演算で計算できる．
今回用いた統計モデル線形式であることから，
本研究で提案する単語埋め込みに基づくモデルは，NWJC2vec
に収録されている語で
構成される文章であれば，人間が解釈しやすい線形式で読み時間を与えることができる．
本モデルを用いて，読み時間に基づく文章の読みやすさの自動評価ができると考える．

本研究は，国立国語研究所コーパス開発センター共同研究プロジェクト「コーパスアノテーションの拡張・統合・自動化に関する基礎研究」によるものです．
本研究の一部はJSPS科研費 基盤研究(A) 17H00917，挑戦的研究（萌芽）
18K18519，新学術領域研究 18H05521 の助成を受けたものです．

::: thebibliography
Asahara, M. 2018a. Between Reading Time and Zero Exophora in
Japanese. In READ2018: International Interdisciplinary Symposium on
Reading Experience & Analysis of Documents,  34--36.

Asahara, M. 2018b. NWJC2Vec: Word embedding dataset from 'NINJAL Web
Japanese Corpus'. , (2),  7--25.

浅原正幸 . 名詞句の情報の状態と読み時間について. , (5),  527--554.

浅原正幸 . 日本語の読み時間と節境界情報---主辞後置言語における wrap-up
effect の検証. , (2),  301--328.

浅原正幸加藤祥 . 読み時間と統語・意味分類. , (2),  219--230.

Asahara, M., Maekawa, K., Imada, M., Kato, S.,  Konishi, H. . Archiving
and Analysing Techniques of the Ultra-large-scale Web-based Corpus
Project of NINJAL, Japan. , (1--2),  129--148.

浅原正幸松本裕治 .
『現代日本語書き言葉均衡コーパス』に対する文節係り受け・並列構造. , (4),
 331--356.

浅原正幸小野創宮本エジソン正 .
『現代日本語書き言葉均衡コーパス』の読み時間とその被験者属性. ,
 473--476.

浅原正幸小野創宮本エジソン正 . BCCWJ-EyeTrack
『現代日本語書き言葉均衡コーパス』に対する読み時間付与とその分析. , ,
 To Appear.

浅原正幸大村舞 . BCCWJ-DepParaPAS:
『現代日本語書き言葉均衡コーパス』の係り受け・並列構造と述語項構造・共参照アノテーションの重ね合わせと可視化. ,
 489--492.

Blei, D. M., Ng, A. Y.,  Jordan, M. I. . Latent Dirichlet Allocation. ,
,  993--1022.

Bojanowski, P., Grave, E., Joulin, A.,  Mikolov, T. . Enriching Word
Vectors with Subword Information. , ,  135--146.

Griffiths, T. L., Steyvers, M.,  Tanenbaum, J. B. . Topics in Semantic
Representation. , (2),  211--244.

Hale, J. . A Probabilistic Earley Parser as a Psycholinguistic Model. In
Proceedings of the 2nd Conference of the North American Chapter of the
Association for Computational Linguistics,  2,  159--166.

加藤祥浅原正幸山崎誠 .
分類語彙表番号を付与した『現代日本語書き言葉均衡コーパス』の書籍・新聞・雑誌データ. ,
(2),  134--141.

Landauer, T. K.  Dumais, S. T. . A Solution to Plato's problem: The
Latent Semantic Analysis Theory of Acquisition, Induction and
Representation of Knowledge. , (2),  211--240.

Levy, R. . Expectation-based Syntactic Comprehension. , ,  1126--1177.

Luhn, H. P. . The Automatic Creation of Literature Abstracts. , (2),
 159--165.

Maekawa, K., Yamazaki, M., Ogiso, T., Maruyama, T., Ogura, H., Kashino,
W., Koiso, H., Yamaguchi, M., Tanaka, M.,  Den, Y. . Balanced Corpus of
Contemporary Written Japanese. , ,  345--371.

Matsumoto, S., Asahara, M.,  Arita, S. . Japanese Clause Classification
Annotation on the 'Balanced Corpus of Contemporary Written Japanese'. In
Proceedings of the 13th Workshop on Asian Language Resources (ALR12),
 1--8.

Mikolov, T., Chen, K., Corrado, G.,  Dean, J. 2013a. Efficient
Estimation of Word Representations in Vector Space. In International
Conference on Learning Representations, **abs/1301.3781**.

Mikolov, T., Sutskever, I., Chen, K., Corrado, G.,  Dean, J. 2013b.
Distributed Representations of Words and Phrases and their
Compositionality. In Advances in Neural Information Processing Systems,
 3111--3119.

Mitchell, J., Lapata, M., Demberg, V.,  Keller, F. . Syntactic and
Semantic Factors in Processing Difficulty: An Integrated Measure. In
Proceedings of the 48th Annual Meeting of the Association for
Computational Linguistics,  196--206.

宮内拓也浅原正幸中川奈津子加藤祥 .
『現代日本語書き言葉均衡コーパス』への情報構造アノテーションとその分析. ,
,  19--33.

Patterson, C.  Drummer, J. . EyeTracking--Focus: Eyetracking During
Reading. Linguistischer Methodenworkshop (HU Berlin).

Pynte, J., New, B.,  Kennedy, A. . On-line Contextual Influences During
Reading Normal Text: A Multiple-regression Analysis. , ,  2172--2183.

Schakel, A. M. J.  Wilson, B. J. . Measuring Word Significance using
Distributed Representations of Words. , abs/1508.02297.

Sorensen, T., Hohenstein, S.,  Vasishth, S. . Bayesian Linear Mixed
Models Using Stan: A Tutorial for Psychologists, Linguists, and
Cognitive Scientists. , ,  175--200.

植田禎子飯田龍浅原正幸松本裕治徳永健伸 .
『現代日本語書き言葉均衡コーパス』に対する述語項構造・共参照関係アノテーション. ,
 205--214.

Vasishth, S.  Drenhaus, H. . Locality in German. , (1),  59--82.
:::

# 分析結果（詳細）

本節では，skip-gram の最大モデル (@xmath56)の結果の詳細について示す．
Rhat が収束判定指標で chain 数 3 以上ですべての値が 1.1 以下を収束とみ
なす．本研究では全ての設定で chain 数 4 とした． n_eff
が有効サンプル数， mean がサンプルの期待値（事後平均）， sdが MCMC
標準偏差（事後標準偏差）， se_mean が標準誤差で，MCMC のサンプルの分散を
n_eff で割った値の平方根を表す．

## 分析結果：FFT skip-gram

表\[tbl:fft\] に FFT skip-gram の最大モデル (@xmath60) の結果を示す．
FFT は文節内の最初の停留の注視時間を評価する． このため文節長の効果
(@xmath61)や文節間に空白を入れるか否かの効果 (@xmath62)
が確認されなかった． 呈示順(@xmath63，@xmath64，@xmath65，
\@xmath66，@xmath67)・レイアウト情報 (@xmath68，@xmath69，@xmath70)
に関して は， 行呈示順(@xmath71) で読み時間が短くなる傾向が，
文節呈示順(@xmath72) で読み時間が長くなる傾向が， 行内最右要素(@xmath73)
で読み時間が短くなる傾向が見られた．

係り受けの数が多いほど読み時間が短くなる傾向(@xmath74)がみられる．
ベクトルのノルム(@xmath75) については，ノルムが大きくなればなるほ
ど読み時間が長くなるという弱い効果がみられる．
隣接要素との類似度(@xmath76) や頻度 (@xmath77)につい
ては，値が大きいほど読み時間が短くなるという強い効果がみられる．

## FPT skip-gram

表\[tbl:fpt\] に FPT skip-gram の最大モデル (@xmath78) の結果を示す．
FPT
は文節内に初めて視線が停留し，その後文節から出るまでの総注視時間である．
出る方向は右方向でも左方向でも構わない． 文節長の効果
(@xmath79)は視線停留対象の面積に相当するために， 正方向に効果が出る．
また，レジビリティの観点である文節間に空白を入れるか否かの効果(@xmath80)ついては，
空白を入れたほうが読み時間が短くなることが確認された．
呈示順(@xmath81，@xmath82，@xmath83， \@xmath84，@xmath85)に関しては，
記事呈示順以外で実験が進むにつれて読み時間が短くなる．
記事呈示順は実験計画として4パターンのみ準備しており，
記事に対するランダム効果 (@xmath86) に吸収されたと考える．
以上の傾向は，RPT， TOTAL についても共通してみられる．

レイアウト情報 (@xmath87，@xmath88，@xmath89) は，
行内最左要素(@xmath90) と右から 2 番目の要素 (@xmath91)
で読み時間が長くなる傾向が見られた．
これは，注視点の復帰改行の移動の効果だと考える．

係り受けの数が多いほど読み時間が短くなる傾向(@xmath92)がみられる．
ベクトルのノルム(@xmath93) については，ノルムが大きくなればなるほ
ど読み時間が長くなるという強い効果がみられる．
隣接要素との類似度(@xmath94) や頻度 (@xmath95)につい
ては，値が大きいほど読み時間が短くなるという強い効果がみられる．

## RPT skip-gram

表\[tbl:rpt\] に RPT skip-gram の最大モデル (@xmath96) の結果を示す．
RPT
は文節内に初めて視線が停留し，その後文節の右側から出るまでの総注視時間である．
左側に抜ける場合は継続して合算する．

文節長・空白・呈示順については，FPT と同じ傾向であった．

レイアウト情報 (@xmath97，@xmath98，@xmath99) は，
行内最右要素(@xmath100) と右から 2 番目の要素 (@xmath101)
で読み時間が長くなる傾向が見られた． これは，RPT
の読み時間の定義から，最右要素や右から 2 番目の要素はこれ以上右につき
ぬけにくいためであろう．

係り受けの数が多いほど読み時間が短くなる傾向(@xmath102)がみられる．
ベクトルのノルム(@xmath103) については，ノルムが大きくなればなるほ
ど読み時間が長くなるという強い効果がみられる．
隣接要素との類似度(@xmath104) や頻度 (@xmath105)につい
ては，値が大きいほど読み時間が短くなるという強い効果がみられる．

## TOTAL skip-gram

表\[tbl:total\] に TOTAL skip-gram の最大モデル (@xmath106)
の結果を示す． TOTAL は文節内の総注視時間である．
文節長・空白・呈示順については，FPT と同じ傾向であった．

レイアウト情報 (@xmath107，@xmath108，@xmath109) は，FPT と同様に，
行内最左要素(@xmath110) と右から 2 番目の要素 (@xmath111)
で読み時間が長くなる傾向が見られた．

係り受けの数が多いほど読み時間が短くなる傾向(@xmath112)がみられる．
ベクトルのノルム(@xmath113) については，ノルムが大きくなればなるほ
ど読み時間が長くなるという強い効果がみられる．
隣接要素との類似度(@xmath114) や頻度 (@xmath115)につい
ては，値が大きいほど読み時間が短くなるという強い効果がみられる．

# Second Pass Time について

Second Pass Time
(SPT)は研究者によって，ゼロ読み時間（読み飛ばし）の扱いが異なり，査読などで議論が対立することが多く，本稿では
SPT の結果を除外した．

自己ペース読文法においては，実験協力者は必ずすべての文節を見るために読み飛ばしが
発生しない． 視線走査法の FFT, FPT, RPT, TOTAL
においては，ゼロ読み時間を考慮しないことが研究
コミュニティにおいて共有されている． SPT
はゼロ読み時間を扱う研究とゼロ読み時間を扱わない研究があり，BCCWJ-EyeTrack
では後者の扱いをとっている．

著者らが考える理由は三つある． 一つ目は TOTAL と FPT の関係である． SPT
においてゼロ読み時間をする場合，TOTAL においてゼロ読み時間の場合を除いて
\@xmath116 の値と SPT の値が完全従属する． 二つ目は対数正規分布
`lognormal` によりモデル化できる点である． 対数正規分布は定義域が
\@xmath117 であり，ゼロ読み時間を評価することができ ない．
しかしながら，正規分布と異なり，サンプリングの際に自然に負の時間を回避できるほか，
外れ値の影響が軽減されるというメリットがある．
三つ目は，本質的に2回目の読み時間がないということは欠損値であると考え，
モデル化する対象から外すことで， 0の値を割り当てるという overspecified
の問題を回避することができる．

Vasishth らは，ゼロ読み時間を扱うものを rereading time と呼び，
ゼロ読み時間を扱わないものを SPT として区別したうえで， SPT
を扱うべきとしている@xcite． さらに rereading time については UMASS の
eyedry[^13]など \@xmath118 としているものもある． Patterson らは
2016年の時点で "controversy over including 0 when no rereading"
\@xciteとし， この扱いについては，まだ議論が収束していない．

なお，SPT 分析結果としては，分散表現のノルムのみ（CBOW, skip-gram
とも）が効果として確認され，
それ以外の効果（単語頻度の幾何平均・隣接文節との類似度）は確認されなかった．

::: biography
:::

[^1]:

[^2]:

[^3]:

[^4]: 確率的言語モデル：@xmath3

[^5]: N-gram：@xmath4

[^6]: PCFG：@xmath5

[^7]: KL ダイバージェンス：@xmath6

[^8]: CBOW か skip-gram か
    以外のオプションは次の通り：`-size 300 -window 8 -negative 25 -hs 0 -sample 1e-4 -iter 15`

[^9]:

[^10]:

[^11]:

[^12]:

[^13]: http://blogs.umass.edu/eyelab/software/
