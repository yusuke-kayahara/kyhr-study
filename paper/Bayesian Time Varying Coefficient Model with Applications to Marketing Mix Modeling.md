# 論文: Bayesian Time Varying Coefficient Model with Applications to Marketing Mix Modeling
[Bayesian Time Varying Coefficient Model with Applications to Marketing Mix Modeling](https://arxiv.org/abs/2106.03322)

## 概要
  - Kernel-based Time-varying Regression(KTR)という手法を提案した
      - タイトル通り、MMM文脈の論文である
  - Orbitで実装されている
  - 一言で言うと「動的トレンド・動的係数をサポートしつつ計算量を抑えた時系列モデル」

## 従来手法と比較した強み
  - Prophet等のシンプルな時系列と比較
    - 係数の変動をモデリングできる
      - 週次・年次トレンドが変わる
      - 共変量の与える影響が変わる
        - ![image](https://github.com/yusuke-kayahara/kyhr-study/assets/89958903/c1814993-66a9-4da5-8116-e86cff13409b)

  - 動的係数モデルと比較
    - 計算量が少ない
      - 係数を潜在変数の加重和で表現することで、パラメータ数の削減に成功した
      - またMCMCではなくSVI(確率的変分推論)を用いて計算量の削減を図っている
  - 精度がいい(いつもの話半分で聞くやつ)
    - 論文ではシミュレーションデータとUberEatsの実データで比較
      - シミュレーションデータではBSTS等より高い精度で係数を推定できている
      - 実データでは新規獲得ユーザー数をProphetやSARIMAより高い精度で推定できている

## どんなときに使えそうか？
  - Prophetでは物足りない時
    - 特に係数が変動していそうで、かつ変動をシンプルにモデリングするのが難しいとき
    - 例：売上の推移をいくつかの販促施策で説明する
      - 販促施策の効果は時点によって異なるが、いつ異なるか明らかでない
      - もしくは異なるタイミングは明らかだが、そのタイミングが多くProphet+共変量の工夫で対応できる範囲を超えている

## 重要ポイント
  - 係数の推定フロー
    - ![image](https://github.com/yusuke-kayahara/kyhr-study/assets/89958903/1a63bd5c-7512-4071-8b33-75e7f7754e11)
  - 係数推定~トレンド項と季節性項の推定
    - 前提
      - tは時点
      - βは回帰係数、Pは回帰子の数、pは特定の回帰子の番号
      - bは潜在変数、Jは潜在変数の数、jは特定の潜在変数の番号
    - 係数
      - $\beta_{t,p} = \sum_{j=1}^{J} w_j(t)b_{j,p}$
      - $w_j(t) = k(t, l_j) \sum_{l=1}^{J} k(t, l)$
          - 係数β_t_pは潜在変数b_j_pの加重和
            - 近いbほど大きいウェイトを持つ
          - ウェイトw_j(t)はカーネルを用いる
    - トレンド
      - $B_{lev} = K_{lev}b_{lev}$
      - $l_{t} = B_{t,lev}$
        - トレンドはつまり動的な切片と考えられる
          - Xt,levが全て1のベクトルで省略されている、という考え方
    - 季節性
      - $B_{seas} = K_{seas}b_{seas}$
      - $s_t = X_{t,seas}B^T_{t,seas}$
        - Xt,seasはフーリエ級数
          - これはProphetと同じ
        - 季節性も動的である(!)
          - これもProphetとの違い
    - 潜在変数
      - $b_{j,lev} \sim \text{Laplace}(b_{-1,lev}, \sigma_{lev})$
      - $b_{j,seas} \sim \text{Laplace}(b_{-1,seas}, \sigma_{seas})$
        - トレンドや季節性の潜在変数はベイズの枠組みで、1つ前(j-1)の潜在変数をパラメータとして持つラプラス分布で推定
        - 変化点検出をLaplace分布で行うのはProphetと一緒だ！
      - $\mu_{reg} \sim \mathcal{N}(\mu_{pool}, \sigma_{pool}^2)$
      - $b_{reg} \sim \mathcal{N}^+(\mu_{reg}, \sigma_{reg}^2)$
        - 回帰子の潜在変数推定は2段階構え
          - 事前分布からチャネル全体のμ_regを推定
          - μ_regからそれぞれの潜在変数bを推定
          - N+は正規分布を0以上で切ったものを表すらしい、便利～
      - それぞれの分布の違い
        - トレンドと季節性：Laplace分布で「基本は変わらないが稀に変わる」を検出する
          - Laplace分布は正規分布よりμの発生率が高いイメージ
  - カーネルの中身（まだよくわかってない）
    - トレンド&季節性
      - $\text{when } t_i \leq t \leq t_{i+1} \text{ and } j \in (i, i+1),$
      - $k_{lev}(t, j) = 1 - \frac{|t - t_j|}{t_{i+1} - t_i}$
        - 三角カーネルぽいやつ
    - 回帰子
      - $k_{reg}(t, l; j) = \exp\left(-\frac{(t - l_j)^2}{2\sigma^2}\right)$
      - ガウシアンカーネル
      - ρはカーネルの幅を決定するスケールパラメータ
    - これ以外にもいろんなカーネル使えて柔軟だよよ～って論文にはあった

感想・メモ
  - bを潜在変数と呼ぶのが分かりづらい
    - bは係数の変化点(スプライン回帰のノット)であると考えると分かりやすい
  - 相当データポイントが多いデータじゃないとワークしなさそう？
    - 実データで試してみたい

## リンク
[Kernel-based Time-varying Regression - Part I — orbit 1.1.4.2 documentation ](https://orbit-ml.readthedocs.io/en/stable/tutorials/ktr1.html)
- Orbitのチュートリアル かなり分かりやすい

[Rでベイジアン動的線形モデルを学ぶ(1)：なぜ「動的」モデルなのか - 渋谷駅前で働くデータサイエンティストのブログ](https://tjo.hatenablog.com/entry/2014/07/25/190717)
- そもそも動的線形モデルって？が掴みやすい
