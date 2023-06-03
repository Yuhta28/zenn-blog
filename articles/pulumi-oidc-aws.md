---
title: "OIDC認証でPulumi CloudとAWSを連携してみた"
emoji: "🌎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["pulumi","aws","oidc"]
published: false
---

# 概要
2022年11月にPulumiはGitHubと連携することでCI/CDプラットフォームを自動構築してくれるPulumi DeploymentをPulumi Cloud上でリリースしました。
https://www.pulumi.com/blog/pulumi-deployments/

Pulumiで記述されたIaCソースコードが存在するGitHubリポジトリと連携することでGitHubにPRすることでPulumi Cloud上で`pulumi preview`(Terraformでいうところのplanコマンド)が実施され、マージすることで`pulumi up`(Terraformでいうところのapplyコマンド)が走り、GitHubフローにそったIaC開発が実現できるようになりました。

ただ当初はAWSのシークレットキーなどの秘文情報を変数化してスタックファイル内に渡すといった運用しかできませんでした。
```ts:IAMユーザーの認証情報を渡す
const stack = await RemoteWorkspace.createOrSelectStack({
    stackName,
    url: "https://github.com/pulumi/examples.git",
    branch: "refs/heads/master",
    projectPath: project,
}, {
    envVars: {
        AWS_REGION:            "us-west-2",
        AWS_ACCESS_KEY_ID:     process.env.AWS_ACCESS_KEY_ID ?? "",
        AWS_SECRET_ACCESS_KEY: { secret: process.env.AWS_SECRET_ACCESS_KEY ?? "" },
        AWS_SESSION_TOKEN:     { secret: process.env.AWS_SESSION_TOKEN ?? "" },
    },
});

const upRes = await stack.up({ onOutput: console.log });
console.log(`url: ${upRes.outputs.websiteUrl.value}`);
```

# 参考文献
https://openid.net/foundation/