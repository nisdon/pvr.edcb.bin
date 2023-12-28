# EDCB PVR Client
KODIのPVRアドオンです。

## 概要
EDCBをバッグエンドとして、KODI上でTV視聴、録画予約、そして録画再生をサポートします。

## 使用するTV録画ソフト
 - EDCB（動作確認ver work+s-221114）

## 対応OS
 - Windows
 - Raspberry Pi
 - Android

## 設定項目
### 基本
| 項目 | 機能 |
----|----
| NWサービスにTCPを使う | PIPEを使用する場合はOffにします。 |
| NWサービスのTCPアドレス | EpgTimerのaddressとport。<br>例 192.168.1.101:4510 |
| timeshiftを使う | TV視聴時にタイムシフト機能を有効にします。 |

#### NWサービスの使用
EpgTimer側で通信を許可する必要があります。

#### PIPEを用いたNWサービス
Windows PC内でのプロセス間通信のみです。

#### タイムシフト機能を使用するには
inputstream.ffmpegdirectアドオンが必要です。

### 再生
| 項目 | 機能 |
----|----
| TV視聴の種類 | UDP、TCP又はHTTPを選択。 |
| UDP アドレス | UDPを受信するアダプタのaddressとport。 |
| TCP アドレス | TCPを受信するアダプタのaddressとport。 |
| HTTP アドレス | HTTPを送信するサーバーのaddressとport。<br>例 http://server:5510/api/TVCast?onid=%d&tsid=%d&sid=%d |

#### UDP、TCPを使用するには
EpgDataCap_Bon側で送信先の設定が必要です。  

#### UDP、TCPのアドレス
受信アダプタを固定する場合に指定します。

#### HTTPを使用するには
TVキャスト用のREST APIが必要です。  
以下はEDCB_Material_WebUIのTVCastを参考にしたサンプルです。  

```lua
function readts(pname)
  f=edcb.io.open(pname, 'rb')

  if not f then
    mg.write('HTTP/1.1 404 Not Found\r\nConnection: close\r\n\r\n')
  else
    mg.write('HTTP/1.1 200 OK\r\nContent-Type: '..mg.get_mime_type('viewts')..'\r\nContent-Disposition: filename=viewts\r\nConnection: close\r\n\r\n')
    while true do
      buf=f:read(48128)
      if buf and #buf ~= 0 then
        if not mg.write(buf) then
          -- キャンセルされた
          mg.cry('canceled')
          break
        end
      else
        -- 終端に達した
        mg.cry('end')
        break
      end
    end
    f:close()
  end
end

ok,pid=edcb.IsOpenNetworkTV(n)
if ok then
  -- 開いているNetworkTVモードを使用
  pname='\\\\.\\pipe\\SendTSTCP_'..n..'_'..pid, 'rb'
  readtsa(pname)
  edcb.CloseNetworkTV(n)
else
  -- 新しくNetworkTVモードを開く
  if onid and tsid and sid then
    ok,pid=edcb.OpenNetworkTV(mode, onid, tsid, sid, n)
    retry=0
    while true do
      -- 最大5秒間パイプの準備待ち
      retry=retry+1
      ff=edcb.FindFile('\\\\.\\pipe\\SendTSTCP_'..n..'_'..pid, 1)
      if ff or retry > 25 then break end
      edcb.Sleep(200)
    end
  end
  if ff and ok then
    pname='\\\\.\\pipe\\'..ff[1].name, 'rb'
    readts(pname)
  end
end
```

### 番組表
| 項目 | 機能 |
----|----
| チャンネル指定 | 使用チャンネルを指定する。 |
| チャンネルファイル | 使用チャンネルを定義したxml。 |
| グループ指定 | 使用グループを指定する。 |
| グループファイル | 使用グループを定義したxml。 |

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
使用チャンネルグループの選択ができます。  

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
| 録画先の種類 | ローカルフォルダ、又は共有フォルダを選択。 |
| 共有フォルダのユーザー | ログインするユーザーとパスワード。 |

#### 録画先の種類として
ローカルネットワーク内で使用する場合は共有フォルダを選択します。

#### 共有フォルダのパスを取得するために
EpgTimerSrv.iniにCompatFlagsを設定します。  
CompatFlags=16（4bit目をOn）

## コンテキストメニュー項目
### 録画
| 項目 | 機能 |
----|----
| ドロップログの表示 | 録画ステータスとドロップ数を表示。 |

## ルールは
自動予約の表示のみ対応。

