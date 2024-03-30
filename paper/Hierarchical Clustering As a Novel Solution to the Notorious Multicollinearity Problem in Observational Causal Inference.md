# 論文: Hierarchical Clustering As a Novel Solution to the Notorious Multicollinearity Problem in Observational Causal Inference
## 概要
  - KDD2023の論文漁ってたら見つけた
  - AirbnbのMMM紹介
    - 階層クラスタリングを使っているのが珍しポイント
    - DMA(Designed Marketing Area)粒度のデータをクラスタ粒度に直すことで、いいかんじになったよ～

## 内容

### 1. Introduction
  - MMMの課題である多重共線性と交絡因子に、階層クラスタリングを使ったアイデアを適用した論文
    - 地域ごとに広告のimpに相関があることを利用する
    - 相関パターンに基づいて、クラスタリングを用い地域をグループ化する
### 2. Bayesian structural model formulation
  - MMMの構造
    - 前提
      - gはエリア、Gはエリアの数
      - tは時点、Tは時点の数、粒度は週次
      - kはメディア、Kはメディアの数
      - xはimp数
      - yは効果、売上かログ変換した売上を用いる
    - 全体像
      - $y_{g,t} = \mu^{t} + \text{seasonality}_{g,t} + \alpha_{z_{g,t}} + \sum_{k=1}^{K} \beta_{k} \text{AdStock}(x_{g,t,k}) + \varepsilon_{g,t}$
        - μはメディアの平均効果
        - seasonalityは季節効果
        - Zは共変量との相関係数、売上の自然成長分を示す
    - 残存効果
      - Bayesian Methods for Media Mix Modeling with Carryover and Shape Effects で提案されたものを採用
      - $\text{AdStock}_{k,g} = \left( \frac{\sum_{l=0}^{L} \tau_{k}^{l} (1 - \theta_{k})^{2} x_{t-l,m} }{\sum_{l=0}^{L} \tau_{k}^{l} (1 - \theta_{k})^{2}} \right)^{\rho}$
        - ρは0~1でスケールを調整するパラメータ
        - τは0~1で残存率を表すパラメータ
        - Θは関数の中心を決定するパラメータ
  - 交絡因子の考慮
    - Google検索クエリのボリュームをZとしてモデルに組み込むことで、市場自体の成長/衰退を捉える

### 3. Data properties
  - 前処理
    - 目的変数と共変量を正規化する
    - impを共通のトレンドと季節性と残差に分解する
      - 次項で残差の相関に注目するため
      - この残差を"imp残差"と呼ぶことにする
  - データの確認
    - 前項で取り出した残差について、地域間で相関係数を調査すると、トレンドや季節性を除いているにも関わらず相関があることが分かる
    - チャンネルごとの相関
      - ![image](https://scrapbox.io/files/64edf9eae89ab8002111fe90.png)
    - チャンネルを揃え、エリアごとの相関
      - Xth, Yth ventileって何のことだ…？
      - ![image](https://scrapbox.io/files/64edfa1082f327001cd2aa12.png)

### 4. Hierarchical clustering as a novel sotution to multicollinearity
  - 距離
    - エリアiとエリアjの距離Distance_i_jは下記式で求める
      - $\text{Distance}_{j,k} = 1 - \text{Correlation}(X_{i,k}, X_{j,k})$
      - $\text{Distance}_{i,j} = \sqrt{\sum_{k=1}^{K} \text{Distance}_{i,j,k}^{2}}$
  - クラスタリング
    - クラスタリングの結果は下記
      - ![image](https://scrapbox.io/files/64edfc7c901c51001b2386f7.png)
      - 最終的にcut-off distanceは1.5に設定し、42のクラスタに分かれた

### 5. Results
  - クラスタリングの結果
    - 同じクラスタに振り分けられたエリアは、やはりimp残差の相関が強い
    - ![image](https://scrapbox.io/files/64edffa5c6a7aa001b8d8432.png)
    - ![image](https://scrapbox.io/files/64edffaf94f906001cb4acbe.png)

  - クラスタリングが多重共線性を軽減する
    - エリアをクラスタに振り分けることで、チャンネル内のimp残差の相関が8%~43%減少した
    - ![image](https://scrapbox.io/files/64edffb9aee014001b0f3d74.png)
  - シンプルな回帰の結果
    - エリア(DMA)粒度では多重共線性により正しく係数が推定できない（符号が反転している）メディアがある
    - クラスタ粒度ではそこが補正され、直感に沿う係数が推定できている
    - ![image](https://scrapbox.io/files/64edfffbb2bb8e001b7680e6.png)
  - MMM(ベイズモデル)の結果
    - 直感に沿うモデルが構築できた
      - ![image](https://scrapbox.io/files/64ee008c73ec2d001bba2ede.png) 
    - メディアごとのパラメータも推定できた
      - 上部: β (アップリフト効果)
      - 下部: τ (残存率)
      - ![image](https://scrapbox.io/files/64ee00e0bf9d70001b19085d.png)

### 6. Discussion of broader applications
  - 略

### 7. Conclusion
  - 略

## 感想
  - アイデアとしては面白いし、パラメータの符号が期待に沿う結果になったのもなるほど感ある
  - 一方で、このアイデアを採用することによる定量的なインパクトが欲しかった感もある
    - 「直感・仮説に沿うモデルになったね」で終わっちゃった感
    - これはそもそもMMMの抱える根深い問題だが…

## リンク
- [PDF](https://drive.google.com/file/d/1HQekrFF1vNNrO7TF4aX43850-Dc6UHgo/view)
- [Slides](https://drive.google.com/file/d/1r0xXFFflSVDFNtGEypUGh845CeycF51q/view)
- [ChatGPTの要約](https://chat.openai.com/share/02ac24a2-e2c1-4175-a0a5-d18ba6c97c03)
