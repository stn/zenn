---
title: "Raspberry Piによるクラスター構築（ハード編）" # 記事のタイトル
emoji: "🍓" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["RaspberryPi"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---
# Raspberry Piによるクラスター構築（ハード編）

おなじみの取り組みだが、自分用の忘備録として残しておく。

ハードウェア構成として「[Raspberry Pi 4Bで4台構成の自宅クラスター！ ラズパイ4B向けPoE HATを試す](https://internet.watch.impress.co.jp/docs/column/shimizu/1325054.html)」の記事を参考にさせてもらった。

お買物リスト
- Raspberry Pi 4B x 4
- ヒートシンク（お好きなものを）HATとの干渉に注意
- GeekPi Raspberry Pi クラスターケース ZP-0088 x 1
- TP-Linkスイッチングハブ TL-SG1005P x 1
- DSLRKIT Power Over Ethernet PoE HAT for Raspberry Pi 4B 3B+ 5V 2.5A x 1
- UCTRONICS PoE HAT for Raspberry Pi 4 x 1
- Elecom LANケーブル CAT6 15cm LD-GPY/BU015 x 4
- Archgon NVMe PCIe M.2 SSD 外付けケース USB3.1Gen2 MSD-221 x 4
- Samsung MZ-V8V500B/EC (500GB M.2) x 3 (タイムセールで3つまでしか買えませんでした)
- Crucial CT500P2SSD8JP (500GB M.2) x 1
- JulyTek USB Type C (USB C to A 13.7cm) x 4

SSDはお好みで。SSDの仕様の違いで使用できるケースが変わるので注意。

## クラスターケースに組み込む前にやっておくこと。

Raspberry Piをクラスターケースに組み込むとmicroSDへのアクセスが難しくなるため、Rasberry PiをUSB起動するように設定する。
全面からアクセスするためのボードも付随しますが本体の右側にはSSDドライブを置きたいため、使用しなかった。

microSDにRasberry Pi OS Liteをインストールし、ログイン（userはpi, パスワードはraspberry）。

`rasp-config`を用いてUSB起動するように設定。

```bash
sudo rasp-config
```

Advanced Options -> Boot Order -> USB Boot

あと、GPUメモリを16MBにしておくとよいという情報もあるが、後でUbuntuに入れ直すので、これは不要かなと思いつつも一応（この設定はBIOSじゃないよね）

Performance Options -> GPU Memory

## Ubuntu のインストール

Rasberry Pi OSは32-bitなので、64-bitのあるUbuntuをSSDドライブにインストールする。

MacにてRaspberry Pi Imagerを起動し、Ubuntu Serverをインストール。
Other general purpose OS -> Ubuntu -> Ubuntu Server 21.04 (RPi 3/4/400)

> Ubuntu Server 20.04.2 LTSは起動しなかった。

Raspberry PiのUSBにSSDドライブを接続し、起動。

ログインはubuntu / ubuntu

ログイン後、すぐにパスワードの変更が求められる。

その他にはホスト名の設定を行っている。

```
hostnamectl set-hostname rasp1
```

> LANケーブルを赤、青、黄色、緑と4色用意して、マシン名をred, blue, ...をしている海外記事を見かけたでのやってみたかったが４色の短いLANケーブルが見つからなかった。

あと、MACアドレスを確認し、DHCP serverに登録する。

```
ip a
```

これを4台分、繰り返す。

## クラスターケースに組み込み。

あとはクラスターに組み込むだけ。
ファンLEDは電流がぎりぎりな（実は少し足りていない）ため一度は光らせたものの外しておく。
電源ケーブルの配線が不要ですっきりとしたクラスターが構築できました。

![](/images/articles/RaspberryPi4Cluster2.jpg)

## 参考

- [Upgrade your Raspberry Pi 4 with a NVMe boot drive](https://alexellisuk.medium.com/upgrade-your-raspberry-pi-4-with-a-nvme-boot-drive-d9ab4e8aa3c2) - NVMeにすると速くなるという話
