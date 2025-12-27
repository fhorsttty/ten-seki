========
git push
========

GitHubにリポジトリを作成して、ローカルからコミットをプッシュする方法

SSH環境の構築

.. code-block:: bash

   cd ~
   ssh-keygen

::

   Generating public/private rsa key pair.
   Enter file in which to save the key (/home/<user>/.ssh/id_rsa):
   Enter passphrase (empty for no passphrase):
   Enter same passphrase again:
   Your identification has been saved in /home/<user>/.ssh/id_rsa
   Your public key has been saved in /home/<user>/.ssh/id_rsa.pub
   The key fingerprint is:
   SHA256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx <user>@localhost
   The key's randomart image is:

.. code-block:: bash

   cd ~
   ssh-keygen
   ls -la .ssh

::

   id_rsa id_rsa.pub

| GitHub上に公開鍵（id_rsa.pub）を登録する。
| [profile-icon] => [settings] => [SSH and GPG keys]
| SSHのコンフィグにGitHubの接続先を設定する。

.. code-block:: none
   :caption: /home/<user>/.ssh/config

   Host github
     User git
     Hostname ssh.github.com
     IdentityFile ~/.ssh/id_rsa
     IdentitiesOnly yes
     Port 443

GitHubに接続の試験をする。

.. code-block:: bash

   ssh -T github

::

   Hi <user>! You've successfully authenticated, but GitHub does not provide shell access.

gitの初期設定をする。

.. code-block:: bash

   git config --global --list
   git config --global user.name <user>
   git config --global user.email <email-address>
   git config --global --unset user.name  # 削除したい時

   git config --global init.defaultBranch main
   git config --global --list

GitHub上でリポジトリを作成して、PCローカルにpullする。

.. code-block:: bash

   git clone https://github.com/<user>/<repository>.git
   cd <repository>
   git init
   git remote -v
   git remote add origin github:<user>/<repository>.git
   git remote -v

テストファイルを作成して、GitHubにpushする。

.. code-block:: bash

   echo "Hello World!" > hello
   git add .
   git status
   git commit -m 'test commt'
   git push origin master
