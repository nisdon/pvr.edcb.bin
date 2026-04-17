# EDCB PVR Client
マルチメディアセンターアプリKODIのPVRアドオンです。

## 概要
TV録画サーバーEDCBをバッグエンドとして、KODI上でTV視聴、録画予約、そして録画再生をサポートします。

## 動作環境
### 対応OS
 - Windows
 - Raspberry Pi
 - Android

### プラットフォーム別のモジュール
環境に対応したファイルを選んでください。

| プラットフォーム | モジュール |
----|----
| Windows 64bit | pvr.edcb.dll（default） |
| Raspberry Pi 64bit | pvr.edcb.so（default） |
| Raspberry Pi 32bit | pvr.edcb.pi.v7a.so |
| Android 64bit | pvr.edcb.android.so（default） |
| Android 32bit | pvr.edcb.android.v7a.so |

> [!TIP]
> default以外の環境では、*addon.xml*に定義されたファイル名を変更するか、ファイル名をdefaultにリネームします。

### EDCB側の必須設定
 - EpgTimerSrvのネットワーク接続を許可.
 - EpgTimerSrvのHTTPサーバーを有効化.
 - EpgTimerSrvの録画動作からドロップログを出力に設定.
 - EpgTimerSrvの録画動作から番組情報を出力に設定（Windows版はUTF-8にする）.
 - EpgTimerSrvの予約情報管理から録画ファイル削除を許可（録画削除時にファイルも削除する場合）.
 - EpgTimerSrvのEPG取得対象サービスから使用するチャンネルを選択.
 - EpgDataCap_Bonのネットワーク設定からTCP送信先にSrvPipeを追加.
 - *EpgTimerSrv.ini*にCompatFlags=128を追加（7bit目ON）.

> [!NOTE]
> 動作確認済みのEDCBバージョンは **xtne6f版work-plus-s-250321** です。

> [!NOTE]
> tkntrec互換フラグCompatFlagsの7bit目ONにより設定ファイルのアクセスを許可しています。

## 設定項目
### 基本/アドレス定義
| 項目 | 機能 |
----|----
| コマンド制御用アドレス | EpgTimerSrvにTCP接続するIPアドレスとポート。<br>例 192.168.1.101:4510 |
| HTTPサーバー用アドレス | EpgTimerSrvにHTTP接続するIPアドレスとポート。<br>例 http://192.168.1.101:5510 |

### 基本/視聴方法
| 項目 | 機能 |
----|----
| TV視聴 | ストリーミング、又はWeb APIの使用を選択。 |
| TV視聴用Web API | サーバー側に配置したWeb APIを指定。 |
| 録画視聴 | KODIのビデオソース、又は公開フォルダ内の録画フォルダ参照を選択。 |
| ビデオソース名 | KODIに追加したビデオソースの名前。 |

> [!TIP]
> TV視聴時にデュアルモノ音声変換やトランスコードを必要とする場合はWeb APIを使用します。
> ```lua
> -- 引数 pid:プロセスID
> -- SrvPipe取得
> dofile(mg.script_name:gsub('[^\\/]*$','')..'util.lua')
> query=mg.request_info.query_string
> pid=mg.get_var(query,'pid')
>
> for i=1,50 do
>   ff=edcb.FindFile(SendTSTCPPipePath('*_'..pid,0),1)
>   if ff and ff[1].name:match('^[^_]+_%d+_%d+') then
>     pipeName='/var/local/edcb/'..ff[1].name
>     break
>   end
>   edcb.Sleep(200)
> end
>
> -- 以下、配信処理を追加
> ```
> [Web API視聴設定の補足](https://github.com/nisdon/pvr.edcb.bin/issues/4)

> [!TIP]
> KODIでは扱う動画の場所をビデオソースとして設定します。  
> 録画ファイルの場所をビデオソースにする事でローカルディスクやネットワークディスク（SMB、NFS、FTP、HTTP等）にアクセスできます。

> [!NOTE]
> 公開フォルダはEDCBのHTTPサーバーで公開される場所（デフォルトではHttpPublicFolder配下）です。

> [!NOTE]
> 録画のサムネイルとして表示される画像は録画視聴の方法により異なります。  
> KODIのビデオソースから視聴する場合、KODIのビデオソースで表示されるサムネイルを使用。  
> 公開フォルダから視聴する場合、EMWUIで表示されるサムネイルを使用。

### その他
| 項目 | 機能 |
----|----
| エンコード用Web API | サーバー側に配置したWeb APIを指定。 |
| エンコード用引数リスト | 0から始まる引数の名前を定義。<br>例 360p_h264,720p_h264 |
| 録画フォルダのサムネイル | ユーザーが用意したサムネイルを使用。 |
| 録画のジャンル別表示 | ジャンル／タイトルとして階層表示。 |

> [!TIP]
> 録画ファイルをエンコードするにはWeb APIを使用します。
> ```lua
> -- 引数 id:録画ID, param:任意の値
> -- 録画ファイル取得
> dofile(mg.script_name:gsub('[^\\/]*$','')..'util.lua')
> query=mg.request_info.query_string
> id=mg.get_var(query,'id')
> info=edcb.GetRecFileInfoBasic(id)
> recfile=info.recFilePath
>
> -- 以下、エンコード等の処理を追加
> ```

> [!TIP]
> 録画フォルダのサムネイルを使用する場合、録画ファイルと同じ場所に *RECORD_FILE_NAME.ts.jpg* で保存します。  
> 録画視聴の方法に関係無く当該サムネイルの表示が優先されます。

## コンテキストメニューの追加項目
### 録画
| 項目 | 機能 |
----|----
| ドロップログの表示 | 録画情報、及び*.err*ファイルを表示。 |
| エンコード | エンコード用Web APIを実行します。 |

### タイマー/ルール
| 項目 | 機能 |
----|----
| プリセット変更 | 録画予約のプリセットを変更します。 |

> [!WARNING]
> 扱えるプリセットの数は最大10個まで。

### クライアント固有の設定
| 項目 | 機能 |
----|----
| プロセスの表示 | 稼動中のチューナー情報を表示します。 |

### チャンネル（録画中）
| 項目 | 機能 |
----|----
| ライブ視聴 | 当該チャンネルのライブ視聴。 |
| 追っかけ再生 | 録画中チャンネルを再生。 |

> [!NOTE]
> 録画中のチャンネルを視聴する際、コンテキストメニューを表示。
