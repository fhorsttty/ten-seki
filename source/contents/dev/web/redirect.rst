=========
Redirect
=========

サーバーサイド
===============

Nginx
------

URLのホスト名にwwwを付加してリダイレクト

.. code-block:: nginx

   server {
     listen 80;
     server_name example.co.jp

     rewrite ^(.*)$ https://www.example.com$1 permantent;
   }

302

.. code-block:: nginx

   location ^~ /foo/bar {
     rewrite ^(.*)$ https://example.co.jp/hoge redirect;
   }

301

.. code-block:: nginx

   location ^~ /foo/bar {
     rewrite ^(.*)$ https://example.co.jp/hoge permanent;
   }

VirtualHostでのリダイレクト振り分け

.. code-block:: nginx


   localtion ^~ /foo/ {
   
     #example.com/foo/bar/ => example.co.jp/foo/bar
     if ($request_uri ~ "(.+)/$") {
       rewrite ^(.+)/$  https://example.co.jp$1 permanent;
     }
     
     #example.com/foo/bar => example.co.jp/foo/bar
     if ($request_uri ~ "(.*)$") {
       rewrite ^(.*)/$  https://example.co.jp$1 permanent;
     }
   }


クライアントサイド
===================

HTML
-----

content内の数字は、リダイレクトを開始するまでの待ち時間（秒）

.. code-block:: html

   <head>
     <meta http-equiv="refresh" content="1; URL=https://example.co.jp/">
   </head>

JS
---

.. code-block:: html

   <script>
     window.location.href('https://example.co.jp')
   </script>

   <noscript>
     <a href="https://example.co..jp"> example site </a>
   </noscript>
