# Levitt (1997) を再現してみる

Levitt (1997) は操作変数法を用いて警察の増強が犯罪件数に与える効果を推定した。

市長選・知事選のタイミングに警官が増強されることに着目し、その年の市長選の有無を操作変数に利用した。

[Levitt, S. D. (1997). Using electoral cycles in police hiring to estimate the effect of policeon crime.](http://home.cerge-ei.cz/gebicka/files/IV_Simultaneity.pdf)


McCraryが再現研究を行っており、データも公開している。このデータを使ってPythonで再現を試みる

[Replication of Steven Levitt (AER, 1997), Justin McCrary 9/01](https://eml.berkeley.edu/replications/mccrary/index.html)


## 結果

- うまく再現できなかった
- SASのコードをそのままlinearmodelsで再現しようとするとランク落ちでエラーになる
- いくつか変数を落として再現しようとしても係数のプラスマイナスが合わなかったりする
- first stageだけはうまく再現できるがsecond stageが再現できない
