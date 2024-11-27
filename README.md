# EDCB PVR Client
KODIのPVRアドオンです。  

## 概要
EDCBをバッグエンドとして、KODI上でTV視聴、録画予約、そして録画再生をサポートします。

## 対応するTV録画ソフト
 - EDCB（動作確認ver work-plus-s-241008）

## 対応OS
 - Windows
 - Raspberry Pi
 - Android

## EDCB側の必須設定
 - EpgTimerSrvにネットワーク接続を許可
 - EpgTimerSrvのHTTPサーバーを有効化
 - EpgDataCap_BonにUDP、TCPによるストリーミング送信先指定
 - ユーザー定義のWeb APIを公開フォルダ/apiに配置
 - EpgTimerSrvに設定ファイルの参照を許可（CompatFlags=128 ※7bit目On）
 - EMWUIのインストール

## 設定項目
### 基本
| 項目 | 機能 |
----|----
| コマンド制御用アドレス | EpgTimerSrvにTCP接続するIPアドレスとポート。<br>例 192.168.1.101:4510 |
| ストリーミング用アドレス | ストリーミングを受信するアダプタのIPアドレス。 |
| HTTPサーバー用アドレス | 録画ライブラリ、又はWeb APIを参照するHTTPアドレス。 |

### ライブ視聴
| 項目 | 機能 |
----|----
| 使用プロトコル | ストリーミングに使用するプロトコルを選択。 |
| 使用ポート | ストリーミングを受信するポートを指定。 |
| 視聴API | ユーザー定義の視聴用APIを指定。 |
| 最大NetworkTVID数 | 使用可能なNetworkTVIDの最大数。 |
| timeshiftの使用 | KODI側のタイムシフト機能を使用。 |

#### 使用プロトコルにTCPを選択した場合
EDCBのReadme_Mod.txtより  
「ポート番号が22000～22999の範囲では送信形式がplain(BonDriver_TCP.dllのための特別なヘッダをつけない)になります。」

#### 使用プロトコルにSrvPipeを選択した場合
視聴用APIを使用して制御したストリーム配信が可能です。  
APIへのパラメータとしてプロセスIDを渡します。

```lua
-- SrvPipe取得
dofile(mg.script_name:gsub('[^\\/]*$','')..'util.lua')
query=mg.request_info.query_string
pid=mg.get_var(query,'pid')

for i=1,50 do
  ff=edcb.FindFile(SendTSTCPPipePath('*_'..pid,0),1)
  if ff and ff[1].name:match('^[^_]+_%d+_%d+') then
    pipeName='/var/local/edcb/'..ff[1].name
    break
  end
  edcb.Sleep(200)
end

-- 以下、トランスコード等の処理を追加
```

#### 最大NetworkTVID数は
使用するチューナーのチャンネル数や接続するクライアント数によって決定します。

#### タイムシフト機能を使用するには
inputstream.ffmpegdirectアドオンが必要です。

### 録画再生
| 項目 | 機能 |
----|----
| 再生方法 | 公開フォルダのライブラリ、又はストリーミング再生を選択。 |
| 使用プロトコル | ストリーミングに使用するプロトコルを選択。 |
| 使用ポート | ストリーミングを受信するポートを指定。 |
| エンコード用API | ユーザー定義のエンコード用APIを指定。 |
| 録画サムネイル指定 | EMWUIの使用する画像、又はユーザー画像のいずれか選択。 |

#### 追っかけ再生は
「再生方法」の設定に関係なくストリーミング再生になります。

#### 録画再生のシークは
公開フォルダのライブラリ使用時のみ可能です。

#### 録画サムネイルにユーザー画像を用いる場合
録画ファイルと同じ場所に「録画ファイル名 + .ts.jpg」で保存します。

#### エンコード用APIは
APIへのパラメータとして録画済み情報IDを渡します。

```lua
-- 録画ファイル取得
dofile(mg.script_name:gsub('[^\\/]*$','')..'util.lua')
query=mg.request_info.query_string
recFile=GetFilePath(query)

-- 以下、エンコード等の処理を追加
```

### 番組表
| 項目 | 機能 |
----|----
| チャンネル指定 | 使用するチャンネル定義を有効にする。 |
| チャンネルファイル | 使用するチャンネルを定義したxml。 |
| グループ指定 | 使用するチャンネルのグループ定義を有効にする。 |
| グループファイル | 使用するチャンネルのグループを定義したxml。 |

#### 例 channels.xml
使用するチャンネルの選択、並び換えができます。  

```xml
<channels>
  <sid>40960</sid>
  <sid>2056</sid>
  <sid>101</sid>
  <sid>103</sid>
</channels>
```
#### 例 groups.xml
使用するチャンネルをグループ化できます。

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

## コンテキストメニュー追加項目
### 録画
| 項目 | 機能 |
----|----
| ドロップログの表示 | .errファイルを表示。 |
| エンコード | エンコード用APIを実行します。 |

## その他
### 録画設定は
デフォルトのプリセットを選択。  
ワンセグ設定は無視します。  

### ルールは
自動予約の表示のみ対応。

### 局ロゴの表示は
EMWUIのapi/logoを使用。

### 録画コンテキストメニューの「録画情報」は
.program.txtファイルを表示。

## 既知のバグ
### ライブ視聴中に録画開始等でチューナー数がオーバーした場合
録画チャンネルが空きになるまで視聴用のチャンネルを取得できません。

### FireTV 32bit版を使用した場合
視聴停止しても配信停止（EpgDataCap_Bonが終了）しない。  
※チューナーチャンネル取得時に停止処理を追加して対処済み。
- [チャンネルを停止してもEDCBが終了しない](https://github.com/nisdon/pvr.edcb.bin/issues/4)
