# モデル: DML(Double Machine Learning)
## 概要
  - 名前の通り、2段階の機械学習モデルを用いてCATE(Conditional Average Treatment Effect)を推定する手法
    - CATE: 条件付き平均処置効果、HTE(Heterogeneous Treatment Effect)と似たものとして理解して良い
  - パラメトリックモデルや古典的統計学アプローチで対応できない場合、処置変数が連続値の場合も対応可能

## アイデア
やりたいことをシンプルに考えると、下記部分線形モデルを推定したいが、後述する問題が生じる
  - $Y = \hat{\theta}T + \hat{g}(X, W) + \hat{\varepsilon}$
    - Y: アウトカム
    - T: 処置
    - X: 共変量
    - W: 制御変数(X以外の共変量)
  - θ: 共変量を制御した状態での、処置にようるアウトカムの変化 = 因果効果

しかしこのアプローチでは2つのバイアスが生じることが知られている
  - 正則化バイアス
  - 過学習バイアス

まず、正則化バイアスを解決できる、Robinsonの手法が存在する
  - 概要
    - 2段階に分けて推定する
    - 1段階目: X,WからYを推定するモデルと、X,WからTを予測するモデルを作る
      - これは好きな機械学習モデルを用いることができる
    - 2段階目: 2つのモデルの残差同士を回帰して、θを求める
      - ここは原論文では線形モデル(OLS)想定だが、EconMLライブラリでは非線形も使用可能
  - 式
    - $Y = \theta \cdot T + g(X, W) + \varepsilon, \quad T = f(X, W) + \eta$
    - $Y - E[Y|X, W] = \theta(T - E[T|X, W]) + \varepsilon$

DMLでは、この手法を拡張している
- 違い
  - θ → θ(X)に
    - θ(X)はすなわちXで条件づけられた効果であり、CATE
- 式
  - $Y = \theta(X) \cdot T + g(X, W) + \varepsilon, \quad T = f(X, W) + \eta$
  - ただし、$E[\varepsilon | X, W] = 0, E[\eta | X, W] = 0, E[\varepsilon \cdot \eta | X, W] = 0$
  - $Y - E[Y | X, W] = \theta(X) \cdot (T - E[T | X, W]) + \varepsilon$
- 補足
  - さらに、cross-fittingを用いることで過学習バイアスにも対処している
  - 下記はDRLでありDMLとは異なるが、アイデアの理解促進のため掲載
    - ![image](https://scrapbox.io/files/658015d6850e1c002bbac45b.png)
    - [出典](https://speakerdeck.com/fullflu/cfmlmian-qiang-hui-number-7-econmlnishi-zhuang-sareteiruyi-zhi-chu-zhi-xiao-guo-notui-ding-shou-fa-noshao-jie-zai-kao?slide=37)

## 例
例1: 基本的なケース
- やりたいこと
  - メール広告による注文金額への効果が知りたい
  - 年齢によってメール広告効果が異なりそうなので、年齢ごとのCATEが知りたい
  - 変数の整理
    - Y: 顧客一人当たりの注文金額
    - T: メール広告配布
    - X: 年齢
    - W: 顧客の性別、会員年数、過去の注文金額デシル
  - わかること
    - メール広告配布による年齢別一人当たり注文金額への因果効果
    - 例: 10代には純減、30代には純増、60代には純減
　    - 2段階目に非線形モデルを用いる場合、単純な比例関係以外も表せる
  - 注意
    - 実際はRCTができそうな問題設定なので、無理してDML使う必要はない

例2: 処置変数が連続的なケース
  - やりたいこと
    - 運動量を増やしたときの体重減少効果が知りたい
    - 年齢によって体重減少効果は異なりそうなので、年齢ごとのCATEが知りたい
  - 変数の整理
    - Y: 体重
    - T: 運動量
    - X: 年齢
    - W: 対象者の属性(性別、筋肉量)
  - わかること
    - 運動量の変化による年齢別一人当たり体重減少効果
      - 例: 若いほうが運動量を1単位増やしたときの体重減少効果は大きい

## 他の手法との比較
  - シンプルな線形/部分線形モデル
    - 前述した二つのバイアスの影響を受ける
  - 傾向スコアマッチング
    - 共変量を「傾向スコア」という1つの指標に落とし込むことに問題がある
    - 傾向スコアのパラドックスの影響を受ける
  - Meta-learner
    - 共変量Xが連続値の場合対応できない
    - 制御変数Wが存在しない場合にしか対応できない
  - DRL(Doubly Robust Learner)
    - 共変量Xが連続値の場合対応できない
    - 処置確率が小さくなる特定のセグメント(X,W)がある場合分散が大きくなる
  - Causal Tree, Causal Forest
    - 決定木やランダムフォレストを用いたDML
    - 余談: 各所から怪しいという話を聞く…

## 参考
- [arXiv: Double/Debiased Machine Learning for Treatment and Causal Parameters](https://arxiv.org/abs/1608.00060)  
    - 提案論文
- [因果推論入門：What's Double Machine Learning?](https://www.youtube.com/watch?v=inUSZcjyBAQ&ab_channel=%E3%83%87%E3%83%BC%E3%82%BF%E7%A7%91%E5%AD%A6%E3%81%AE%E3%83%A1%E3%82%BD%E3%83%89%E3%83%AD%E3%82%B8%E3%83%BC)
    - 分かりやすい解説動画
- [Heterogeneous Treatment Effect Estimation using EconML: Double Machine Learning, Doubly Robust Learning, and Meta Learners - Speaker Deck](https://speakerdeck.com/fullflu/heterogeneous-treatment-effect-estimation-using-econml-double-machine-learning-doubly-robust-learning-and-meta-learners?slide=19)
  - 解説資料。シンプルな回帰, Meta-learner, DRL, DMLの比較が分かりやすい
- [CFML勉強会#7 EconMLに実装されている異質処置効果の推定手法の紹介・再考 - Speaker Deck](https://speakerdeck.com/fullflu/cfmlmian-qiang-hui-number-7-econmlnishi-zhuang-sareteiruyi-zhi-chu-zhi-xiao-guo-notui-ding-shou-fa-noshao-jie-zai-kao)
  - 上の資料と同じ著者がより詳細に解説した資料
- [機械学習で因果推論~Double Machine Learning~](https://zenn.dev/s1ok69oo/articles/4da9e3b01a0a93)
  - 理論が簡潔にまとめられている資料
- [たとえ木の中林の中森の中(Causal Tree, Causal Forest) #Python - Qiita](https://qiita.com/yellow_detteiu/items/e0915ef1042a6af49382#はじめに)
  - ポケモンを用いた事例説明
- [MoT TechTalk #14 タクシーアプリ『GO』の施策検証、因果推論が解決します - Speaker Deck](https://speakerdeck.com/mot_techtalk/mot-techtalk-14?slide=32)
  - 後半でCausal Forest適用事例を紹介している
- [因果フォレスト（Causal Forests）をPythonで実践的に学ぶ（その３）実践！因果フォレストを用いたデータ分析 – セールスアナリティクス](https://www.salesanalytics.co.jp/datascience/datascience187/)