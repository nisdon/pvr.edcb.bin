# EDCB PVR Client
EpgTimerのコマンド制御確認用として作成したkodi用pvrアドンです。  
可能な確認動作としてEPG表示、EPG予約、TV視聴、そして録画の再生ができます。  

## 使用するTV録画ソフト
 - EDCB（動作確認ver work+s-221114）

## 対応OS
 - Windows
 - Raspberry Pi（Bullseye 32bit）

## 特徴
### 番組表
 - 使用チャンネルの選択、並び換えができます（「channels.xml」を使用）。
 - 使用チャンネルグループの選択ができます（「groups.xml」を使用）。
 - 局ロゴ表示はタイプ5のみ対応。

### TV視聴
- UDPのみ対応。
- 視聴用にひとつのEpgDataCap_Bonを占有します。
- 視聴を終えてもストリームは停止しません。
- PowerSaving開始時にストリームを停止。

### 番組予約
 - 予約機能はEPG予約の追加、削除のみ。
 - EPG予約追加時のオプションはデフォルト。
 - 自動予約、プログラム予約は表示のみ。

### 録画
 - 録画の保存はローカルドライブ、又はSamba接続先を想定。
 - sambaでアクセスする場合、サブディレクトリは使用でません。
 - 録画の削除時は録画ファイルも削除します。
 - 録画のサムネイル表示は出来ません。

## 設定項目
### 基本
| 項目 | 機能 |
----|----
| NWサービスにTCPを使う | OFFの場合、PIPEを使用。 |
| NWサービスのTCPアドレス | EpgTimerの使用するaddressとport。<br>例 192.168.1.101:4510 |

### 再生
| 項目 | 機能 |
----|----
| TV視聴用アドレス | UDPを受信するアダプタのaddressとport。<br>例 udp://receive_adapter:1234 |

### 番組表
| 項目 | 機能 |
----|----
| チャンネル指定 | 指定したチャンネルを使用する。 |
| チャンネルファイル | 指定チャンネルの定義ファイル。 |
| グループ指定 | 指定したグループを使用する。 |
| グループファイル | 指定グループの定義ファイル。 |

#### チャンネルファイル
```xml
<channels>
  <sid>40960</sid>
  <sid>2056</sid>
  <sid>101</sid>
  <sid>103</sid>
</channels>
```
#### グループファイル
```xml
<groups>
  <group>
	<groupName>GP1</groupName>
	<channels>
	  <sid>40960</sid>
	  <sid>2056</sid>
	</channels>
  </group>
  <group>
	<groupName>GP2</groupName>
	<channels>
	  <sid>101</sid>
	  <sid>103</sid>
	</channels>
  </group>
</groups>
```

### 録画
| 項目 | 機能 |
----|----
| 録画パスの指定 | OFFの場合、ローカルパスを使用。 |
| 録画パス | 録画ファイルが保存されている場所。<br>例 smb://user:pass@samba_addr/ |
