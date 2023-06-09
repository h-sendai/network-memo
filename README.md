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

``/sys/devices/system/cpu/smt/control``に``echo off``すると
dmesgで次のような出力が出てくる:
```
[25142.533405] IRQ 52: no longer affine to CPU8
[25142.546184] IRQ 65: no longer affine to CPU8
[25142.558950] IRQ 82: no longer affine to CPU8
[25142.573205] smpboot: CPU 8 is now offline
[25142.614123] IRQ 39: no longer affine to CPU9
[25142.626892] IRQ 74: no longer affine to CPU9
[25142.639653] IRQ 75: no longer affine to CPU9
[25142.652417] IRQ 77: no longer affine to CPU9
[25142.665185] IRQ 84: no longer affine to CPU9
[25142.679053] smpboot: CPU 9 is now offline
[25142.728125] IRQ 35: no longer affine to CPU10
[25142.742211] smpboot: CPU 10 is now offline
[25142.766126] IRQ 37: no longer affine to CPU11
[25142.780655] smpboot: CPU 11 is now offline
[25142.802184] smpboot: CPU 12 is now offline
[25142.825170] smpboot: CPU 13 is now offline
[25142.856593] smpboot: CPU 14 is now offline
[25142.897265] smpboot: CPU 15 is now offline
```
CPU N offlineの他にofflineになったCPUが担当していた
IRQについて担当が変更になったというメッセージがでている。

offだったのを
``echo on > /sys/devices/system/cpu/smt/control``
でHyperthredingを有効にすると
```
[25475.690989] smpboot: Booting Node 0 Processor 8 APIC 0x1
[25475.735968] smpboot: Booting Node 0 Processor 9 APIC 0x3
[25475.753788] smpboot: Booting Node 0 Processor 10 APIC 0x5
[25475.772238] smpboot: Booting Node 0 Processor 11 APIC 0x7
[25475.790845] smpboot: Booting Node 0 Processor 12 APIC 0x9
[25475.809179] smpboot: Booting Node 0 Processor 13 APIC 0xb
[25475.827645] smpboot: Booting Node 0 Processor 14 APIC 0xd
[25475.846145] smpboot: Booting Node 0 Processor 15 APIC 0xf
```
とでる。

HyperThreadに関連して

Linux kernelに付属するドキュメント
Scaling in the Linux Networking Stack
https://www.kernel.org/doc/Documentation/networking/scaling.txt
には「HyperThreadは割り込みハンドリングには利益がない」と書かれている。
> Per-cpu load can be observed using the mpstat utility, but note that on
> processors with hyperthreading (HT), each hyperthread is represented as
> a separate CPU. For interrupt handling, HT has shown no benefit in
> initial tests, so limit the number of queues to the number of CPU cores
> in the system.

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

以下``$nic``にはネットワークインターフェイス名
（eth0、enp0s3、eno1など）を指定する。

### ethtool -l $nic

注: オプション-lはアイ I (eye)ではなくてエル l (ell)

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

ネットワークデバイス関連で結合(coalesce)の設定を行う。

たとえばパケットがやってきてから割り込みを出すまでに遅延させる
時間をマイクロ秒単位で指定するには

```console
% ethtool -C $nic rx-usecs 125
```
のようにする。指定する数値はマイクロ秒だが
0 - 4の値は特別な意味となるドライバもある。

例: e1000e (廉価版Intel 1 GbE)
https://www.kernel.org/doc/Documentation/networking/e1000e.txt
``ethtool -C $nic rx-usecs NNN``で設定する値は

> InterruptThrottleRate
>
> Valid Range:   0,1,3,4,100-100000 (0=off, 1=dynamic, 3=dynamic conservative,
>                                   4=simplified balancing)
> Default Value: 3

で0,1,3,4は単位がマイクロ秒の時間ではない。

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
に入っているREADME、あるいはlinux kernelソースコードの
Documentation/networking/device_drivers/ethernet/intel/ixgbe.rst
あるいは
https://www.kernel.org/doc/html/latest/networking/device_drivers/ethernet/intel/ixgbe.html#sideband-perfect-filters
には

(以下引用中hash '#'を'$'に変更した)
> NOTES:
> For each flow-type, the programmed filters must all have the same matching
> input set. For example, issuing the following two commands is acceptable:
> 
> ```
> $ ethtool -U enp130s0 flow-type ip4 src-ip 192.168.0.1 src-port 5300 action 7
> $ ethtool -U enp130s0 flow-type ip4 src-ip 192.168.0.5 src-port 55 action 10
> ```
> 
> Issuing the next two commands, however, is not acceptable, since the first
> specifies src-ip and the second specifies dst-ip:
> 
> ```
> $ ethtool -U enp130s0 flow-type ip4 src-ip 192.168.0.1 src-port 5300 action 7
> $ ethtool -U enp130s0 flow-type ip4 dst-ip 192.168.0.5 src-port 55 action 10
> ```
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

### ethtool -S $nic

``ethtool -S $nic``で統計情報がとれる。
取得できる統計情報はNIC、ドライバによって異なる。
``drop``、``buffer``、``miss``などの単語に注意する。

統計情報は
``/sys/class/net/$nic/statistics/rx_droped``その他でも
取得することができる。どれが更新されるか、またいつ更新されるかは
ドライバによる。

ixgbeだと``ethtool -S``でキューごとに送信、受信したパケット数と
バイト数がそれぞれ
``rx_queue_N_packets``, ``rx_queue_N_bytes``, ``tx_queue_N_packets``, ``tx_queue_N_bytes``
として表示される:
```
tx_queue_0_packets: 22918271
tx_queue_0_bytes: 13927114958
tx_queue_1_packets: 24053769
tx_queue_1_bytes: 1717815986
(省略)
rx_queue_0_packets: 137230359
rx_queue_0_bytes: 207270155624
rx_queue_1_packets: 2147500012
rx_queue_1_bytes: 3251308679248
```

カウンタなので1秒間の値を知りたければ引き算する必要がある。
サンプルpythonスクリプト: [count-nic-stats](count-nic-stats)

``ethtool -S $nic``を実行したときに数字とおぼしき
ところを1秒に1回引き算して表示する。数字でない(とおぼしき)
ところはそのまま表示する(数字かどうかの判定は
python3のint()を使っているだけ)。

``ethtool -S $nic``の出力例:
- [e100](ethtool-S/e100.txt)
- [e1000e](ethtool-S/e1000e.txt)
- [igb](ethtool-S/igb.txt)
- [tg3](ethtool-S/tg3.txt)
- [ixgbe](ethtool-S/ixgbe.txt)
- [i40e](ethtool-S/i40e.txt)

### ethtool -d $nic

``ethtool -d``でNICレジスタのダンプができる。出力結果は
NICにより、e1000e, igb, ixgbeはレジスタ名も表示されてわかりやすい。
これらのNICについては``ethtool -d $nic hex on``で16進でも表示可能。
一方、tg3, i40eは最初から16進で表示されるので意味を知りたければ
コントローラのデータシートを読む必要がある。

- [e1000eの例](e1000e.dump.txt)
- [igbの例](igb.dump.txt)、 [メモ](igb.md)

### ethtool -k/-K $nic

NICのいろいろの機能の状態(ON/OFF)をみたりセットしたりするオプション。

#### offload機能

Large Receive Offload (lro)はLinuxでは無効に設定されている。
かわりにソフトウェア実装のGeneric Receive Offloadが有効化されていて
tcpdumpで見るとlengthが(IPでみて)1460バイトを越えるパケットが観測される。

NICで受信したパケットはrx descriptorが指すPC上のメモリに
DMAで転送されるがそのときのサイズはlroが無効になっているせいか
1500バイト程度で入るようだ。

tcp-server-client/serverで
```
./server -b $((1448*2)) -r $((1448*10))
(1448*2バイトのバッファをwrite()して1448*10バイト/秒になるように
データを送る)
```
と起動して受信してtcpdumpで見ると``length 2896``とでてくる。
同時に``ethtool -d $nic``で1秒おきに``RDH0N``の増加数を
みると1秒に10個づつふえているので、rx descriptorがさす
バッファに1500*2バイト入るわけではないことが確認できる。

## ソケットレシーブバッファ

``cat /proc/sys/net/ipv4/tcp_moderate_rcvbuf``が1なら
ソケットレシーブバッファの自動調節機能が有効になっている。
この場合の最小値、デフォルト値、最大値は
受信: ``/proc/sys/net/ipv4/tcp_rmem``
送信: ``/proc/sys/net/ipv4/tcp_wmem``にある3個の数値である。
大きくするのは最後の最大値のみにしておくのが無難な
気がする。
大きくするには、たとえばこのスクリプト
[get-net-params](get-net-params)
を走らせ、ファイルに保存し、``max``の値を書き換える
(このスクリプトでは1GBにしている)。

## データ転送速度

## ドライバ

たいていのイーサネットカードのドライバはLinuxに含まれている。
メーカーによっては別に配布されているものもある。
いままで使ってきたIntelの製品についてはLinuxに含まれる
ドライバ(100 Mbpsのe100、1 Gbpsのe1000, e1000e, igb,
10Gbpsのixgbe, 40 Gbpsのi40e)で困ったことはない。
NVIDIA Mellanoxについては経験がほとんどないので不明。

- Intel<br>
NICのドライバはLinuxソースに入っているものとは別に
https://sourceforge.net/projects/e1000/files/
で配布されている。e1000とパスにはいっているが、
e1000e, igb, ixgbe, i40e, iceドライバもここから
ダウンロードできる。
Linuxソースに入っているもとの比べると
ethtoolインターフェイスの実装がはいっていないようで
``ethtool -C $nic rx-usecs NNN``の設定はカーネルコマンドラインで
指定するようだ。
- NVIDIA Mellanox<br>
https://network.nvidia.com/products/infiniband-drivers/linux/mlnx_ofed/
から取得できる。使ったことはない。

## CPUコア、キャッシュ共有状態の取得

hwloc-lsコマンドでCPU番号の割り振りとL1, L2, L3キャッシュの
共有状態についてまとめたテキストデータが取得できる。
lstopoコマンドを使うと画像データとして視覚的に
表現したものが得られる。
AlmaLinuxではそれぞれhwlocパッケージ、hwloc-guiパッケージに
入っている。

CPUキャッシュの共有状態については
``/sys/devices/system/cpu/cpuN/cache/indexM/shared_cpu_list``
(N, Mにはそれぞれ整数値が入る)をみてもわかる。

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

ネットワークキューの割り込み番号は
``/proc/interrupts``を見るとわかる。``:``の前が割り込み番号を表す。
``/proc/interrupts``は横に長いので``less -S /proc/interrupts``として
端末に表示しきれないところはチョップするのがいいかもしれない。右の
ほうをみるには右矢印キーを使う。

各IRQを処理するCPU番号は
``/proc/irq/<IRQ番号>/smp_affinity_list``
をみればわかる。担当CPUを変更するにはこのファイルに
echoコマンドで整数値をかけばよい。

デフォルトではirqbalanceデーモンが起動していて
デフォルトでは10秒に1回、
各CPUの各割り込み処理回数を平均化する処理をしている。
手動でCPU番号を指定する場合にはirqbalanceで上書き
されないようにirqbalanceを停止しておく。停止するコマンドは
```
systemctl stop irqbalance
```

``/proc/intterrupts``にはCPUコア毎に処理したハードウェア割り込みの
回数が書かれている。1秒間に起きた回数を知りたい場合は1秒おきに読み、
前の数字をおぼえておいて引き算するという作業が必要になる。

ハードウェア割り込み処理のあとにソフトウェア割り込み処理
(softirq)が行われる。softirqはハードウェア割り込み処理を
おこなったのと同一CPUコアで行われる。

ソフトウェア割り込み(softirq)の回数は
``/proc/softirqs``に種類別に書かれている。

## softnet_stat

``/proc/net/softnet_stat``にCPUごとの統計情報がのっている。

- 各行CPUごとのデータ(CPUコアが8コアなら8行になっている)
- 数値は全部16進数
- 第1コラム: ``sd->processed``: 処理したフレーム数
- 第2コラム: ``sd->dropped``: キューにあきがなくてドロップした数
- 第3コラム: ``sd->time_squeeze``: net_rx_actionループがbudgetをくいつぶしたか、時間制限をこえたかで終了したしたがまだ処理できた回数。

ここで第1コラムのフレーム数というのはgro, lroのあとのフレーム数の
ことのようだ。gro, lroのあとのフレームのデータ長は1448あるいは1460以上ある。
たとえばtcpdumpで見るとlengthとして下のように1460以上あるように
表示される。
```
length 8688
length 10136
length 8688
length 10136
length 8688
length 10136
length 4344
```

## パケット処理の観察

### レジスタ
[e1000eレジスタの観察](e1000e.md)

### mpstat -P ALL 1と/proc/softirqsの出力

CPUコア8個のPCでテスト。10GbE (Intel ixgbe)を使用。
データはポート1234からやってくる。

```
sudo ethtool -U texp0 flow-type tcp4 src-port 1234 action 7
```
でポート1234からのデータをqueue #7に送り、
CPU affinityを
```
irq cpu aff. queue
--- -------- --------------------------
 62        0 texp0-TxRx-0
 63        1 texp0-TxRx-1
 64        2 texp0-TxRx-2
 65        3 texp0-TxRx-3
 66        4 texp0-TxRx-4
 67        5 texp0-TxRx-5
 68        6 texp0-TxRx-6
 69        7 texp0-TxRx-7
```
とセットしてqueue #7の処理をCPU #7にさせるようにセットし、
sched_setaffinity()でユーザープロセスをCPU #2で走らせたときの
``mpstat -P ALL 1``の出力:
```
             CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
16:40:30     all    0.88    0.00    3.25    0.00    0.12    2.88    0.00    0.00    0.00   92.88
16:40:30       0    0.00    0.00    0.99    0.00    0.00    0.00    0.00    0.00    0.00   99.01
16:40:30       1    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
16:40:30       2    1.01    0.00   24.24    0.00    0.00    1.01    0.00    0.00    0.00   73.74
16:40:30       3    4.95    0.00    0.99    0.00    0.00    0.00    0.00    0.00    0.00   94.06
16:40:30       4    1.01    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00   98.99
16:40:30       5    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
16:40:30       6    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
16:40:30       7    0.00    0.00    0.00    0.00    0.99   21.78    0.00    0.00    0.00   77.23

```

またこのときの``/proc/interrupts``の``当該キュー``の部分
(スクリプトを書いて1秒間の増加分に変更している):
```
16:40:30    62:      1      0      0      0      0      0      0      0 IR-PCI-MSI 1574912-edge      texp0-TxRx-0 
16:40:30    63:      0      1      0      0      0      0      0      0 IR-PCI-MSI 1574913-edge      texp0-TxRx-1 
16:40:30    64:      0      0  25853      0      0      0      0      0 IR-PCI-MSI 1574914-edge      texp0-TxRx-2 
16:40:30    65:      0      0      0      1      0      0      0      0 IR-PCI-MSI 1574915-edge      texp0-TxRx-3 
16:40:30    66:      0      0      0      0      1      0      0      0 IR-PCI-MSI 1574916-edge      texp0-TxRx-4 
16:40:30    67:      0      0      0      0      0      1      0      0 IR-PCI-MSI 1574917-edge      texp0-TxRx-5 
16:40:30    68:      0      0      0      0      0      0      1      0 IR-PCI-MSI 1574918-edge      texp0-TxRx-6 
16:40:30    69:      0      0      0      0      0      0      0  13244 IR-PCI-MSI 1574919-edge      texp0-TxRx-7 
```

またこのときの``/proc/softirqs``の``NET_{RX,TX}``の部分
```
                   CPU0   CPU1   CPU2   CPU3   CPU4   CPU5   CPU6   CPU7
16:40:30 NET_TX:      0      0      0      0      0      0      0      0 
16:40:30 NET_RX:     29     42  25875     39     61     57     25  15029 
```

- ``/proc/interrupts``を見るとプロセスが稼働しているCPU 2でハードウェアわりこみ処理を
している。TCP ack送信(?)
  ``/proc/softirqs``のほうのNET_TXは微動だにせず0のままだった。
- ``/proc/softirqs``のCPU2はユーザープロセスのソケットに
  データをくべているのが計上されている(?)。
- ``/proc/softirqs``のCPU2と7以外のものは組織内LANに接続した
  別NICが受信したパケットの処理。

## ネットワークインターフェイスデータシート

ときどき改定されて下記URLからダウンロードできなくなっているかもしれない。

- Intel 8257x Contoller データシート (Linuxではドライバ名e1000eのもの)
https://sourceforge.net/projects/e1000/files/8257x%20Developer%20Manual/Revision%201.8/
- Intel I350 Controller データシート (Linuxではドライバ名igbのもの)
https://www.intel.com/content/dam/www/public/us/en/documents/datasheets/ethernet-controller-i350-datasheet.pdf
- Intel 82599 10 GbE Controllerデータシート (Linuxではドライバ名ixgbeのもの) https://cdrdv2-public.intel.com/331520/82599-datasheet-v3-4.pdf
- Intel X550 10 GbE Controllerデータシート (Linuxではドライバixgbeのもの) https://cdrdv2-public.intel.com/333369/333369-X550_Datasheet_Rev2.6.pdf
- Intel 40GbE Controller X710 データシート https://www.intel.com/content/www/us/en/content-details/332464/intel-ethernet-controller-x710-xxv710-xl710-datasheet.html
- Broadcom NetXtreme BCM5751 (tg3) https://docs.broadcom.com/doc/1211168564147
- NVIDIA Mellanox https://forums.developer.nvidia.com/t/is-there-a-programmers-guide-available-for-the-connextx-5-series-of-cards/206137 に
"This document is only available for customers with a valid support contract."と書かれている。

## ボードメーカー解説

- Intel 40GbE (i40e) Linux Performance tuning guide https://www.intel.com/content/www/us/en/content-details/334019/intel-ethernet-controller-x710-xl710-and-intel-ethernet-converged-network-adapter-x710-xl710-family-linux-performance-tuning-guide.html
PDF: Ethernet Controller X710/XL710 (40GbE) Linux Performance Tuning Guide
https://cdrdv2-public.intel.com/334019/xl710-x710-performance-tuning-linux-guide.pdf
- Intel Ethernet 800 Series Linux Performance Tuning Guide (100GbE) https://www.intel.co.jp/content/www/jp/ja/support/articles/000088688/ethernet-products/800-series-network-adapters-up-to-100gbe.html
PDF:
https://cdrdv2-public.intel.com/636781/800%20Series%20Linux%20Performance%20Tuning%20Guide_Rev1.1.pdf

- Intel Flow Directorがサポートされている機器リスト
https://www.intel.co.jp/content/www/jp/ja/support/articles/000031907/ethernet-products/700-series-controllers-up-to-40gbe.html
- Intel Flow Director設定
https://www.intel.com/content/www/us/en/developer/articles/training/setting-up-intel-ethernet-flow-director.html

## 解説URLメモ

- Monitoring and Tuning the Linux Networking Stack: Receiving Data https://blog.packagecloud.io/monitoring-tuning-linux-networking-stack-receiving-data/
- Illustrated Guide to Monitoring and Tuning the Linux Networking Stack: Receiving Data https://blog.packagecloud.io/illustrated-guide-monitoring-tuning-linux-networking-stack-receiving-data/
- Linux network ring buffers https://medium.com/coccoc-engineering-blog/linux-network-ring-buffers-cea7ead0b8e8
- linux-network-performance-parameters https://github.com/leandromoreira/linux-network-performance-parameters
- ESnet Fasterdata Knowledge Base https://fasterdata.es.net/
- 100GbEをテストする　計測編 Mellanox ConnectX-5 https://blog.nishi.network/2020/08/26/Mellanox-100GbE-1/
- Linuxでロードバランサやキャッシュサーバをマルチコアスケールさせるためのカーネルチューニング https://blog.yuuk.io/entry/linux-networkstack-tuning-rfs
- 10GbE時代のネットワークI/O高速化 https://speakerdeck.com/syuu1228/ogao-su-hua
