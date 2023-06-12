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
