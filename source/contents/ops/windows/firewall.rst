========
firewall
========

ファイアウォールの初期化と復元
================================

ファイアウォールの設定をエクスポートする。

.. attention::

   バイナリ形式のため設定内容を確認することはできません。

.. code-block:: bat

   netsh advfirewall export "C:/temp/firewall-dump.wfw"

ファイアウォールの設定をリセットする。

- ユーザーが作成したカスタムルールを削除する。
- 各プロファイル（ドメイン／プライベート／パブリック）を有効に戻す。
- 受信を遮断／送信を許可
- ログの保存先、サイズ制限がデフォルトに戻る。
- その他の既定のルールが有効に戻る。

.. code-block:: bat

   netsh advfirewall reset

カスタムルール、既定ルールも含めてすべてのルールを削除する。

.. code-block:: powershell

   Get-NetFirewallRule | Remove-NetFirewallRule

ファイアウォールの設定をインポートする。

.. code-block:: bat

   netsh advfirewall import "C:/temp/firewall-dump.wfw"


ファイアウォールの操作
=======================

ファイアウォールの設定を出力する。

.. code-block:: powershell

   # コマンドとして登録する
   # notepad $PROFILE
   # . $PROFILE

   function Get-MyNetFirewallRule {
     Get-NetFirewallRule | ForEach-Object {
       $port = $_ | Get-NetFirewallPortFilter
       $address = $_ | Get-NetFirewallAddressFilter
       $service = $_ | Get-NetFirewallServiceFilter
       $app = $_ | Get-NetFirewallApplicationFilter

       [PSCustomObject]@{
         Name = $_.DisplayName
         Profile = $_.Profile
         Enabled = $_.Enabled
         Direction = $_.Direction
         Protocol = $_.Protocol
         LocalAddr = $address.LocalAddress
         LocalPort = $port.LocalPort
         RemoteAddr = $address.RemoteAddress
         RemotePort = $port.RemotePort
         Action = $_.Action
         Service = $service.Service
         Program = $app.Program
       }
     }

.. attention::

   ルールの基本情報（名前、方向等）と詳細情報（IPアドレス、ポート番号、プロトコル等）はWindows内部では別テーブルで管理されています。


ルールの色々な出力方法

.. code-block:: bat

   Get-NetFirewallRule | Select-Object DisplayName | Format-List | Out-File "firewall-dump.txt"
   Get-NetFirewallRule | Select-Object DisplayName | Format-Table -AutSize | Out-File -FilePath "firewall-dump.txt" -Width 512
   Get-NetFirewallRule | Select-Object DisplayName | Export-Csv -Path "firewall-dump.csv" -Encoding UTF8 -NoTypeInformation

.. code-block:: bat

   netsh advfirewall firewall show rule name=all verbose > firewall-dumb.txt

各プロファイル（ドメイン／プライベート／パブリック）の状態を確認する。

.. code-block:: powershell

   Get-NetFirewallProfile | Select-Object Name, Enabled

.. code-block:: bat

   netsh advfirewall show allprofiles


各プロファイルでファイアウォールの有効化／無効化

.. code-block:: powershell

   Set-NetFirewallProfile -All -Enabled True
   Set-NetFirewallProfile -All -Enabled False

ルールの検索

.. code-block:: powershell

   Get-NetFirewallRule -DisplayName "*<string>*"

ルールの所属するプロファイルを確認

- Domain: ドメインネットワーク接続時に有効
- Private: プライベートネットワーク接続時に有効
- Public: パブリックネットワーク接続時に有効
- Any: すべてのネットワークで有効

.. code-block:: powershell

   Get-NetFirewallRule -DisplayName "<rule-name>" | Select-Object DisplayName, Profile

ルールの有効化／無効化

.. code-block:: powershell

   Enable-NetFirewallRule -DisplayName "<rule-name>"
   Disable-NetFirewallRule -DisplayName "<rule-name>"

ルールの追加

.. code-block:: powershell

   New-NetFirewallRule -DisplayName "Allow Port 8080" -Direction Inbound -Protocol TCP -LocalPort 8080  -Action Allow
   New-NetFirewallRule -DisplayName "Allow DNS Service" -Direction Outbound -Service dnscache -Protocol UDP -Action Allow
   New-NetFirewallRule -DisplayName "Allow Edge" -Direction Outbound -Protocol TCP -RemotePort 443 -Action Allow -Program "C:/Program Files (x86)/Microsoft/Edge/Application/msedge.exe"
   
.. code-block:: bat

   netsh advfirewall firewall add rule name="Allow RDP" dir=in action=allow protocol=TCP localport=3389

ルールの削除

.. code-block:: powershell

   Remove-NetFirewallRule -DisplayName "Allow Port 8080"



設定例
=======

特定のIPアドレスから受信した通信を遮断する。

.. code-block:: powershell

   New-NetFirewallRule -DisplayName "Block IP" -Direction Inbound -RemoteAddress 192.168.1.100 -Action Block

ワイルドカードを使ってルールを友好化する。

.. code-block:: powershell

   Enable-NetFirewallRule -DisplayName "*Remote*"
   Enable-NetFirewallRule -DisplayGroup "ファイルとプリンターの共有"

プロファイルを指定してルールを有効化する。

.. code-block:: powershell

   Get-NetFirewallRule | Where-Object { $_.Profile -eq "Public" } | Enable-NetFirewallRule

ルールに紐付くプロファイルを変更する。

.. code-block:: powershell

   Set-NetFirewallRule -DisplayName "<rule-name>" -Profile Any
