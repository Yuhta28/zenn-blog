---
title: "OpenTelemetryに触れてみた"
emoji: "🔭"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["監視","opentelemetry","初心者"]
published: false
---

# 概要
オライリーから発売された「オブザーバビリティ・エンジニアリング[^1]」は非ベンダー依存を目指したアプリケーションモニタリングの実装方法としてOpenTelemetryを紹介しています。
監視基盤を実装する場合、Datadog[^2]やNew Relic[^3]などの監視SaaSを利用するかAWSやAzureが提供しているマネージドな監視サービスを利用すると思います。
直近の業務で使う機会はなさそうですが、アプリケーションモニタリングを収集するためのAPI、ライブラリ、エージェントを提供するOpenTelemetryという技術に興味が出てきましたので、概要やデモなどを紹介します。
またOpenTelemetryのドキュメントは開発者向けと運用者向けでスタートガイドが明確に分けられていました。
https://opentelemetry.io/docs/getting-started/
私は運用側のエンジニアなので運用視点でOpenTelemetryについて書き記します。(開発者は上のリンクからDevを参照してみてください)

[^1]: https://www.oreilly.co.jp/books/9784814400126/
[^2]: https://www.datadoghq.com/ja/
[^3]: https://newrelic.com/jp

# OpenTelemetryとは
OpenTelemetryとは分散トレーシングやメトリクスなどのオブザーバビリティデータを収集、処理、エクスポートするためのオープンソースプロジェクトになります。
https://opentelemetry.io/
最初モニタリングやオブザーバビリティのオープンソースコミュニティはOpenTracing[^4]とOpenCensus[^5]というプロジェクトを立ち上げました。この2つのプロジェクトはテレメトリーデータを収集してリアルタイムに好きなプログラミング言語でバックエンドに転送できることを目的にライブラリを提供してきました。両グループは2019年にCNCF[^6]傘下のOpenTelemetryプロジェクトとして統合されました。

:::message
オブザーバビリティ・エンジニアリングでは、アプリケーションから送られるログ、メトリクスなどのデータをテレメトリーとよんでいます。特定のリクエストからの処理に対してコードがどう影響しているのか記録し、システムの動作やユーザー体験を可視化、最適化することを目的としています。
:::

# OpenTelemetryコンポーネント
OpenTelemetryのコンポーネントについていくつか紹介します。
- コレクター
    - テレメトリーデータを受信、処理、エクスポートできる非ベンダー依存のプロキシ(サイドカー)です。PrometheusやJaegerなどのデータを受け取って複数のバックエンド先にデータを送信できます。
- スパン
    - 特定の操作の始まりから終わりまでをまとめた作業単位。ユーザーが会員サイトにログインする場合、ログイン処理が完了するまでの一連の流れをトレースすることで、ログイン失敗時にどこでエラーが起きたのか特定しやすくなります。
- トレーサー
    - サービス内のリクエストなど、実行した操作で何が起こっているのか詳細な情報を含むスパンを生成します。
- コンテキスト伝搬
    - 分散トレーシングを可能にする最も重要なコンセプトです。関連したスパン同士を紐づけ、トレースを組み立てます。

[^4]:https://opentracing.io/
[^5]:https://opencensus.io/
[^6]: https://www.cncf.io/

# テレメトリーの取得
OpenTelemetryは多くの言語で書かれたコードの計装をサポートしています。さらにそれらの言語は直接アプリケーションコードに手を加える必要がなく、自動計装を活用すれば短時間でテレメトリーの収集が可能になります。
2023年4月現在のサポート言語は以下のとおり[^7]です。

| Language | Traces | Metrics | Logs |
| :--- | :---: | :---: | ---: |
| C++ | Stable | Stable | 検証中 |
| C#/.NET | Stable | Stable | 検証中 |
| Erlang/Elixir | Stable | 検証中 | 検証中 |
| Go | Stable | ベータ版 | 未実装 |
| Java | Stable | Stable | 検証中 |
| JavaScript/TypeScript | Stable | Stable | 開発版 |
| PHP | ベータ版 | ベータ版 | アルファ版 |
| Python | Stable | Stable | 検証中 |
| Ruby | Stable | 未実装 | 未実装 |
| Rust | ベータ版 | アルファ版 | 未実装 |
| Swift | Stable | 検証中 | 開発中 |

ログに関してはまだどの言語も開発途中といったところのようです。
ここでは試しにTypeScriptによるトレーシングハンズオンを体験してみます。
https://opentelemetry.io/docs/instrumentation/js/getting-started/nodejs/

## TypeScriptハンズオン
まずはトレーシング対象となる簡易的なアプリケーションを用意します。

```shell-session: expressのインストール
$ npm install --save-dev typescript \
  ts-node \
  @types/node \
  express \
  @types/express \

$ ./node_modules/.bin/tsc --init
```

```ts:app.ts
import express, { Express } from "express";

const PORT: number = parseInt(process.env.PORT || "8080");
const app: Express = express();

app.get("/", (req, res) => {
  res.send("Hello World");
});

app.listen(PORT, () => {
  console.log(`Listening for requests on http://localhost:${PORT}`);
});
```

```shell-session:アプリ実行
$ ./node_modules/.bin/ts-node ./app.ts
Listening for requests on http://localhost:8080
```

![](/images/what-is-opentelemetry/image1.png =1000x)
*Hello World*

アプリケーションが実行できましたらトレーシングコードを実装します。

```shell-session:自動実装インストール
$ npm install --save-dev @opentelemetry/sdk-node \
  @opentelemetry/auto-instrumentations-node
```

```ts:tracing.ts
import { NodeSDK } from '@opentelemetry/sdk-node';
import { ConsoleSpanExporter } from '@opentelemetry/sdk-trace-node';
import { getNodeAutoInstrumentations } from '@opentelemetry/auto-instrumentations-node';

const sdk = new NodeSDK({
  traceExporter: new ConsoleSpanExporter(),
  instrumentations: [getNodeAutoInstrumentations()]
});

sdk
  .start()
```

```shell-session:アプリ再実行
$ ./node_modules/.bin/ts-node --require ./tracing.ts app.ts
Listening for requests on http://localhost:8080
```

何回か`http://localhost:8080`にアクセスしますとコンソール上にトレーシング情報が出力されます。

```powershell:トレーシング情報
{
  traceId: '78aaa7b5db4ab17ad10668e53d3aea6e',
  parentId: 'bcd528ec1efd529f',
  traceState: undefined,
  name: 'middleware - query',
  id: 'c2db165d1b7289c9',
  kind: 0,
  timestamp: 1680404455241000,
  duration: 37,
  attributes: {
    'http.route': '/',
    'express.name': 'query',
    'express.type': 'middleware'
  },
  status: { code: 0 },
  events: [],
  links: []
}
{
  traceId: '78aaa7b5db4ab17ad10668e53d3aea6e',
  parentId: 'bcd528ec1efd529f',
  traceState: undefined,
  name: 'middleware - expressInit',
  id: '41131b313aea88c1',
  kind: 0,
  timestamp: 1680404455241000,
  duration: 54,
  attributes: {
    'http.route': '/',
    'express.name': 'expressInit',
    'express.type': 'middleware'
  },
  status: { code: 0 },
  events: [],
  links: []
}
{
  traceId: '78aaa7b5db4ab17ad10668e53d3aea6e',
  parentId: 'bcd528ec1efd529f',
  traceState: undefined,
  name: 'request handler - /',
  id: '001fa0f7ed43abaf',
  kind: 0,
  timestamp: 1680404455242000,
  duration: 5,
  attributes: {
    'http.route': '/',
    'express.name': '/',
    'express.type': 'request_handler'
  },
  status: { code: 0 },
  events: [],
  links: []
}
{
  traceId: '78aaa7b5db4ab17ad10668e53d3aea6e',
  parentId: undefined,
  traceState: undefined,
  name: 'GET /',
  id: 'bcd528ec1efd529f',
  kind: 1,
  timestamp: 1680404455241000,
  duration: 1706,
  attributes: {
    'http.url': 'http://localhost:8080/',
    'http.host': 'localhost:8080',
    'net.host.name': 'localhost',
    'http.method': 'GET',
    'http.scheme': 'http',
    'http.target': '/',
    'http.user_agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/111.0.0.0 Safari/537.36 Edg/111.0.1661.62',
    'http.flavor': '1.1',
    'net.transport': 'ip_tcp',
    'net.host.ip': '::1',
    'net.host.port': 8080,
    'net.peer.ip': '::1',
    'net.peer.port': 56849,
    'http.status_code': 304,
    'http.status_text': 'NOT MODIFIED',
    'http.route': ''
  },
  status: { code: 0 },
  events: [],
  links: []
}
```
直接アプリケーションコードに改修を加えることなくトレーシング情報を取得できました。

[^7]: https://opentelemetry.io/docs/instrumentation/

# エクスポート
取得したトレースを分析するためにはJaeger[^8]やZipkin[^9]などの分析ツールにデータをエクスポートする必要があります。OpenTelemetryはOpenTelemetry Protocol(OTLP)と呼ばれるエクスポーターが提供されています。OTLPを使って収集したデータをJaegerに送信してみます。

まずOTLPエクスポーターをインストールします。
```shell-session:エクスポーターインストール
$ npm install --save-dev @opentelemetry/exporter-trace-otlp-http
```

次に先ほど書いた`tracing.ts`を以下のように書き換えます。

```ts:tracing.ts
import * as opentelemetry from '@opentelemetry/sdk-node';
import {
  getNodeAutoInstrumentations,
} from "@opentelemetry/auto-instrumentations-node";
import {
  OTLPTraceExporter,
} from "@opentelemetry/exporter-trace-otlp-http";

const sdk = new opentelemetry.NodeSDK({
  traceExporter: new OTLPTraceExporter({
    // optional - default url is http://localhost:4318/v1/traces
    // optional - collection of custom headers to be sent with each request, empty by default
    headers: {},
  }),
  instrumentations: [getNodeAutoInstrumentations()],
});
sdk.start();
```

JaegerのDockerコンテナを実行します。

```shell-session:Jaegerコンテナ起動
$ docker run -d --name jaeger \
  -e COLLECTOR_ZIPKIN_HOST_PORT=:9411 \
  -e COLLECTOR_OTLP_ENABLED=true \
  -p 6831:6831/udp \
  -p 6832:6832/udp \
  -p 5778:5778 \
  -p 16686:16686 \
  -p 4317:4317 \
  -p 4318:4318 \
  -p 14250:14250 \
  -p 14268:14268 \
  -p 14269:14269 \
  -p 9411:9411 \
  jaegertracing/all-in-one:latest
  ```

`http://localhost:16686/search`にアクセスするとJaegerのUI画面にアクセスできます。
![](/images/what-is-opentelemetry/image2.png)
*Jaeger画面*

トレースの中にスパンが構成されており、トレース情報をクリックすると各スパンの詳細情報がチェックできます。
![](/images/what-is-opentelemetry/image3.png)
*トレース情報*

[^8]: https://www.jaegertracing.io/
[^9]: https://zipkin.io/

# 参考文献
https://www.oreilly.co.jp/books/9784814400126/