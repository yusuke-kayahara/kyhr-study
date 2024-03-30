# 論文: Estimating True Post-Click Conversion via Group-stratified Counterfactual Inference

## 概要
  - adKDD2021、清華大学 & 浙江大学 & Huawei
  - 広告主にとって、CTRやCVRベースの入札額の検討は望ましくない、というのが出発点？
    - 「広告出さなくても買う人」を除外した純増CVRで考えよう
  - 純増CVRをTCVR (true post-click conversion rate) と名付けて、推定の方法を検討
    - CVのうち純増CVは25%以下であることが知られている(らしい)
    - ソース: [Estimating the Causal Impact of Recommendation Systems from Observational Data](https://arxiv.org/abs/1510.05569)
  - Group-stratified Counterfactual Inference (GCI) algorithmという手法を提案
## 内容
### Introduction
  - 整理
    - ![image](https://scrapbox.io/files/64d7172baefa97001c1bb33f.png)
    - 前提
      - T: 広告にexposeされたか否か
      - C: 広告をclickしたか否か
      - Y: CVしたか否か
    - 群の説明
      - A: どんなケースでもCVしない
      - B: 広告はクリックしないが、広告を当てても当てなくてもCVする
      - C: 広告を当てるとクリックするが、どんなケースでもCVしない
      - D: 広告を当てるとクリックするし、広告を当てた場合のみCVする
      - E: 広告を当てるクリックするが、広告を当てても当てなくてもCVする
        - "free-rider "
    - ポイント
      - 通常はC,D,Eの割合がCTR, D,Eの割合がCVRとして評価されてるけどフェアじゃないよね
      - Dの割合(free-riderを除いた割合 )を真のCVRとしてTCVRと名付ける
    - メモ
      - CVに関してはいつもの反実仮想だか、clickに関しても同じフレームワークで整理するの賢い

## Problem and methodology
  - DAG
    - ![image](https://scrapbox.io/files/64d73917f8f678001b265db7.png)
  - 問題点
    - RCTにできない
      - T(広告を当てられたか )はコントロール不能
      - TはC, Yと独立でない
    - バイアスなくTCVRを見積もるのが難しい
      - $\text{TCVR} = p(Y = 1|X,T = 1) - p(Y = 1|X,T = 0)$
      - つまり「広告見た&CVした 」割合から、反実仮想で導いた「広告見なかった&CVした 」の割合を引いたもの
  - 手法
    - 動機: バイアスなくTCVRを見積もるにはどうすればいいか？
    - ポイント
      - ![image](https://scrapbox.io/files/64d73d4e4e150a001c18918c.png)
      - いくつかの仮定を置くとこれが導ける
  - アップリフトモデリングとの比較
    - 本手法のほうが優れているという主張
    - 表
      - $\text{CTR}(X = x) = P(C|x) + p(D|x) + p(E|x)$
      - $\text{CVR}(X = x) = p(D|x) + p(E|x)$
      - $\text{TCVR}(X = x) = p(D|x)$


    - 前提: アップリフトモデリングはCACE(Conditional Average Causal Effect )を推定してる
    - アップリフトモデリングには"free-rider "の考え方が足りない
      - free-rider metricというものを考えましたよ〜

## ESTIMATION
  - GCI (Group-stratified Countnrfactual Inference )をやるよ〜
  - 前提
    - ※上のDAG参照
    - ユーザー情報Xはimp, click, cv全てに影響を与える
    - 加えて、取得できない潜在変数Uもある
      - 例: 他媒体でレコメンドを受けたとか、ギフト受注かどうかとか
  - GCIの説明
    - step
      - 1.特徴量から各ユーザーのCTR, CVRを予測できるモデルを作る
      - 2.それを使って計算してTCVRを得る
    - モデルの解説
      - ![image](https://scrapbox.io/files/64d74404b31ead001c876c7e.png)
      - $\text{CTR}(X = x) = P(C|x) + p(D|x) + p(E|x)$
      - 流れ
        - Embedding Layer
          - ユーザー特徴量とアイテム特徴量をEmbedding
        - Embedding Concatenation
          - Embeddingをconcat
        - Fully Connected Layer
          - 多層パーセプトロンで全結合？
      - 不明点
        - t=1とt=0でclassifier分けてる…？
        - yの話してるけどcはどこ行った…？

## 感想
    - 課題の分解が丁寧で感動した
      - 群に分けて整理
      - 仮定を置いてシンプルな計算式に落とす
      - 既存手法で考えられていなかった部分の明確化

## 参考
[Estimating True Post-Click Conversion via Group-stratified Counterfactual Inference | AdKDD 2021](https://www.adkdd.org/Papers/Estimating-True-Post-Click-Conversion-via-Group-stratified-Counterfactual-Inference/2021)