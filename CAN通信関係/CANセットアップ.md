# CANable のセットアップと送受信テスト

車輪IDとデータ形式は [CANプロトコル](CANプロトコル.md) を参照してください。

## 1. USB 接続と CAN の設定

1. CANable を Ubuntu PC に USB 接続します。CAN_H 同士、CAN_L 同士を接続し、非絶縁の構成では信号GNDも共有します。バスの物理的な両端に120 Ω終端を1つずつ置きます。CANable側に終端スイッチがあり、そこが端点なら既設の終端と重複しないよう設定します。
2. マイコンとCANableをいずれも **Classic CAN・1 Mbit/s** に設定します。フレームは **29ビット拡張ID** を使います。
3. Ubuntu に送受信用ツールを入れ、CANable がどちらの形式で認識されているか確認します。

```bash
sudo apt install can-utils
ip -brief link
ls /dev/serial/by-id/ /dev/ttyACM*
```

### Candlelight の場合

USB接続時に `can0` などの名前が出ます。表示された名前で設定します。

```bash
sudo ip link set can0 down
sudo ip link set can0 type can bitrate 1000000
sudo ip link set can0 up
ip -details -statistics link show can0
```

### SLCAN の場合

USBシリアルとして表示されます。実際のデバイス名を使います。`-s8` は1 Mbit/sです。

```bash
sudo slcand -o -c -s8 /dev/ttyACM0 slcan0
sudo ip link set slcan0 up
```

以下のコマンドでは `can0` を使用します。SLCANで設定した場合は `slcan0` に置き換えてください。

## 2. 最初の送受信テスト

駆動を無効にし、マイコン側で受信ID、拡張フレームかどうか、DLC、データをログに出せるようにします。PCのターミナルを2つ開き、片方で受信ログを表示します。

```bash
candump can0
```

先に物理バスの外でIDとデータだけを確認する場合は、仮想CANを使用できます。

```bash
sudo modprobe vcan
sudo ip link add dev vcan0 type vcan
sudo ip link set vcan0 up
```

片方で `candump vcan0`、もう片方で `cansend vcan0 1A000F01#A5` を実行します。仮想CANでは送信フレームを確認できます。実バスでは、マイコンに診断応答を実装してから、基板0に次の通信確認フレームを送ります。

```bash
cansend can0 1A000F01#A5
```

8桁のIDを指定すると拡張フレームになります。`candump` でマイコンからの応答 `1B000F01 [1] A5` を確認します。要求IDだけが表示された場合は、マイコン側の受信・応答を確認してください。届かない場合は、インターフェース名、配線、終端、1 Mbit/s、有効な受信ノード、マイコンの拡張IDフィルターの順に確認します。Candlelightでは `ip -details -statistics link show can0` でエラーやBUS-OFFを確認できます。

## 参考

- [CANable公式セットアップ](https://canable.io/getting-started.html)
- [Linux CANのコマンド群 can-utils](https://github.com/linux-can/can-utils)
