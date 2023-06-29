# igb

## sudo ethtool -d $nic

[igb.dump.txt](igb.dump.txt)

sudo ethtool -d $nicでレジスタダンプさせると
rx descriptor headについて
```
0x02810: RDH0        (Rx descriptor head0)            0x0000000D
0x02910: RDH1        (Rx descriptor head1)            0x00000000
0x02A10: RDH2        (Rx descriptor head2)            0x00000000
0x02B10: RDH3        (Rx descriptor head3)            0x00000001
```
のように4個分しかでない。txについても
```
0x03810: TDH0        (Transmit descriptor head0)      0x00000000
0x03910: TDH1        (Transmit descriptor head1)      0x00000008
0x03A10: TDH2        (Transmit descriptor head2)      0x00000000
0x03B10: TDH3        (Transmit descriptor head3)      0x00000000
```
となり同様。
dmesgでみると(このPCにはigbとして認識されるNICが2個口分ついている)

```
igb 0000:05:00.0: Using MSI-X interrupts. 8 rx queue(s), 8 tx queue(s)
igb 0000:05:00.1: Using MSI-X interrupts. 8 rx queue(s), 8 tx queue(s)
```
とでるのでqueueはrx, txそれぞれ8個/1 NICだと思うんだが。
と思ったらigbのデータシート
8.10.8に

> Note: In order to keep compatibility with the 82575, for queues 0-3,
> these registers are aliased to addresses 0x2810, 0x2910, 0x2A10 &
> 0x2B10 respectively.

という記述があった。0x2810などは上のethtool -gコマンドの出力結果に
でてきているアドレスなのでプログラム側でこのエイリアスを読んでいる
ということだろう。
RDHのアドレスはセクションタイトルでは
```
Receive Descriptor Head - RDH (0xC010 + 0x40*n [n=0...7]; RO)
```
となっている。

受信したパケットサイズ別ヒストグラムデータもある:
```
x0405C:  PRC64       (Packets rx (64 B) count)        0x00000010
0x04060: PRC127      (Packets rx (65-127 B) count)    0x00000020
0x04064: PRC255      (Packets rx (128-255 B) count)   0x0000000B
0x04068: PRC511      (Packets rx (256-511 B) count)   0x00000000
0x0406C: PRC1023     (Packets rx (512-1023 B) count)  0x000A5BBD
0x04070: PRC1522     (Packets rx (1024-max B) count)  0x0071F1BB

0x040AC: ROC         (Receive oversize count)         0x00000000
```
これはe1000eにはない。

ixgbeだとなぜか小文字になる:
```
0x0405C: prc64       (Packets Received (64B) Count)   0x00000014
0x04060: prc127      (Packets Rx (65-127B) Count)     0x0000001E
0x04064: prc255      (Packets Rx (128-255B) Count)    0x0000000C
0x04068: prc511      (Packets Rx (256-511B) Count)    0x000007BA
0x0406C: prc1023     (Packets Rx (512-1023B) Count)   0x0003A84C
0x04070: prc1522     (Packets Rx (1024-Max) Count)    0x00283C4E
```


