
セットアップできたら、早速踏み込んで使ってみましょう。この節では <a href="http://www.alpinelinux.org/" target="_blank">Alpine Linux</a>（軽量 Linux ディストリビューション）コンテナを走らせ、`docker run` コマンドの味見をしてみます。

ではまず、ターミナルで次のコマンドを実行しましょう。

```
$ docker pull alpine
```

> 補足: Docker をどのようにインストールしたかによりますが、もしかすると `permission denied` と表示されるかもしれません。その場合、Linux であれば `docker` コマンドの前に `sudo` をつけるか、<a href="http://docs.docker.jp/engine/installation/ubuntulinux.html#docker" target="_blank">docker グループの作成</a> によって問題を解決することができます。

`pull` コマンドは **Docker レジストリ** から alpine **イメージ** をダウンロードし、そのデータをあなたの端末に保存します。`docker images` コマンドを使えば、保存されたイメージの一覧を確認することができます。

```
$ docker images
REPOSITORY              TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
alpine                  latest              c51f86c28340        4 weeks ago         1.109 MB
hello-world             latest              690ed74de00f        5 months ago        960 B
```

### 2.1 Docker Run

では続けて、このイメージを使って Docker **コンテナ** を起動してみましょう。`docker run` コマンドを使います。

```
$ docker run alpine ls -l
total 48
drwxr-xr-x    2 root     root          4096 Mar  2 16:20 bin
drwxr-xr-x    5 root     root           360 Mar 18 09:47 dev
drwxr-xr-x   13 root     root          4096 Mar 18 09:47 etc
drwxr-xr-x    2 root     root          4096 Mar  2 16:20 home
drwxr-xr-x    5 root     root          4096 Mar  2 16:20 lib
......
```

何が起きたのでしょうか？このコマンドの裏では、とてもたくさんのことが起こっています。`run` を実行すると Docker クライアントはイメージを探し（このケースでは alpine）、コンテナを作成し、そのコンテナの *中で* 指定したコマンド（ls -l）を実行します。結果として、コンテナ内部のディレクトリが一覧表示されています。

もっと楽しいことをやってみましょう。

```
$ docker run alpine echo "hello from alpine"
hello from alpine
```

何か表示されました！このケースでは Docker クライアントは alpine コンテナ内部で忠実に `echo` を実行した後、そこから exit したようです。お気づきのように、すべてがとても高速です。仮想マシンを起動し、コマンドを実行し、それを終了することを想像してみてください。誰もがコンテナは速い！という意味をお分かりいただけたと思います。

もうひとつ別のコマンドを試しましょう。

```
$ docker run alpine uptime
00:16:48 up  1:48,  0 users,  load average: 0.00, 0.01, 0.04
```

いいですね。ではもうひとつ。

```
$ docker run alpine /bin/sh
```

おっと、何も起きない！バグでしょうか・・？いえ、違います。この手の対話型シェルは、対話のできる *ターミナル内で実行しない限り* 、記述されたコマンドを実行するとすぐに exit してしまうのです。そこで改めて、exit しないよう `-it` オプションをつけて `docker run -it alpine /bin/sh` を実行します。

これでコンテナ内部のシェルと対話できる状態になりました。`ls -l` や `uptime` などいくつかコマンドを試してみましょう。`exit` でコンテナから抜けることができます。

次に、`docker ps` コマンドをみてみましょう。`docker ps` コマンドは現在起動しているコンテナの一覧を返してくれます。

```
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

実行中のコンテナがないので 1 行も表示されません。ではここで、便利なオプションを指定してみましょう: `docker ps -a`

```
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS               NAMES
36171a5da744        alpine              "/bin/sh"                5 minutes ago       Exited (0) 2 minutes ago                        fervent_newton
305297d7a235        alpine             "uptime"                  5 minutes ago       Exited (0) 4 minutes ago                        distracted_goldstine
a6a9d46d0b2f        alpine             "echo 'hello from alp"    6 minutes ago       Exited (0) 6 minutes ago                        lonely_kilby
ff0a5c3750b9        alpine             "ls -l"                   8 minutes ago       Exited (0) 8 minutes ago                        elated_ramanujan
c317d0a9e3d2        hello-world         "/hello"                 34 seconds ago      Exited (0) 12 minutes ago                       stupefied_mcclintock
```

表示されたのはこれまで実行したすべてのコンテナです。`STATUS` 列を見ると、コンテナたちが数分前に exit したことがわかります。ところで、コンテナの中で複数のコマンドを実行する方法があるのか気になった方のために、次はそれを試してみましょう。

```
$ docker run -it alpine /bin/sh
/ # ls
bin      dev      etc      home     lib      linuxrc  media    mnt      proc     root     run      sbin     sys      tmp      usr      var
/ # uptime
 05:45:21 up  5:58,  0 users,  load average: 0.00, 0.01, 0.04
```

`-it` オプションをつけて `run` すると、対話モードでコンテナ内の tty に接続できます。こうなれば、あとは好きなだけコンテナ内でコマンドを実行できます。いくつか好きなコマンドを実行してみてください。

> **Danger Zone**: If you're feeling particularly adventurous you can try `rm -rf /bin` in the container. Make sure you run this command in the container and **not** in your laptop. Doing this will not make any other commands like `ls` and `grep` work. Once everything stops working, you can exit the container and then start it up again with the `docker run -it alpine sh` command. Since Docker creates a new container every time, everything should start working again.

That concludes a whirlwind tour of the `docker run` command which would most likely be the command you'll use most often. It makes sense to spend some time getting comfortable with it. To find out more about `run`, use `docker run --help` to see a list of all flags it supports. As you proceed further, we'll see a few more variants of `docker run`.

### 2.2 用語集

In the last section, you saw a lot of Docker-specific jargon which might be confusing to some. So before you go further, let's clarify some terminology that is used frequently in the Docker ecosystem.

- *Images* - The Filesystem and configuration of our application which are used to create containers. To find out more about a Docker image, run `docker inspect alpine`. In the demo above, you used the `docker pull` command to download the **alpine** image. When you executed the command `docker run hello-world`, it also did a `docker pull` behind the scenes to download the **hello-world** image.
- *Containers* - Created using Docker images and run the actual application. You created a container using `docker run` which you did using the alpine image that you downloaded. A list of running containers can be seen using the `docker ps` command.
- *Docker daemon* - The background service running on the host that manages building, running and distributing Docker containers.
- *Docker client* - The command line tool that allows the user to interact with the Docker daemon.
- *Docker Hub* - A [registry](https://hub.docker.com/explore/) of Docker images. You can think of the registry as a directory of all available Docker images. You'll be using this later in this tutorial.

## 次のステップ

次のチュートリアル [3. Docker を使った Web アプリケーション](/workshop/3) に進みましょう。
