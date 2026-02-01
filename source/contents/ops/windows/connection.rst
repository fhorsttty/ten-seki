===========
connection
===========

AllJoynルーター
=================

Quarcommが開発し、Allseen Allianceが推進していたIoTデバイス間、IoTデバイスとアプリ間の通信仕様を定めたオープンソース・フレームワーク

Windows10から標準搭載されたが、現在は **Matter** が業界標準の仕様となったため、Windows11 24H2やWindows Server 2025では **非推奨** となっている。


通信遮断による影響
-------------------

メリット
^^^^^^^^^

- **セキュリティ向上** : 外部からのスキャンや潜在的な脆弱性を突いた攻撃からのリスクを軽減できる。

- **トラフィックの低減** : デイバスを探索するためのブロードキャスト、マルチキャストがなくなるため、ネットワーク帯域の負荷を低減できる。

デメリット
^^^^^^^^^^^

- Windowsをスマートホームのハブとして利用したり、AllJoyn対応のスマート家電を遠隔操作できなくなる。


通信の停止方法
---------------

.. code-block:: powershell

   # 通信遮断
   Disable-NetFirewallRule -DisplayName "AllJoyn ルーター*"

   # サービス停止
   Stop-Service -Name "AJRouter" -Force
   Set-Service -Name "AJRouter" -StartupType Disabled


Connected User Experiences and Telemetry
==========================================

Windowsの安定性向上を目的に、Microsoftへ診断データを送信するための通信

- OSの動作状況、アプリケーションのクラッシュログ、ハードウェア構成情報などを収集し、Microsoftのサーバーへ送信する。
- Windowsの新機能の提案、使用状況に合わせた情報の提供を行う。

通信遮断による影響
--------------------

メリット
^^^^^^^^^

- **機密性の保持** : 組織内のシステム構成やエラー内容が外部に送信されるのを防げる。

デメリット
^^^^^^^^^^^

- 特定の環境でのみ発生するバグの存在が、Microsoft側で認識されない
- 診断データがないため、システムに最適なドライバーや更新プログラムの判別が正しく行われないことがらう。
 

通信の停止方法
---------------

.. code-block:: powershell

   # 通信遮断
   New-NetFirewallRule -DisplayName "Block Telemetry (DiagTrack) Outbound" `
     -Direction Outbound `
     -Service "DiagTrack" `
     -Action Block

   # サービス停止
   Stop-Service -Name "DiagTrack" -Force
   Set-Service -Name "DiagTrack" -StartupType Disabled

グループポリシーによる制限

1. ``gpedit.msc`` 
2. [コンピュータの構成] > [管理用テンプレート] > [Windowsコンポーネント] > [データ収集とプレビューのビルド]
3. [診断データを許可する] を [無効] に設定する。



mDNS(UDP送信)
===============

マルチキャストアドレス（ ``224.0.0.251`` ）と ``5353/UDP`` ポートを使用して通信する。

- ネットワーク上のプリンター、ファイル共有、IoTデバイスなどを、DNSサーバーを経由せずに ``<hostname>.local`` という形式で名前解決を行う。
- Bnjourサービス、Webブラウザ、特定のアプリケーションが、ネットワーク近傍のサービスを検索するときに利用する。


通信遮断による影響
--------------------

メリット
^^^^^^^^^

- **セキュリティ強化** : ネットワークに侵入した攻撃者が、mDNSからサーバーの存在や稼働しているサービス情報を収集するのを防止できる。
- **DoS攻撃のリスク低減** : mDNSの脆弱性を突いたDoS攻撃のターゲットになるのを防ぐことができる。
- **トラフィックの低減** : 名前解決によるブロードキャスト、マルチキャストがなくなるため、ネットワーク帯域の負荷を低減できる。

デメリット
^^^^^^^^^^^

- サーバーに対して、DNSを経由せずに ``<hostname>.local`` という名前の形式でアクセスできなくなる。
- ネットワークプリンターの自動セットアップなどが失敗することがある。


通信の停止方法
---------------

.. code-block:: powershell

   # 受信の遮断
   Disable-NetFirewallRule -DisplayName "mDNS (UDP-In)"

   # 送信の遮断
   New-NetFirewallRule -DisplayName "Block mDNS Outbound" `
     -Direction Outbound `
     -Portocol UDP `
     -LocalPort 5353 `
     -Action Block

   # サービスの停止
   Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\DNSClient"


グループポリシーによる制限

1. ``gpedit.msc`` 
2. [コンピュータの構成] > [管理用テンプレート] > [ネットワーク] > [DNSクライアント]
3. [マルチキャスト名前解決をオフにする] を [有効] に設定する。

.. attention::

   LLMNRも制限されます。


Windows Feature Experience Pack
==================================

OSの大型アップデートより先に、UIや一部の機能を個別に更新するための仕組み。

- スクリーンキャプチャ、テキスト入力、シェルなどのUIに関連する更新をMicrosoft StoreやWindows Updateを経由して提供する。



通信遮断による影響
--------------------

メリット
^^^^^^^^^

- **UIの固定** : 管理ツールなどのUIが意図せず変更されるのを防止する。
- **不要な通信の停止** : Microsoft StoreやWindows Updateの更新確認に関連する外部通信を減らすことができる。

デメリット
^^^^^^^^^^^

- Snipping ToolやIMEなどの不具合修正が適用されなくなる。


通信の停止方法
---------------

.. code-block:: powershell

   New-NetFirewallRule -DisplayName "Block Feature Experience Pack Update" `
     -Direction Outbound `
     -Program "%SystemRoot%\System32\svchost.exe" `
     -Service "wuauserv"
     -Action Block

.. caution::

   Windows Updateが遮断されるため、セキュリティパッチの適用も遮断されるので、グループポリシーによる制限が推奨される。


グループポリシーによる制限

1. ``gpedit.msc`` 
2. [コンピュータの構成] > [管理用テンプレート] > [Windowsコンポーネント] > [Windows Update] > [Windows Updateから提供される更新プログラムの管理]
3. [プレビュービルドや機能更新プログラムを一時停止する] を [有効] に設定する。
4. [オプションの更新プログラムを無効にする] を [有効] に設定する。


Windows Search
================

ファイル、メール、アプリケーションなどのコンテンツを高速に検索するためのインデックスを作成するためのサービス

- ファイルのプロパティや内容をインデックス化し、検索ボックスからの回答までの処理を高速化する。
- **Bing検索の統合** が有効である場合、検索キーワードがMicrosoftのサーバーへ送信される。
- ファイルサーバ機能を有効にしている場合、共有フォルダ内にあるファイル検索の高速化にも利用される。


通信遮断による影響
--------------------

メリット
^^^^^^^^^

- **システム負荷の軽減** : インデックス作成にかかるディスクI/OやCPUのリソース消費を減らせる。
- **機密性の保持** : 検索ボックスに入力した内容が、Bingへ送信されるのを防止できる。
- **ディスク寿命の延命** : インデックス更新による書き込みを削減できる。

デメリット
^^^^^^^^^^^

- ファイル検索時に毎回全セクタを走査するため、検索に数分かかる。
- Outlook上でメールの検索ができなくなる。


通信の停止方法
---------------

.. code-block:: powershell

   # 通信遮断
   New-NetFirewallRule -DisplayName "Block Windows Search Web Outbound" `
     -Direction Outbound `
     -Program "%SystemRoot%\System32\SearchFilterHost.exe" `
     -Action Block

   # サービス停止
   Stop-Service -Name "WSearch" -Force
   Set-Service -Name "WSearch" -StartupType Disabled


グループポリシーによる制限

1. ``gpedit.msc`` 
2. [コンピュータの構成] > [管理用テンプレート] > [Windowsコンポーネント] > [検索]
3. [Web検索を許可しない] を [有効] に設定する。
4. [検索でWebを検索したりWebの結果を表示したりしない] を [有効] に設定する。

