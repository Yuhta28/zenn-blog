---
title: "RDSのリザーブドインスタンスに気をつけよう"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS","RDS"]
published: false
---
# 概要
RDSのリザーブドインスタンス(RI)に関する注意点、もとい私がやらかした失敗談です。
EC2のRIとは微妙に仕様が異なっていて罠にハマりましたので戒めの意味を込めて個々に記します。
