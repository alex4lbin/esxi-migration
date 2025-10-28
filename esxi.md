1. VSS の状態を確認する

```bash
esxcli network vswitch standard list
```

出力例

```
vSwitch0
   Name: vSwitch0
   Class: cswitch
   Num Ports: 1590
   Used Ports: 4
   Configured Ports: 128
   MTU: 1500
   CDP Status: listen
   Beacon Enabled: false
   Beacon Interval: 1
   Beacon Threshold: 3
   Beacon Required By:
   Uplinks:
   Portgroups: MGMT
```

2. vSwitch0 が存在していれば削除する

```bash
esxcli network vswitch standard remove -v vSwitch0
```

3. 新たに vSwitch0 を作成する

```bash
esxcli network vswitch standard add -v vSwitch0
```

4. MGMT ポートグループを vSwitch0 に追加する

```bash
esxcli network vswitch standard portgroup add -v vSwitch0 -p MGMT
```

5. MGMT ポートグループに VLAN を設定する

```bash
esxcli network vswitch standard portgroup set -p MGMT -v {VLAN ID}
```

6. VDS 状態を確認する。VDS 名、対象 uplink とその Port ID をメモしておく。

```bash
esxcli network vswitch dvs vmware list
```

出力例

```
TEST
   Name: TEST
   VDS ID: 49 19 f7 9c cd ac 3a 6f-92 ab 12 53 31 b6 2e 97
   Class: cswitch
   Num Ports: 3200
   Used Ports: 16
   Configured Ports: 512
   MTU: 1500
   CDP Status: advertise
   Beacon Timeout: -1
   Uplinks: vmnic0
   VMware Branded: true
   DVPort:
         Client: vmnic0
         DVPortgroup ID: dvportgroup-2003
         In Use: true
         Port ID: 8
```

7. DVS から uplink を削除する。

```bash
esxcfg-vswitch -Q {uplink} -V {Port ID} {VDS name}
```

8. 作成した vSwitch0 に uplink を追加する。

```bash
esxcli network vswitch standard uplink add -u {uplink} -v vSwitch0
```

9. vmk0 を削除する。

```bash
esxcli network ip interface remove -i vmk0
```

10. vmk0 を作成し、vSwitch0 の MGMT ポートグループに追加する。

```bash
esxcli network ip interface add -i vmk0 -p MGMT
```

11. vmk0 に IP アドレスと DFG を設定する

```bash
esxcli network ip interface ipv4 set -i vmk0 -I {IP address} -N {netmask} -g {gateway} -t static
```

12. vmk0 を使ったデフォルトルートを設定する。

```bash
esxcli network ip route ipv4 add -g {gateway} -n default
```

### その他確認コマンド

- ポートグループ状態

```bash
esxcli network vswitch standard portgroup list
```

- IP アドレス設定

```bash
esxcli network ip interface ipv4 get
```

- ルーティング設定

```bash
esxcli network ip route ipv4 list
```

- NIC 状態

```bash
esxcli network nic list
```

### VSAN 関連

- クラスター状態確認

```
esxcli vsan cluster get
```

- クラスターから解除

```
esxcli vsan cluster leave
```

- クラスターへ追加

```
esxcli vsan cluster join -u <UUID>
```
