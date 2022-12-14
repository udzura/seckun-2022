# 演習2

## 2.1 Dockerのインストール

* 公式サイトの手順で構いませんので、Dockerをインストールしましょう。

https://docs.docker.com/engine/install/ubuntu/

## 2.2 コンテナを立ち上げる

* 任意のコンテナをデーモンとして立ち上げ（httpdなどがいいでしょう）、その状態でdockerdがどのようなプロセス/プロセスツリーを作るかを観察しましょう。

```
root         990  0.0  0.9 5537252 74216 ?       Ssl  10月31  14:50 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
...
root         802  0.5  0.6 1556512 49072 ?       Ssl  10月31 113:58 /usr/bin/containerd
...
root       39292  0.6  0.0 113368  7868 ?        Sl   22:01   0:00 ???? -namespace moby -id fb2278199b08796be5c0f30aa62fb7ef130baaee3f0059
root       39312  0.6  0.0   7300  4112 ?        Ss   22:01   0:00  \_ httpd -DFOREGROUND
www-data   39347  0.0  0.0 1998448 2952 ?        Sl   22:01   0:00      \_ httpd -DFOREGROUND
www-data   39348  0.0  0.0 1998448 3004 ?        Sl   22:01   0:00      \_ httpd -DFOREGROUND
www-data   39349  0.0  0.0 1998448 2944 ?        Sl   22:01   0:00      \_ httpd -DFOREGROUND

```

* その上で、それぞれの層のプログラムが何者かを調査しましょう。

## 2.3. youkiのインストール

* 公式の手順に従い、コンテナランタイム「youki」をビルド、インストールしてください。

https://containers.github.io/youki/user/basic_setup.html

* 正直いくつか罠があります。
  * Rustのツールセットに暗黙に依存しています。ドキュメントで読み飛ばしがちですが、　[Rust公式サイトの手順](https://www.rust-lang.org/ja/tools/install)でインストールできます
  * cloneに失敗する際は、 https:// 始まりの git URL を試してください
  * `make release-build` の方が確実に動くバイナリかも？

* Docker において OCI 低レベルランタイムはどのように置き換えればいいか、設定を調査してください。ヒントはyoukiの公式ドキュメントです。

https://containers.github.io/youki/user/basic_usage.html

* その上で `docker run` したらyoukiを使ってコンテナが立ち上がる状態にして、同じく httpd などを実行してください。

* その状態でのプロセスツリーはどうなっていますか？
  * youki 経由のコンテナであることはどう解るだろうか？

## 2.3.x 発展: crun/gVisorのインストール

ここから先は講師も手順を検証していませんが、一緒にトラブルを解決できればと思います。

* さらに別の低レベルランタイムを用います。
* crun
  * インストール/ビルドは公式のREADMEの通りです
  * https://github.com/containers/crun#readme
* gvisor
  * 同じく公式のドキュメントをご参照ください
  * https://gvisor.dev/docs/user_guide/install/
* docker 経由で動かす場合は、youkiのドキュメントの手順が参考になります。例えば以下のようになるでしょう。

```
$ sudo nohup dockerd --experimental --add-runtime="crun=/usr/local/bin/crun" >/tmp/dockerd.log &
$ docker run -it --rm --runtime crun hello-world
...
```

* 同様に、docker+runcと色々比較してみましょう。

## 2.3.y 大発展おまけ

* podmanについて以下を調査してみましょう。
  * インストール
  * httpd コンテナの立ち上げ方
  * ランタイムの切り替え方(crun/youki/...)と確認方法

----

## トラブルシュート

* youkiが動かない
* caps error: PR_CAPBSET_DROP failure: Invalid argument (os error 22) というエラーが出た

環境が間違って Ubuntu 20.04 である可能性があります。~youkiはおそらく Linux 5.8 以前では動作しません~ 最新mainでは直っています。

