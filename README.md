# X68000キーボードコネクタにUSBキーボードとUSBマウスを接続します

## おことわり
自分が普段使用しているキーボードやマウスでテストを行っているため、すべてのUSB機器に対して稼働確認が取れているわけではありません。相性や根性が足りない場合動かないものもありますのでご了承ください。

## 必要機材(例)
* Arduino Uno + USB HOSTシールド
* Arduino pro mini(5V) + USB HOSTシールド(pro mini 5V用)
* Arduino pro mini(3.3V) + USB HOSTシールド(pro mini 3.3v用) + レベルコンバータ

X68000本体から5Vが流れてくるのでArduino側も5Vに合わせる必要があります。
3.3V版arduinoを使用の際はレベルコンバータを使用し5Vにします。

## テスト環境
* ELECOM TK-FDM109MBK(2.4GHz無線キーボード／マウス)
* ELECOM TK-FDM086MBK(2.4GHz無線キーボード／マウス)
* SANWA SUPPLY USB-CVP22(USB-PS/2コンバータケーブル) + IBM RT3200
* Logicool M171(ワイヤレスマウス)
* ELECOM M-BT12BR(Bluetoothマウス)
* Lenovo SK-8827

## 使えなかったもの
* HHKB Lite2(USBハブを内蔵しているためUSBHOSTのライブラリで具合悪いらしい)

## 配線図
![](images/keyboard_connector.png)
```
//  本体側             Arduino側
//  -------------------------------------------
//  1:Vcc2 5V(out) -> 5V
//  2:MSDATA(out)  <- TX(1)
//  3:KEYRxD(in)   <- A0(14) softwareSerial TX 
//  4:KEYTxD(out)  -> A1(15) softwareSerial RX
//  5:READY(out)
//  6:REMOTE(in)
//  7:GND(--)      -- GND
```

## キーアサイン
109キーボードではすべてのキーをアサインすることができません。なので使用頻度の低いキーをアサインしていません。
もし変更したいときはArduinoのソースを修正するか、[KeyWitch.x](http://retropc.net/x68000/software/system/key/keywitch/)等ソフトウェアでキーアサインを変更できるソフトを使用してください。

私はSted2でリズム画面に入るため記号入力や登録キーを押す必要がありますが、Keywitchでsted2を使用するときだけ「BREAK」と「COPY」を「記号入力」と「登録」に入れ替えて使用しています。

日本語109キーボードアサイン例 `赤字部分がX68000のキーアサイン`
![](images/layout109.png)

キートップにテプラを貼りました
![](images/109.jpg)

#### キーアサイン初期状態
```
//  ・F11      -> かな(0x5a)
//  ・F12      -> ローマ字(0x5b)
//  ・LeftWin  -> ひらがな(0x5f)
//  ・LeftAlt  -> XF1(0x55)
//  ・無変換    -> XF2(0x56)
//  ・変換      -> XF3(0x57)
//  ・カタカナ   -> XF4(0x58)
//  ・RightAlt  -> XF5(0x59)
//  ・RgihtWin  -> N/A
//  ・Menu      -> OPT.1(0x72)
//  ・RightCtrl -> OPT.2(0x73)
//  ・END       -> UNDO(0x3a)
//  ・ScrollLock-> HELP(0x54)
//  ・Pause     -> BREAK(0x61)
//  ・PrintScr  -> COPY(0x62)
//  ・NumLock   -> CLR(0x3f)
//  ・Num /     -> 記号入力(0x52)
//  ・Num *     -> 登録(0x53)
//  ・Num -     -> コード入力(0x5c)
```

※記号入力/登録/コード入力をテンキーの一部にアサインしました。

## Arduino Pro Mini用のUSBHOSTを使用したバージョンの作成方法

### USBHOSTの加工
参考ページ:https://ht-deko.com/arduino/shield_usbhost_mini.html

__カット部分__
![](images/cut.jpg)
__テスターでチェック__
ここが通電しなければOK
![](images/DSC04025.jpg)

### Arduinoの足付け
そのまま足を付けると斜めになるので、下にUSBHOSTを敷いてはんだ付けすると良いよ
![](images/DSC04026.jpg)
![](images/DSC04027.jpg)

USB端子とFTDI端子が反対になるように合わせる
![](images/DSC04030.jpg)

USBの5VをRAWをつなげるためのケーブル(約43mm)
![](images/DSC04032.jpg)
裏側で結線する
![](images/DSC04034.jpg)

レベルコンバータを両面テープで絶縁して貼り付ける
![](images/DSC04035.jpg)
![](images/DSC04037.jpg)

A0とA1とVCCの足を曲げてLV4,LV3,LVに接続する。
A2とA3は邪魔なので根本からカット
![](images/DSC04040.jpg)

GND(黒)の接続とTXとLV1(白)を接続する
![](images/DSC04041.jpg)

`概ね終了。あとはMiniDin7pとの接続`

実際に接続するのは5本
![](images/pin.jpg)

### PS/2ケーブルの加工
キーボードコネクタは7ピンですが6ピン目のREMOTEはこの変換機では使用していないのでMiniDIN6pケーブルを加工して使用できます。また中央にあるプラスティックの足はニッパーなどで切断することで7ピン端子に刺すことが可能になります。

[カモンのPSK-18](https://www.sengoku.co.jp/mod/sgk_cart/detail.php?code=4AC4-DTEN)なんてお手頃です。1本から2つ採取できます。

![](images/miniDIN7pin_male.jpg)

### 完成
![](images/kansei.jpg)
![](images/kansei2.jpg)

## キーリピート間隔の設定(Ver.0.2)
Ver.0.2よりswitch.xで設定したキーリピート間隔の設定値を反映させました。
計算式は「30ms + 設定値^2 * 5ms」なので、設定値2だと、30+2^2*5=50msになります。
しかし実機キーボードとは挙動が異なるため、同じようにはなりません（でした

## USBマウスと本体マウスの同時使用について(Ver.0.3)
X68000の純正キーボード側面にマウスを接続することでマウスも使用することが出来ます。ようするにキーボードコネクタから
マウスの通信に必要なデータも流れています。
UDBKBD2X68Kではキーボードとマウス両方使用できるようにマウスの通信処理もコーディングしていますが、
実際にマウスが繋がっていなくても、マウスがあることになってしまうため、USBKBD2X68Kを接続すると本体に接続したマウスが動かなくなる事象が報告されました。
マウスが繋がっているかどうかの判定をロジックで行うことが出来なかったため（調査中です）、~~「MOUSE_MODE」のフラグを設けました。
MOUSE_MODEを1にすることでキーボードとマウス両方使用できます。0にすることでマウスは使わない（本体マウスが有効）になります。~~ `Ver.0.4にて対応`

## マウスモードOFF機能(Ver.0.4)
このアダプタを接続するとマウスポートに接続したマウスを使うことが出来なくなります。
D2とGNDをショートさせることで、アダプタのマウス機能をOFFにすることが出来、マウスポートに刺したマウスが使うことが出来るようになります。
![](images/mouse_mode.jpg)


## USB-HUB使用時の挙動（半分諦め）
USBハブに刺した順番に初期化処理を行うようです。キーボードを2つ接続すると先に認識したほうしか入力できない？
2021.2.14追記 USBハブ経由での接続の挙動が良くわからないため、正式対応を宣言しません。

## Bluetoothの挙動（サポート対象外）
BluetoothキーボードとBluetoothマウスも使用可能です。
2021.2.14追記 ロジックは入っているので使用できるかもしれませんが、ペアリング等課題があるためサポート対象外にします。


## 実装していないコト
* マウスカーソルのオーバーフロー処理
* マウスの速度調整変更

## 履歴
* 2020.11.09 Ver.0.1	 とりあえず動いた版
* 2020.12.30 Ver.0.2   キーリピート間隔の実装（しかし実機と挙動が異なる）
* 2021.01.02 Ver.0.3   MOUSE_MODEでマウスを使用しないときはOFF出来るようにした
* 2021.02.14 Ver.0.4   MOUSE_MODE廃止,MOUSE CONTROLを実装,記号入力/登録/コード入力をテンキーに割り当て

## 謝意
USBキーボードをX68000に繋げるネタ、情報提供、共同作成したzα2さん、ありがとうございます。

技術的情報提供、相談に乗ってもらいましたハルぴこさん、ありがとうございます。

## 父のパソコンを超えろ
