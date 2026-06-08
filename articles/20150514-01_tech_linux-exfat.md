---
title: "ubuntu14.04(Linux Mint 17.1)でのexFATのマウント方法"
emoji: "🎵"
type: "tech"
topics: ["ubuntu", "linuxmint", "exfat", "linux"]
published: true
published_at: 2015-05-14 16:28
---

## 経緯
NW-A16にはWalkmanで恐らく（間違ってたらごめんなさい）初めてmicroSDに対応したので・・・
flacの高音質なハイレゾ音源を入れるために64GBのmicroSDXCを買ったわけだ。
And, NW-A16でmicroSDXCをフォーマットするともれ無くexFATでフォーマットされてデフォルトのubuntu系（むしろLinux）では読めません。

というわけで、[ここ](http://itlx.ldblog.jp/archives/52048765.html)を参考にしてexfat-fuseとexfat-utilsをインストールしようにもコメント欄にあるようになぜか意味不明な公開鍵が出現し、リポジトリが登録され無い模様。

まぁ、Windowsじゃないんだしそういうこともあるさ。
というわけで、ならば手動で！！

## 追記（2015年12月6日）
makeができるようになったので、普通にソースインストールできます。

https://github.com/relan/exfat

以下の記事の必要性はないですが、過去ログとしておいておきます。

-----
### 【過去ログ】手動でのexfat-fuseとexfat-utilsのインスコ方法
#### [Downloads - exfat](https://code.google.com/p/exfat/wiki/Downloads?tm=2)
上記リンクより最新版のexfat-fuseとexfat-utilsを落とします。
2015/05/14現在最新はexfat-utils-1.1.1.tar.gzとfuse-exfat-1.1.0.tar.gz

#### 展開しましょう
ダウンロードしたファルダに`cd`してください。

```
tar xfz exfat-utils-1.1.1.tar.gz
tar xfz fuse-exfat-1.1.0-.tar.gz
```

#### インスコに必要なものを集める

```
sudo apt-get install fuse libfuse-dev scons
```

#### インスコ

```
cd fuse-exfat-1.1.0
sudo scons install
cd ../exfat-utils-1.1.1
sudo scons install
```

以上です。

インスコに関しては[展開しましょう](#展開しましょう)で同階層で2つのファイルを展開してることが前提なのでそれ以外の場所で展開したのなら`cd`のパスを適宜変更してください。
これでマウントできます。

**※過去ログはここまで**

-----

## 使い方
コマンドなら・・・

```shell-session
sudo mount.exfat-fuse /dev/sdXn /mnt/exfat
```

/etc/fstabに書くなら・・・

```shell-session
UUID=[uuid]   /home/share   exfat-fuse   async,auto,dev,exec,rw,suid,nouser   0   0
```

上手にできました〜。

## 参照URL
[exFatの外付けドライブを任意のユーザーでマウント](http://asithink001.blogspot.jp/2013/05/exfat.html)
[HOWTO - exFAT - Quick Start Guide](https://code.google.com/p/exfat/wiki/HOWTO)
