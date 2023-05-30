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
## ハードウェア情報の取得

hwlocおよびlwloc-guiパッケージをインストールするとCPUのコア番号の振り方、
L1, L2, L3キャッシュのコア間での共有情報がわかる。

```console
% hwloc-ls > hwloc-ls.out
% lstopo mymachine.png
```

最初のコマンドで情報がテキストで保存される。2番目のコマンドで
png画像ファイル化されたものが作られる。

例: ASUS ESC4000サーバー

- [ASUS-ESC4000A-E10/hwloc-ls.txt](ASUS-ESC4000A-E10/hwloc-ls.txt)
- [ASUS-ESC4000A-E10/ESC4000A.png](ASUS-ESC4000A-E10/ESC4000A.png)


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

### ethtool -g/-G

### ethtool -l/-L

ネットワークインターフェイスのリングバッファ数の取得、セット。
```console
% ethtool -l eth0
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
