# ネットワークメモ

Linuxネットワークに関するメモ

## ip(1)コマンド

[ip-command.md](ip-command.md)に書いた。

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

CPU番号についていくつか出力されるが
``P#N``のNが``taskset -c N``とか``sched_setaffinity()``で使われる
CPUコア番号を示している。

画像中PCIeバスの近くに数字が書いてあるが、これはPCIe機器の
リンクアップスピードを表しているようだ。単位はGB/s ギガバイト毎秒。
lspci -vvvでリンクアップスピードが取得できる。lspciのほうは単位が
GT/s。1GT/sは1Gbps * (128/130)でほぼGbpsと同じ。

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

### ethtool -a/-A eth0

flow control (pause)の設定状況および設定ができる。

```console
% ethtool -A enp4s0f0 rx on tx on
```

flow controlはリンクパートナー側(スイッチに接続しているなら
スイッチ)の設定も必要になることがある。

### ethtool -c/-C

### ethtool -U

```
sudo ethtool -U $nic flow-type tcp4 src-port 1234 action 2
```
ソースポートが1234であるパケットを2番目のキューに入れる。
キューの番号は0から始まる。

(注)
```
sudo ethtool -K $nic ntuple on
```
してからでないと``ethtool -U``できないNICもある
(Intel 10GbEが該当する。
Mellanox MT2892 Family (ConnectX-6 Dx)では必要なかった。
(注終わり)

設定は``ethtool -u $nic``で読める。

Intel 10GbE (ixgbe)の例:
```console
% sudo ethtool -u texp0
8 RX rings available
Total 1 rules

Filter: 2045
    Rule Type: TCP over IPv4
    Src IP addr: 0.0.0.0 mask: 255.255.255.255
    Dest IP addr: 0.0.0.0 mask: 255.255.255.255
    TOS: 0x0 mask: 0xff
    Src port: 0 mask: 0xffff
    Dest port: 80 mask: 0x0
    VLAN EtherType: 0x0 mask: 0xffff
    VLAN: 0x0 mask: 0xffff
    User-defined: 0x0 mask: 0xffffffffffffffff
    Action: Direct to queue 1
```
消すのは``Filter:``で指定されている番号を使う。
```
ethtool -U $nic delete 2045
```

Mellanox MT2892 Family (ConnectX-6 Dx)の例:
```
Filter: 1023
    Rule Type: TCP over IPv4
    Src IP addr: 0.0.0.0 mask: 255.255.255.255
    Dest IP addr: 0.0.0.0 mask: 255.255.255.255
    TOS: 0x0 mask: 0xff
    Src port: 1234 mask: 0x0
    Dest port: 0 mask: 0xffff
    Action: Direct to queue 6
```

portの他にIPアドレスなども指定できるようだ。

#### 注意

Intelが出しているixgbeドライバ
https://sourceforge.net/projects/e1000/files/ixgbe%20stable/5.18.13/
に入っているREADMEには

(以下引用中hash '#'を'$'に変更した)
> NOTES:
> For each flow-type, the programmed filters must all have the same matching
> input set. For example, issuing the following two commands is acceptable:
> 
> $ ethtool -U enp130s0 flow-type ip4 src-ip 192.168.0.1 src-port 5300 action 7
> $ ethtool -U enp130s0 flow-type ip4 src-ip 192.168.0.5 src-port 55 action 10
> 
> Issuing the next two commands, however, is not acceptable, since the first
> specifies src-ip and the second specifies dst-ip:
> 
> $ ethtool -U enp130s0 flow-type ip4 src-ip 192.168.0.1 src-port 5300 action 7
> $ ethtool -U enp130s0 flow-type ip4 dst-ip 192.168.0.5 src-port 55 action 10
> 
> The second command will fail with an error. You may program multiple filters
> with the same fields, using different values, but, on one device, you may not
> program two tcp4 filters with different matching fields.
> 
> The ixgbe driver does not support matching on a subportion of a field, thus
> partial mask fields are not supported.

という注意がある。たしかに

```
% sudo ethtool -U $nic flow-type tcp4 dst-port 1234 action 3
Added rule with ID 2045
% sudo ethtool -U $nic flow-type tcp4 src-port 1234 action 3
rmgr: Cannot insert RX class rule: Invalid argument
```

となる。

### ethtool -d $nic

``ethtool -d``でNICレジスタのダンプができる。出力結果は
NICにより、e1000e, igb, ixgbeはレジスタ名も表示されてわかりやすい。
これらのNICについては``ethtool -d $nic hex on``で16進でも表示可能。
一方、tg3, i40eは最初から16進で表示されるので意味を知りたければ
コントローラのデータシートを読む必要がある。

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

- Intel 8257x Contoller データシート (Linuxではドライバ名e1000eのもの)
https://sourceforge.net/projects/e1000/files/8257x%20Developer%20Manual/Revision%201.8/
- Intel I350 Controller データシート (Linuxではドライバ名igbのもの)
https://www.intel.com/content/dam/www/public/us/en/documents/datasheets/ethernet-controller-i350-datasheet.pdf
- Intel 10GbE Controller X710 データシート https://www.intel.com/content/www/us/en/content-details/332464/intel-ethernet-controller-x710-xxv710-xl710-datasheet.html
- Intel 40GbE (i40e) Linux Performance tuning guide https://www.intel.com/content/www/us/en/content-details/334019/intel-ethernet-controller-x710-xl710-and-intel-ethernet-converged-network-adapter-x710-xl710-family-linux-performance-tuning-guide.html
- Intel Ethernet 800 Series Linux Performance Tuning Guide (100GbE) https://www.intel.co.jp/content/www/jp/ja/support/articles/000088688/ethernet-products/800-series-network-adapters-up-to-100gbe.html

- Broadcom NetXtreme BCM5751 (tg3) https://docs.broadcom.com/doc/1211168564147

## 解説URLメモ

- https://blog.packagecloud.io/monitoring-tuning-linux-networking-stack-receiving-data/
- https://blog.packagecloud.io/illustrated-guide-monitoring-tuning-linux-networking-stack-receiving-data/
- https://medium.com/coccoc-engineering-blog/linux-network-ring-buffers-cea7ead0b8e8
- https://github.com/leandromoreira/linux-network-performance-parameters
- https://fasterdata.es.net/
- https://blog.nishi.network/2020/08/26/Mellanox-100GbE-1/
- https://blog.yuuk.io/entry/linux-networkstack-tuning-rfs
