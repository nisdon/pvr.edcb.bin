# EDCB PVR Client
EpgTimerのコマンド制御確認用として作成したkodi用pvrアドンです。  
可能な確認動作としてEPG表示、EPG予約、TV視聴、そして録画の再生ができます。  

## 使用するTV録画ソフト
 - EDCB（動作確認ver work+s-221114）

## 対応OS
 - Windows
 - Raspberry Pi
 - Android

## その他
 - 予約機能はEPG予約の追加、削除のみ対応。
 - EPG予約追加時のオプションはデフォルト設定を使用。
 - 自動予約、プログラム予約は表示のみ対応。
 - 局ロゴの表示はタイプ5を使用。

## 設定項目
### 基本
| 項目 | 機能 |
----|----
| NWサービスにTCPを使う | OFFの場合、PIPEを使用。 |
| NWサービスのTCPアドレス | EpgTimerの使用するaddressとport。<br>例 192.168.1.101:4510 |
| timeshiftを使う | timeshiftの使用を切替えます。 |

#### NWサービスのPIPE使用
ローカルPCに限りアクセス可能。

### 再生
| 項目 | 機能 |
----|----
| TV視聴の種類 | UDP、TCP又はHTTPを選択。 |
| UDP アドレス | UDPを受信するアダプタのaddressとport。<br>例 udp://receive_adapter:1234 |
| TCP アドレス | TCPを送信するアダプタのaddressとport。<br>例 tcp://receive_adapter:22000 |
| HTTP アドレス | HTTPを送信するサーバーのaddressとport。<br>例 http://server:5510/api/TVCast?onid=%d&tsid=%d&sid=%d |

#### UDP、TCPの使用
NetworkTVモードにより視聴します。  
開始されたNetworkTVモードはPowerSaving発動時に停止します。  
timeshift使用時はNetworkTVモードは停止しません。  

#### HTTPの使用
HTTPでアクセスするには別途TVキャスト用のREST APIが必要です。  
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
| 録画パスの種類 | ローカルフォルダ、又は共有フォルダを選択。 |
| 共有フォルダのユーザー | 使用するユーザーとパスワード。 |

#### 「Shared folder」の使用
EpgTimerSrv.iniにCompatFlagsを定義する必要があります。  
CompatFlags=16（4bit目をON）  

## コンテキストメニュー項目
### 録画
| 項目 | 機能 |
----|----
| ドロップログの表示 | 録画ステータスとドロップ数を表示。 |

