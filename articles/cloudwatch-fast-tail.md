---
title: "CloudWatch Logsをいい感じにtailしてくれるcwを使ってみた"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aws","cloudwatch","cw"]
published: false
---
# 概要
AWS CLIにはCloudWatch Logsのログをtailしてくれるコマンドがあります。
` aws logs tail <ロググループ名>`