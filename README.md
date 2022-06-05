# vscode-notebook-devcontainer
Simple vscode project for development python and jupyter notebook in docker.
This repositry uses [Jupyter Docker Stacks](https://jupyter-docker-stacks.readthedocs.io/en/latest/).

# Usage

## Install Remote Container

Install Extensino for Visual Studio Code. 
[Remote - Containers - Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)


## clone repository

clone this repository

## Open in Container

1. Press <kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>p</kbd> to open Command Palette in Visual Studo Code.
2. Exec `Remote-Containers: rebuild and Reopen in Container`
3. wait until docker-compose build and VSCode open work space

# メモ
## bashを起動したい場合
新しいモジュールをインストールしたり、ファイル操作や`apt-get`など`bash`を操作したい場合があると思う。その場合は`docker exec -it コンテナID /bin/bash`などすれば、`bash`を起動できる。または`VS code`で`Remote - container`している場合は新しくターミナルを開けばいい。

## デフォルトコマンドについて

`jupyter/minimal-notebook`や`jupyter/scipy-notebook`など [Jupyter Docker Stacks](https://jupyter-docker-stacks.readthedocs.io/en/latest/)のシリーズのDockerは立ち上がると同時にデフォルトで`start-notebook.sh`が走り、各種設定を行う。(GRAT_SUDOの有効化など)。`start-notebook.sh`が必要な設定を行うのでデフォルトコマンドは変更しないほうがいい。

## Jupyter notebookを使うための設定

[Jupyter Docker Stacks](https://jupyter-docker-stacks.readthedocs.io/en/latest/index.html)は2022年1月ごろ、デフォルトのサーバーバックエンドをjupyter notebookからjupyter serverに切り替えました。これに伴いデフォルトのエディタがjupyter notebookからjupyter Labに切り替わりました。

バックエンドがjupyter serverのままでも、jupyter notebookは引き続き使えます。`loacalhost:8888`にアクセスすると、`jupyter lab`に、`loacalhost:8888/tree`にアクセスすると、従来の`jupyter notebook`にアクセスします。


しかし、様々な拡張機能があるjupyter_contrib_nbextensionsはjupyter notebookでしか使えず、しかもバックエンドがjupyter serverだと`/tree`のnotebookからもnbextensionは使えなくなってしまいます。私は選択中の文字を強調する`Highlit Selected Word`を使いたいため、jupyter notebook+jupyter_contrib_nbextensionsを使い続けています。いまのところ同じ機能はjupyter labは対応していないようです。

なのでバックエンドをjupyter notebookに切り替える必要があります。


[Common Features — Docker Stacks documentation](https://jupyter-docker-stacks.readthedocs.io/en/latest/using/common.html#switching-back-to-the-classic-notebook-or-using-a-different-startup-command)によるとコンテナ内の環境変数`DOCKER_STACKS_JUPYTER_CMD`を`notebook`に設定することで、立ち上がったサーバのバックエンドjupyter notebookに切り替えることができます。なので、`docker-compose.yml`中で次のように設定しています。

```docker-compose.yml
environment:
    - DOCKER_STACKS_JUPYTER_CMD=notebook
```

## ユーザーと権限について

ユーザーは管理者である`root`と、`jovyan`がいる。`jupyter notebook`を管理者権限で実行するのはセキュリティ的にまずいので、`jupyter notebook`は`jovyan`で起動するようになっている。

普通に立ち上げるとユーザーは`jovyan`になる。権限がないので`apt-get update`や`apt-get install`はできない。

しかし`Dockerfile`中で

```Dockerfile
USER root
```

と`root`に切り替えた場合、`bash`は`root`で起動し、`apt-get`などが使えるようになる。

この状態で、`su jovyan`などとすれば、`jovyan`に切り替わる。この`jovyan`では`/opt/conda/bin/`のパスが通っていないため、`pip`コマンドや`conda`コマンドが使えない。`export PATH=$PATH:/opt/conda/bin`などとして、環境変数の`PATH`に`/opt/conda/bin/`を加えれば、`pip`コマンドや`conda`コマンドが使えるようになる。

またDockerfile中で`root`に切り替え、`bash`は`root`で起動するようにした上で、`docker-compose.yml`で、

```docker-compose.yml
- GRANT_SUDO=yes
```

としていれば、`jovyan`も`sudo`が使えるようになる。

つまり`GRANT_SUDO=yes` かつ`USER root`なら`jovyan`は`sudo`を使えるということ

https://jupyter-docker-stacks.readthedocs.io/en/latest/using/recipes.html

参照

## モジュールのインストールとユーザー権限について

`conda install`や`pip install`は`root`と`jovyan`どちらのユーザーで実行しても、`Python`のプログラミングには問題ないように思える。

しかし唯一`jupyter_contrib_nbextensions`のインストールは`jovyan`で行ったほうがいい。

`root`で`conda install -y jupyter_contrib_nbextensions`としてしまうと、`nbextensions`関連のファイルが`root`権限になってしまい、`jupyter`起動ユーザーが`jovyan`なため、`jupyter`上に`nbextensions`のタブが現れても、どのエクステンションを有効にして書き換えても実際は権限の問題で書き換わらずに、F5で更新すると無効になってしまう。

だから`jupyter_contrib_nbextensions`のインストールは`Dockerfile`内`jovyan`ユーザーで予めやっておく。

```Dockerfile
RUN conda install -y jupyter_contrib_nbextensions
```

詳しくは
[jupyter notebookが使えるdockerにnbextensionsが導入できない](https://teratail.com/questions/193568)

参照
