# EDCB PVR Client
KODIのPVRアドオンです。  

## 概要
EDCBをバッグエンドとして、KODI上でTV視聴、録画予約、そして録画再生をサポートします。

## 対応するTV録画ソフト
 - EDCB（動作確認ver work-plus-s-250321）

## 対応OS
 - Windows
 - Raspberry Pi
 - Android

## EDCB側の必須設定
 - EpgTimerSrvのネットワーク接続を許可
 - EpgTimerSrvのHTTPサーバーを有効化
 - EpgTimerSrvの録画動作からドロップログを出力に設定
 - EpgTimerSrvの録画動作から番組情報を出力に設定（Windows版はUTF-8にする）
 - EpgDataCap_Bonのネットワーク設定からTCP送信先にSrvPipeを追加[^録画ストリーミングのポート問題]
 - *EpgTimerSrv.ini*にCompatFlags=128を追加（7bit目ON）[^tkntrec版互換設定]

## 設定項目
### 基本/アドレス定義
| 項目 | 機能 |
----|----
| コマンド制御用アドレス | EpgTimerSrvにTCP接続するIPアドレスとポート。<br>例 192.168.1.101:4510 |
| HTTPサーバー用アドレス | 録画ライブラリ、又はWeb APIを参照するHTTPアドレス。 |

### 基本/視聴方法
| 項目 | 機能 |
----|----
| TV視聴 | ストリーミング、又はWeb APIの使用を選択。 |
| TV視聴用Web API | サーバー側に配置したWeb APIを指定。 |
| 録画視聴 | ストリーミング、又は公開フォルダからの再生を選択。 |

#### TV視聴用Web APIにより
サーバー側でストリームを処理して配信します。  
Web APIへのパラメータとしてプロセスIDを渡します。  
[Web API視聴設定の補足](https://github.com/nisdon/pvr.edcb.bin/issues/4)

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

-- 以下、配信処理を追加
```

#### 録画視聴時におけるシーク等のファイル操作は
公開フォルダ使用時のみ可能です。

### その他
| 項目 | 機能 |
----|----
| エンコード用Web API | サーバー側に配置したWeb APIを指定。 |
| 録画サムネイル指定 | EMWUIの生成する画像、又はユーザー画像を選択。 |
| 録画のジャンル別表示 | ジャンル／タイトルとして階層表示。 |

#### エンコード用Web APIにより
録画ファイルのエンコードができます。  
パラメータとして録画済み情報IDを渡します。

```lua
-- 録画ファイル取得
dofile(mg.script_name:gsub('[^\\/]*$','')..'util.lua')
query=mg.request_info.query_string
recFile=GetFilePath(query)

-- 以下、エンコード等の処理を追加
```

#### 録画サムネイルにユーザー画像を用いる場合
録画ファイルと同じ場所に 録画ファイル名+*.ts.jpg* で保存します。

## コンテキストメニュー追加項目
### 録画
| 項目 | 機能 |
----|----
| ドロップログの表示 | *.err*ファイルを表示。 |
| エンコード | エンコード用Web APIを実行します。 |

### タイマー/ルール
| 項目 | 機能 |
----|----
| プリセット変更 | 録画予約のプリセットを変更します。 |

#### 扱えるプリセットの数は
最大10個まで。

### クライアント固有の設定
| 項目 | 機能 |
----|----
| プロセスの表示 | 稼動中のチューナー情報を表示します。 |

### 追っかけ再生
録画中のTVチャンネルを視聴する際に選択できます。  
又、視聴方法の選択に関係なくストリーミング再生します。

### 録画予約は
ワンセグ設定を無視します。

### 局ロゴの取得は
*HttpPublic/legacy/logo.lua*を使用。

### 録画の番組情報は
*.program.txt*ファイルを表示します。

### TVチューナーは
最大8チャンネル（NetworkTVID）を割り当て可能。

[^録画ストリーミングのポート問題]: EpgDataCap_BonのTCP送信先にTCPを追加すると、録画/追っかけ再生のストリーミング用ポートに干渉します。
[^tkntrec版互換設定]: ここでは設定ファイルの参照を可能にしています。
