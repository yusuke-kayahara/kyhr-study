# 論文: Geo-level Bayesian Hierarchical Media Mix Modeling
[Google Research](https://research.google/pubs/geo-level-bayesian-hierarchical-media-mix-modeling/)

## 概要
- 2017年、Google
- GBHMMM(geo-level Bayesian hierarchical media mix model)の提案論文
- 地域粒度のデータを用いたMMMによって、国粒度のモデルより精度を改善した

## 内容

### 1. Abstract
- 省略

### 2. Introduction
- 省略

### 3. Methodology
#### 3.1 Geo-level data
- 都市粒度のデータ: 有用だが、小さいエリアはノイズに左右されやすい
  - 特定の「エリア」粒度に集計することを推奨している
  
#### 3.2 Notation
- $g$: 特定の地理粒度
- $t$: 時間
- $m$: 広告媒体・チャネル
- $Y_{t,g}$: 特定の地理粒度gにおける時点tの応答変数
  - $y_{t,g} = Y_{t,g} / s_{g}$
  - 指標によっては対数変換なども検討する
- $X_{t,g,m}$: 特定の地理粒度gにおける時点tの広告媒体mのメディア変数
  - $x_{t,g,m} = X_{t,g,m} / s_{g}$
  - 基本的にimp数、モデルによってはコスト等を使用してもよい
  - Min-Max正規化することを推奨
- $Z_{t,c,g}$: 制御変数
  - $z_{t,c,g} = Z_{t,c,g} / s_{g}$
  - 製品の価格、プロモーション状況や、マクロ経済要因(市況)等
- エリア$g$の人口$s_{g}$で除した変数を下記のように小文字で定義する
- AdstockとHill関数を用いる
  - 表記はJin et al.（2017）に従う
  - 他論文と同様なので省略

#### 3.3 Model specification and estimation
- モデル式
  - $y_{y,g} = τ_{g} + \sum_{m=1}^{M} \beta_{m,g} \text{Hill}(x_{t,m,g}^{*}(α_{m},L);\mathcal{K}_{m},\mathcal{S}_{m}) + \sum_{c=1}^{C} \gamma_{c,g} z_{c,g} + \epsilon_{t,g}$
    - $\beta_{m,g} \sim \text{Normal}(\beta_{m},\eta_{m}^{2})$
    - $\gamma_{c,g} \sim \text{Normal}(\gamma_{c}, \xi_{c}^{2})$
    - $\tau_{g} \sim \text{Normal}(\tau,\kappa^{2})$
- 事前分布の置き方
  - 省略
- 階層構造の注意
  - 全ての変数を階層構造化する必要はない
  - 重要な変数にのみ階層構造を適用することで、推定が必要なパラメータ数を減らすことができる
  - CVやWAICを用いてモデル選択を行うことを推奨する
- さらにブランド粒度も加えて拡張できる
  - 省略

#### 3.4 Comparing GBHMMM with BMMM
- BMMM(Bayesian Media Mix Model)との比較
  - 省略

#### 3.5 Attribution metrics
- ROAS, mROASの算出方法
  - 基本的にJin et al.（2017）に従う
  - 省略

### 4 Results From Simulations
- シミュレーションデータで検証した結果、GBHMMMの方が精度が高く、推定幅も狭いことが示された
- 省略  

### 5 Real Data Case Study
- 実データでの検証
- 省略

### 6 Discussion
- 気になったところのみメモ
  - [Zhang and Vaver (2017)](https://research.google/pubs/introduction-to-the-aggregate-marketing-system-simulator/)でマーケティングシミュレータが提案されている？気になる

## 感想
- 割とシンプルなモデルで使いやすそう
