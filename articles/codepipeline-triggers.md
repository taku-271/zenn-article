---
title: "[AWS CDK]Code Pipeline V2を使用したトリガーフィルター"
emoji: "😨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [aws, cdk, codepipeline, github]
published: false
publication_name: "uniformnext"
---

# はじめに
こんにちは、たくみです。
最近は新しい車が納車されて、暇があったらドライブに行っています。

Code兄弟を使用した、CI/CDパイプラインを含むインフラを構築することが多くなってきました。
大体のアプリケーションはモノレポを採用しており、インフラやアプリケーションが同じリポジトリに存在するケースが多くあります。

しかし、モノレポ環境でCode Pipeline環境を構築すると、インフラのコードを修正しただけでもアプリケーションのデプロイが走ってしまいます。
また、Code Pipelineのトリガーにはパスフィルターが実装されていますが、CDK（L2コンストラクト）からはそれが設定できません。

そこで、コストや運用優秀性の観点からどうにかパスフィルターがcdk上で実装できないかを試行錯誤しました。

# 解決方法
結論から言うと、CDKで作成したL2のCode Pipelineコンストラクタに対して、CloudFormationプロパティを上書きするようにすれば解決できます。
:::message
以下のコードは**GitHub**からCode Connectionsを使用してCI/CDを回す例となっています。
:::

```typescript
const sourceAction = new cdk.aws_codepipeline_actions.CodeStarConnectionsSourceAction({
    ...
});

const pipeline = new cdk.aws_codepipeline.Pipeline(this, "CodePipeline", {
    pipelineName: "CodePipeline",
});

const cfnPipeline = pipeline.node.defaultChild as cdk.aws_codepipeline.CfnPipeline;

cfnPipeline.addPropertyOverride("Triggers", [
    {
        GitConfiguration: {
            Push: [
                {
                    FilePaths: {
                        /** 除外したいパスを書く **/
                        Excludes: ["packages/infra/**"]
                        /** 含めたいパスを書く **/
                        Includes: ["packages/app/**"]
                    }
                }
            ],
            SourceActionName: sourceAction.actionProperties.actionName,
        },
        ProviderType: cdk.aws_codepipeline.ProviderType.CODE_STAR_SOURCE_CONNECTION,
    }
]);
```

# `defaultChild`
さてこれは、CDKの`defaultChild`というプロパティを使用することで解決しています。
これは、L2コンストラクトに定義されていないそのL1コンストラクトにアクセスできるプロパティのことです。

つまり、L2コンストラクトで作成されるリソースのCloudFormationプロパティにアクセスすることができるということです。

先ほどの例で言うと、L2コンストラクトのインスタンスである`pipeline`に対して、`Triggers`と言うCloudFormationプロパティを上書きしていることになります。

```typescript
/** L2コンストラクトである`pipeline`インスタンスを作成 **/
const pipeline = new cdk.aws_codepipeline.Pipeline(this, "CodePipeline", {
    pipelineName: "CodePipeline",
});

/** defaultChildを用いて、CloudFormationプロパティを持った変数を作成 **/
const cfnPipeline = pipeline.node.defaultChild as cdk.aws_codepipeline.CfnPipeline;

/** .addPropertyOverrideを用いて、`Triggers`プロパティを上書き **/
cfnPipeline.addPropertyOverride("Triggers", [
    {
        ...
```

これを使用すれば、L2コンストラクトからでは手が届かないパラメーターも編集することができます。

# 最後に
さて、今回はCode Pipelineのトリガー設定と、おまけとして`defaultChild`の解説を行いました。
やはり、CDKは奥深い。どれだけ触っても底が見えなさそうでドキドキします。
便利すぎて作成者様方には感謝しかないです。ありがとうございます。
私もいつか、何かしらの方法で貢献できたらと思っています。

では、良いCDKライフを！

