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

## 設定項目
### 基本
| 項目 | 機能 |
----|----
| コマンド制御用アドレス | EpgTimerSrvに接続するIPアドレスとポート。<br>例 192.168.1.101:4510 |
| ストリーミング用アドレス | ストリーミングを受信するアダプタのIPアドレス。 |
| HTTPサーバー用アドレス | 公開ディレクトリ、又はWeb APIを参照するアドレス。 |
| 録画サムネイル指定 | EMWUIの使用する画像、又はユーザー定義画像のいずれか選択。 |

#### コマンド制御するために
EpgTimerSrv側で「ネットワーク接続を許可する」の設定が必要です。

#### ストリーミング用のアドレスは
UDP、又はTCPで共通です。

#### 録画サムネイルをユーザーが用意する場合
ファイルはHTTP 公開ディレクトリになければ使用できません。  
録画ファイルと同じ場所に「録画ファイル名 + .ts.jpg」で保存します。

### ライブ視聴
| 項目 | 機能 |
----|----
| 使用プロトコル | EpgDataCap_Bonに接続するプロトコルを選択。 |
| 使用ポート | ストリーミングを受信するポート。 |
| 視聴API | ユーザー定義の視聴用APIを指定。 |
| 最大NetworkTVID数 | 使用可能なNetworkTVIDの最大数。 |
| timeshiftの使用 | KODI側のタイムシフト機能を使用。 |

#### ストリーミングを受信するため
EpgDataCap_Bon側で「ネットワーク設定」の設定が必要です。

#### TCP使用時のポートについて
EDCBのReadme_Mod.txtより  
「ポート番号が22000～22999の範囲では送信形式がplain(BonDriver_TCP.dllのための特別なヘッダをつけない)になります。」

#### 視聴用APIを使用するには
「HTTP 公開ディレクトリ/api」にWeb APIを配置します。  
使用することでストリームを制御することが可能です。  
パラメータとしてSrvPipe作成時のプロセスIDを渡します。

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
| 使用プロトコル | EpgDataCap_Bonに接続するプロトコルを選択。 |
| 使用ポート | ストリーミングを受信するポート。 |
| HTTP 公開ディレクトリ | HTTPサーバーで公開された録画ファイルのディレクトリを指定。 |

#### HTTPサーバーを使用する場合
公開ディレクトリ内のサブディレクトリから録画ファイルにアクセス出来ません。  
ファイル名に半角文字を含む場合は禁則文字に注意が必要です。

#### 録画再生のシークは
HTTP使用時でのみ可能です。

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

## コンテキストメニュー項目
### 録画
| 項目 | 機能 |
----|----
| ドロップログの表示 | 録画ステータスとドロップ数を表示。 |
| エンコード | ユーザー定義のエンコード用APIを指定。 |

#### エンコード用APIを使用するには
「HTTP 公開ディレクトリ/api」にWeb APIを配置します。  
使用することでエンコード等が可能です。  
パラメータとして録画済み情報IDを渡します。

```lua
-- 録画ファイル取得
dofile(mg.script_name:gsub('[^\\/]*$','')..'util.lua')
query=mg.request_info.query_string
recFile=GetFilePath(query)

-- 以下、エンコード等の処理を追加
```

## その他
### 録画設定は
デフォルトのプリセットを選択。  
ワンセグ設定は無視します。  
プリセット情報を読み込むためEpgTimerSrv.iniに下記設定が必要です。  
CompatFlags=128（7bit目をOn）

### ルールは
自動予約の表示のみ対応。

### 局ロゴは
タイプ5のみサポート。

## 既知のバグ
チューナーのチャンネル数オーバーにより視聴中のチャンネルが録画開始等で取られた場合  
チャンネルが空きになるまで視聴用にチャンネルが取得できません。
