---
theme: default
title: mirrordでKubernetes環境での開発体験を向上させる！
class: center
highlighter: shiki
drawings:
  persist: false
transition: slide-left
mdc: true
colorSchema: dark
fonts:
  sans: "M PLUS Rounded 1c"
---

## mirrordでKubernetes環境での開発体験を向上させる！

<div class="absolute bottom-10">

Kubernetes Novice Tokyo #33

2024/7/30 @mishio-n

</div>

---

## 自己紹介

<Spacer :p=20 />

<img src="/icon.png" class="rounded-full w-50"/>

<div class="absolute top-20 left-80">

### Mishio Nagadome (@mishio-n)

<Spacer :p=20 />

- Sky株式会社
- 社内システムの開発、運用
  - SRE、プラットフォームエンジニアリング
  - Kubernetes歴2年
  - CI/CD
  - BackStageプラグイン作成

</div>

<div class="absolute bottom-10 text-sm">

＊発表内容は私自身の見解であり、必ずしも所属する企業や組織の立場、戦略、意見を代表するものではありません。

</div>

---

<Spacer :p=20 />

mirrordはKubernetes環境でローカルプロセスを実行できる、シンプルかつパワフルなツールです。

Kubernetes、あるいはマイクロサービスでの開発において、動作確認のためのデータがローカル環境に用意できなかったり、サービス同士の依存関係により書いたコードをステージング環境にデプロイしないと動作を確認できない、といった課題がありました。

~~これらを改善すべくmirrordを導入した話や、mirrordを使った開発体験を紹介したいと思います。~~

<Spacer :p=40 />

<v-click>

### → mirrordというツールの紹介をします！

</v-click>

---
layout: cover
---

## ローカル環境で開発していて、こんな経験ありませんか？

<Spacer :p=30 />

- フロントエンドを修正したいだけなのに、バックエンド、DBを<Underline important>ローカルで用意</Underline>しないと確認できない
- 開発するとき、システムが依存するサービス全てを<Underline important>手元で立ち上げ</Underline>ないといけない
- 動くか不明だけど、開発・ステージング環境でないと検証できないからとりあえずマージ/デプロイする
- 何かよくわからないエラーが発生、それっぽいところにログを仕込んでデプロイする（をN回繰り返す）
- コードをプッシュして、CIでデプロイされるまで10分待ちぼうけになる

---
layout: center
---

## 🥺　つらい、めんどい、やりたくない　🥺

---
layout: cover
---

## こんなことできたらいいな、と思ったことないですか？

<Spacer :p=30 />

- 手元の環境から直接、ステージングにデプロイされているサービスにリクエストを送りたい
- ローカルでも、実際の認証認可処理を通過したリクエストを検証したい
- デプロイしているアプリケーションのコードを直接いじってデバッグしたい
- 手元の修正コードを一瞬だけデプロイして確認したい

---
layout: center
---

## 🪞　できます。そう、mirrordならね。　🪞

---

<img src="https://raw.githubusercontent.com/metalbear-co/mirrord/main/images/logo.svg" />

---

<Transform scale="0.75" origin="top">

<img src="/mirrord-site.png" />

</Transform>

---

- https://mirrord.dev/

- https://github.com/metalbear-co/mirrord


## 「Develop <span class="text-[#756df3] font-bold">Locally</span> with Your Kubernetes Environment」

mirrord lets developers <Underline important>run local processes in the context of their Kubernetes environment.</Underline>  
It’s meant to provide the benefits of running your service on a cloud environment  
(e.g. staging) <Underline important>without actually going through the hassle of deploying it there</Underline>,  
and without disrupting the environment by deploying untested code.

<Spacer :p=20 />

### ”デプロイすることなく、Kubernetes環境でコードを動かせる”

---

<img src="/deploy-cycle.png" />


---

## mirrordのアーキテクチャ

https://mirrord.dev/docs/reference/architecture/

<img src="https://mirrord.dev/docs/reference/architecture/architecture.svg" />

---

## mirrordの使い方

<Spacer :p=20 />

- VSCode拡張
- IntelliJプラグイン
- <span v-mark="{ at: 1, color: 'orange', type: 'underline' }">CLIツール（バイナリ）</span>

<Spacer :p=20 />

### CLIのインストール

https://github.com/metalbear-co/mirrord?tab=readme-ov-file#cli-tool

- homebrew
  ```
  brew install metalbear-co/mirrord/mirrord
  ```

- bash
  ```
  curl -fsSL https://raw.githubusercontent.com/metalbear-co/mirrord/main/scripts/install.sh | bash
  ```

---

## mirrordの使い方


<Spacer :p=20 />

ミラーリングしたいワークロードと動かすローカルプロセスをコマンドで指定して実行する

<Spacer :p=20 />

```
mirrord exec --target/-t <target> <process command> 
```

<Spacer :p=10 />


```sh
# example
mirrord exec -t pod/nginx node dist/main.js
```

---
layout: center
---

## どんなことができるの？ 🤔

---

## ネットワークのミラーリング/スチール

<Spacer :p=20 />

- Kubernetesクラスター内のネットワークトラフィックを用いてローカルプロセスを動かす
- トラフィックのミラーリングとスチールの両方が可能
  - スチールした場合は全てのトラフィックがローカルに転送される
- 対象とするトラフィックをHTTPヘッダーやアドレスなどで条件指定することも可能

<Spacer :p=20 />

例えば、

- ステージング環境に流れるリクエストを使ってローカルでデバッグ、動作確認
- ミラーリングしたリクエストに対し、手元のコードにのみデバッグログを仕込んで中身を確認

とかができる！

---

## リモートコンテキストを使用したリクエストの送信

<Spacer :p=20 />

- ローカルプロセスから送信されるリクエストは<Underline important>リモートのコンテキストで実行される</Underline>
- 名前解決もリモートコンテキストで実行されるので、<Underline important>後続のサービス間通信がそのまま行える</Underline>
  - Kubernetes → ローカル → Kubernetesのようなイメージ
- DB等、アクセス元が制限されているケースでも突破できる
- 逆に後続処理は別のローカルプロセスに転送したい場合など、挙動を細かくカスタマイズすることもできる

<Spacer :p=10 />

依存関係があっても、<Underline important>修正したいアプリケーションだけ</Underline>手元で起動すればOK！

<Transform scale="0.35" origin="">

<img src="/remote.png" />

</Transform>

---

## 環境変数の継承

<Spacer :p=20 />

- mirrordがターゲットのPodの持つ環境変数をローカルプロセスにも設定してくれる
- DBの接続情報や依存サービスの宛先を環境変数で管理していても、そのまま活用できる
- KubernetesのSecretが読み取り放題になってしまうのは注意点かも

---

## デモ

https://github.com/mishio-n/mirrord-demo

- 適当なHTTPサーバー3つ用意しました
- リクエスト-> `service-a` -> `service-b` -> `service-c` の順にトラフィックが流れます
- デプロイ済みのワークロードに対し、mirrordを使って挙動を変えてみます

<Spacer :p=10 />

<Transform scale="0.5" origin="">

<img src="/demo.png" />

</Transform>

---
layout: cover
---

## デモできなくてごめんなさい 🙇

---

## mirrordの設定

`mirror`と`steal`2つのモードが用意されており、  
デフォルトは`mirror`で通信をミラーリングする挙動になります。

<Spacer :p=10 />

このようなデフォルトの設定を変更する場合、jsonで設定ファイルを書いて読み込ませる必要があります。  

```json
// VS Code拡張を入れている場合、`mirrord.json`というファイル名にすると補完が効きます
{
    "feature": {
        "network": {
            "incoming": "steal"
        }
    }
}
```

<div class="absolute bottom-10 text-sm">

＊現在はIstioなどサービスメッシュだとミラーリングはサポートされてないらしいです

</div>

---

### その他のオプション

<Spacer :p=10 />

ミラーリング・スチールする通信の条件を設定して、対象を限定することができます。  
例えばHTTPヘッダーにアクセス者の情報が含まれている場合、自分にだけ影響を絞ることも可能です。

```json
{
    "feature": {
        "network": {
            "incoming": {
                "mode":"steal",
                // X-Real-IPとかでIPで制御もできるかも
                "http_filter": {
                    "header_filter": "access-user-account: mishio"
                }
            }
        }
    }
}
```

---

### その他のオプション

<Spacer :p=10 />

ポートを合わせなくても、マッピングすることが可能です。

```json
{
    "feature": {
        "network": {
            "incoming": {
                "mode": "steal",
                // ローカルが3000で、リモート（ポッド）が3030
                "port_mapping": [[3000, 3030]]
            }
        }
    }
}
```

---

### その他のオプション

コマンドでターゲット（Pod）を指定しない場合、`ターゲットレスモード` になります。

<Spacer :p=10 />

```
mirrord exec <process command> 
```

<Spacer :p=10 />

ネットワークのミラーリング・スチールは行われませんが、  
送信時のコンテキストはリモートのものを活用できるので、  
新規のワークロードからサービス間通信を試したい場合に有効かもしれません

---

## Telepresenseとの比較

https://mirrord.dev/compare/telepresence/

<Spacer :p=10 />

<img src="/compare.png" />

---
layout: center
---

# Awesome！

---

### ● メモ・補足

<Spacer :p=5 />

- 本番でやるのはNG。ステージングまでにしましょう
- ローカルプロセスに対しては、localhostでのリクエストももちろんOK
- mac以外で動くか検証できてない（<Underline important>WSLだとうまく動かなかった</Underline>）
- フレームワークとの相性など、他にも動かない条件はそこそこあるかも。。。
- 内部的にポートフォワードを使うので、権限管理している場合は注意

<Spacer :p=30 />

### ● ミラーリングとスチールの使い分け

<Spacer :p=5 />

- スチールは本来のリクエストを奪うので、場合によっては他の人に影響がでる。  
  ステージングなど安定性が求められる環境の場合は周知するのが吉。
- ミラーリングは影響が少ない反面、書き込み処理が2重に走る可能性があるので  
  検証内容によってはうまく使えない。
---
layout: cover
---

## まとめ

<Spacer :p=10 />

● ローカル開発環境、たくさんつらみあるよね  
● **mirrord**を使ってKubernetes環境をうまく使えばもっと楽になるかも

---
layout: center
---

# よきKubernetesライフを！

ご清聴ありがとうございました
