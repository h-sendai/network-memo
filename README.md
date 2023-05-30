# ネットワークメモ

Linuxネットワークに関するメモ

## ネットワークインターフェイス名の取得

``/sys/class/net/``にインターフェイス名の
シンボリックファイルがある:

```console
% ls /sys/class/net
enp2s0@  exp0@	jlan@  lo@

% ls -l /sys/class/net
total 0
lrwxrwxrwx. 1 root root 0 May 29 08:57 enp2s0 -> ../../devices/pci0000:00/0000:00:1c.5/0000:02:00.0/net/enp2s0/
lrwxrwxrwx. 1 root root 0 May 29 08:57 exp0 -> ../../devices/pci0000:00/0000:00:1c.1/0000:03:00.0/net/exp0/
lrwxrwxrwx. 1 root root 0 May 29 08:57 jlan -> ../../devices/pci0000:00/0000:00:01.0/0000:05:00.0/net/jlan/
lrwxrwxrwx. 1 root root 0 May 29 08:57 lo -> ../../devices/virtual/net/lo/
```

## HyperThreadの無効化

たいていHyperThreadが有効な状態で起動する。無効化するには
以下のいずれかの方法を使う。

- BIOSあるいはUEFIファームウェアで無効化する
- Linuxカーネルコマンドラインにnosmtを追加する
- ``echo off | sudo tree /sys/devices/system/cpu/smt/control``を実行する
- ``echo 0 | sudo tee /sys/devices/system/cpu/cpuN/online``を必要な数だけ繰り返す。NにHyperThreadに対応するCPUコア番号を入れる。

## ハードウェア情報の取得

hwlocおよびlwloc-guiパッケージをインストールするとCPUのコア番号の振り方、
L1, L2, L3キャッシュのコア間での共有情報がわかる。

```console
% hwloc-ls > hwloc-ls.out
% lstopo mymachine.png
```

最初のコマンドで情報がテキストで保存される。2番目のコマンドで
png画像ファイル化されたものが作られる。

(注: 以下2台ともHyperThreadはオフにした状態で起動している)

例: ASUS ESC4000サーバー

- [ASUS-ESC4000A-E10/hwloc-ls.txt](ASUS-ESC4000A-E10/hwloc-ls.txt)
- [ASUS-ESC4000A-E10/ESC4000A.png](ASUS-ESC4000A-E10/ESC4000A.png)

例: Dell PowerEdge T430 (CPUパッケージが2個ある例)

- [Dell-PowerEdge-T430/T430.txt](Dell-PowerEdge-T430/T430.txt)
- [Dell-PowerEdge-T430/T430.png](Dell-PowerEdge-T430/T430.png)

CPUが複数ある場合には使用するネットワークインターフェイスが
どのCPUパッケージのPCIeバスにつながっているか把握する必要がある。
上の出力でもわかるし
```console
% cat /sys/class/net/$NIC/device/local_cpulist
0,2,4,6,8,10,12,14,16,18
```
のように``/sys/class/net/$NIC/device/local_cpulist``でもわかる。
``$NIC``にはネットワークインターフェイス名を入れる。

## Ipコマンド

### IPアドレスの取得

```console
% ip address
```

``ip a``と省力化できる。

### IPアドレスのセット

```console
% ip a set 192.168.10.10/24 dev eth0
```

### ARPテーブル

```console
% ip n
```

### リンク

```console
% ip l
```

### ルーティングテーブル

```console
% ip r
```

## ネットワークインターフェイスの設定

ethtoolコマンドでネットワークインターフェイスの設定状況の取得、
変更ができる。

### ethtool -l eth0

ネットワークインターフェイスのキュー(リングバッファ数)の取得、セット
ができる。最大値はCPUコアになっていてそれより多くセットしようとすると
エラーになる。

例: igb (Intel 1 GbE)
```console
% ethtool -l exp0
Channel parameters for exp0:
Pre-set maximums:
RX:		n/a
TX:		n/a
Other:		1
Combined:	8
Current hardware settings:
RX:		n/a
TX:		n/a
Other:		1
Combined:	8
```

例: ixgbe (Intel 10 GbE) (CPUコアが8個のPCで実行した例。ハードウェアスペックとしてはもっと多い)

```console
% ethtool -l texp0
Channel parameters for texp0:
Pre-set maximums:
RX:		n/a
TX:		n/a
Other:		1
Combined:	8
Current hardware settings:
RX:		n/a
TX:		n/a
Other:		1
Combined:	8
```

例: ixgbe (Intel 10 GbE) (HyperThreadをいれてCPUコアが12個のPCで実行した例)

```console
% ethtool -l enp4s0f0
Channel parameters for enp4s0f0:
Pre-set maximums:
RX:		n/a
TX:		n/a
Other:		1
Combined:	12
Current hardware settings:
RX:		n/a
TX:		n/a
Other:		1
Combined:	12
```

### ethtool -g/-G eth0

リングバッファの長さの取得、設定ができる。

まず現在の設定値を確認し、4096(最大値)に変更する例:
```console
% ethtool -g enp4s0f0
Ring parameters for enp4s0f0:
Pre-set maximums:
RX:		4096
RX Mini:	n/a
RX Jumbo:	n/a
TX:		4096
Current hardware settings:
RX:		512
RX Mini:	n/a
RX Jumbo:	n/a
TX:		512

% sudo ethtool -G enp4s0f0 rx 4096 tx 4096

% ethtool -g enp4s0f0
Ring parameters for enp4s0f0:
Pre-set maximums:
RX:		4096
RX Mini:	n/a
RX Jumbo:	n/a
TX:		4096
Current hardware settings:
RX:		4096
RX Mini:	n/a
RX Jumbo:	n/a
TX:		4096
```

### ethtool -c/-C

## ソケットレシーブバッファ

## データ転送速度

## ドライバ

## CPUコア、キャッシュ共有状態の取得

## iperf2

[Intel Ethernet 800 Series Linux Performance Tuning Guide](https://www.intel.co.jp/content/www/jp/ja/support/articles/000088688/ethernet-products/800-series-network-adapters-up-to-100gbe.html)には

> Intel recommends iperf2 over iperf3 for most benchmarking situations
> due to the ease of use and support of multiple threads in a single
> application instance.

と書いてある。

```
iperf -i 1 -e -c remote
```

とするとRetransmissionのデータがでる。

## iperf3

## 割り込み

## ネットワークインターフェイスデータシート

ときどき改定されて下記URLからダウンロードできなくなっているかもしれない。

- Intel I350 Controller データシート (Linuxではドライバ名igbのもの)
https://www.intel.com/content/dam/www/public/us/en/documents/datasheets/ethernet-controller-i350-datasheet.pdf
- Intel 10GbE Controller X710 データシート https://www.intel.com/content/www/us/en/content-details/332464/intel-ethernet-controller-x710-xxv710-xl710-datasheet.html
- Intel 40GbE (i40e) Linux Performance tuning guide https://www.intel.com/content/www/us/en/content-details/334019/intel-ethernet-controller-x710-xl710-and-intel-ethernet-converged-network-adapter-x710-xl710-family-linux-performance-tuning-guide.html
- Intel Ethernet 800 Series Linux Performance Tuning Guide (100GbE) https://www.intel.co.jp/content/www/jp/ja/support/articles/000088688/ethernet-products/800-series-network-adapters-up-to-100gbe.html

## 解説URLメモ

- https://blog.packagecloud.io/monitoring-tuning-linux-networking-stack-receiving-data/
- https://blog.packagecloud.io/illustrated-guide-monitoring-tuning-linux-networking-stack-receiving-data/
- https://medium.com/coccoc-engineering-blog/linux-network-ring-buffers-cea7ead0b8e8
- https://github.com/leandromoreira/linux-network-performance-parameters
- https://fasterdata.es.net/
- https://blog.nishi.network/2020/08/26/Mellanox-100GbE-1/
