---
title: D - K-th Nearest【Atcorder】【WriteUp】
tags:
  - C++
  - AtCoder
  - 初心者
  - 競技プログラミング
  - writeup
private: true
updated_at: '2024-08-21T22:18:01+09:00'
id: 5019c705535f862d4040
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに
茶色コーダーです。しばらく参加してませんが、緑目指して頑張ってみようと思います。

# 問題

[D - K-th Nearest](https://atcoder.jp/contests/abc364/tasks/abc364_d)

### 問題文
数直線上に $N+Q$ 個の点 $A_1,\ldots,A_N,B_1,\ldots,B_Q$ があり、点 $A_i$ の座標は $a_i$ 、点 $B_j$ の座標は $b_j$ です。
$j=1,2,\ldots,Q$ それぞれについて、以下の問題に答えてください。
- 点 $A_1,A_2,\ldots,A_N$ のうち点 $B_j$ との距離が $k_j$ 番目に近い点を $X$ としたとき、点 $X$ と点 $B_j$ との距離を求めよ。より厳密には、点 $A_i$ と点 $B_j$ との距離を $d_i$ として、$(d_1,d_2,\ldots,d_N)$ を昇順に並び替えてできる列を $(d_1^{\prime},d_2^{\prime},\ldots,d_N^′)$ としたとき、$d_{kj}^{\prime}$ を求めよ。

### 制約
- $1≤N,Q≤10^5$
- $−10^8≤a_i,b_j≤10^8$
- $1≤k_j≤N$
- 入力は全て整数

# 解答と解説
難しくて解説みながらやったので、コードが解説と(~~同じ~~)似ています。

### 解説
[「答えを決め打つ」タイプの二分探索](https://betrue12.hateblo.jp/entry/2019/05/11/013403)を用いて解いていきます。
「答えを決め打つ」タイプの二分探索とは記事によると、
1. 「○○という条件を満たす $X$ の最小値を求めよ」という問題において、
2. 「ある値 $X$ が与えられたとき、○○を満たすことはできるか？」という判定問題を考え、
3. その判定問題を繰り返し解くことでYesになる $X$ と $N_o$ になる $X$ の間の境界を特定し、元の問題の答えを求める解法。
です。

今回の問題では $f(x) = (点A_1,\ldots,A_NのうちB_jとの距離がx以下の点がk以上か)$ という判定問題を考え、条件を満たす最小の $k$ を求めるという問題です。
ソートした $A$ に対して上記の判定問題を用いて二分探索を行います。
少し難しいので簡単な例を用いてなぜ解けるかを解説します。

- $A =  \lbrace -3,-1,5,6 \rbrace $
- $B_j = -2$
- $k = 3$

で考えてみます。

$f(x)$ をグラフにすると下のようになります。みてわかる通り単調増加です。
![D - K-th Nearest.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3310678/5a08fc17-a6b6-444a-1b26-b50e2c402415.png)
グラフを見てわかる通り $x=7$ のときが条件を満たす最小の $x$ です。
この $f(x)$ が $k$ 以上か以下であるかという判定を用いて二分探索を行うと効率よく探索を行えます。

$f(x)$ は$B_j-x$以上$B_j+x$以下の要素がいくつあるかという問題に置き換えることができます。実装はこの置き換えた問題を用いて行います。

### 計算量

- 計算量は最初に $A$ をソートするので $O(N\log{N})$
- $Q$ 個のクエリに対して
  - $f(x)$ を用いた二分探索が $O(\log{A})$ ( $A$ は $A$ の要素が取りうる要素の範囲。つまり $2\times 10^8$ )
  - $f(x)$ の中で要素を数える際に二分探索を行うため $O(\log{N})$

合わせると $O(N\log{N} + Q\log{A}\log{N})$ 

### コード
通常の二分探索では、目的の値を見つけたらその時点で終了かつ境界の遷移はその時点で探索している場所の一つ隣であるため、存在しない場合は $left>=right$ の状態まで遷移します。
今回は境界をみつけるのが目的($left+1=right$ の状態が目的)なのでそれを条件にし、境界の遷移は現在探索している値を代入していきます。
```c++
#include <bits/stdc++.h>

using namespace std;

int main() {
  int N,Q; cin>>N>>Q;
  vector<int> A(N);
  
  for(int i=0;i<N;i++) {
    cin >> A[i];
  }
  
  sort(A.begin(),A.end());
  
  for(int i=0;i<Q;i++) {
    int b,k;cin>>b>>k;
    
    auto f = [&](int x) {
      auto ub = upper_bound(A.begin(),A.end(),b+x);
      auto lb = lower_bound(A.begin(),A.end(),b-x);
      
      return k <= ub-lb;
    };
    
    int left = -1;
    int right = 2e8 + 1;
    int mid = (left+right)/2;
    while(left+1 < right) {
      if(f(mid)) {
        right = mid;
      }else{
        left = mid;
      }
      mid = (left+right)/2;
    }
    cout << right << endl;
  }
}
```

# さいごに
二分探索系の問題いっぱいやってみようと思います！
