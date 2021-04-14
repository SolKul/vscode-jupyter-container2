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

1. open Command Palette in Visual Studo Code.
2. Exec `Remote-Containers: Reopen Folder in Container`
3. wait until docker-compose build

# メモ
## bashを起動したい場合
新しいモジュールをインストールしたり、ファイル操作や`apt-get`など`bash`を操作したい場合があると思う。その場合は`docker exec -it コンテナID /bin/bash`などすれば、`bash`を起動できる。または`VS code`で`Remote - container`している場合は新しくターミナルを開けばいい。

## デフォルトコマンドについて

`jupyter/minimal-notebook`や`jupyter/scipy-notebook`など [Jupyter Docker Stacks](https://jupyter-docker-stacks.readthedocs.io/en/latest/)のシリーズのDockerは立ち上がると同時にデフォルトで`start-notebook.sh`が走り、各種設定を行う。(GRAT_SUDOの有効化など)。`start-notebook.sh`が必要な設定を行うのでデフォルトコマンドは変更しないほうがいい。

## Jupyter Labについて

徐々に推奨エディタが`Lab`に移り変わっていると思われる。

`loacalhost:8888/tree`にアクセスすると、従来の`jupyter notebook`に、  
`loacalhost:8888/lab`にアクセスすると、`jupyter lab`にアクセスする

```docker-compose.yml
environment:
    - JUPYTER_ENABLE_LAB=yes
```

とすると、`localhost:8888`にアクセスした時点で`jupyter lab`にアクセスするするようになる。

## nbextensionsについて

`jupyter_contrib_nbextensions`は`jupyter notebook`でしか使えない。私は選択中の文字を強調する`Highlit Selected Word`を使いたいため、`jupyter notebook`+`jupyter_contrib_nbextensions`を使い続けている。いまのところ同じ機能は`jupyter lab`は対応していない。

しかも、

```docker-compose.yml
environment:
    - JUPYTER_ENABLE_LAB=yes
```

としてしまうと、`jupyter notebook`が`jupyter lab`管理下のものになってしまい、`jupyter_contrib_nbextensions`は使えなくなる。`jupyter_contrib_nbextensions`を使いたい場合は上はしないようにする。

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
