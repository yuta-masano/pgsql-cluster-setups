## playbook

PostgreSQL 3台、Pgpool-II 2台のクラスタを構成します。
PostgreSQLは pg_wal を tmpfs上に、fsync=off とした、apply同期設定の
セミオンメモリのストリーミングレプリケーション構成です。

2台の Pgpool-II は watchdogクラスタを構成しています。
設定値の例は AWS上のプライベートIPを代表IPとする構成です。
Pgpool-II配置インスタンスに cli実行可能なIAMロールが予め
割り当ててある想定です。
フェイルオーバ、pg_rewindによるフォローマスタ、オンラインリカバリが
設定済みです。


    $ ansible-playbook -i hosts setup-hosts.yml

/etc/hosts と /etc/hostname を頒布します。
ネットワークが設定済みなら不要です。


    $ ansible-playbook -i hosts setup-keypair.yml

インタラクティブなパスワード入力無しのsshログインを設定します。
以下のユーザ・ホスト間でそのように設定します。同一キーペアを使います。
PostgreSQLサーバのSELinux解除も行います。

                root@[g-pool] --ssh--> postgres@[g-pg-p,g-pg-s]
    postgres@[g-pg-p,g-pg-s] <--ssh--> postgres@[g-pg-p,g-pg-s]


    $ ansible-playbook -i hosts drop-all-pg.yml

プライマリ・スタンバイのPostgreSQLを停止してデータを削除します。


    $ ansible-playbook -i hosts stop-pgpool.yml

Pgpool-IIを停止します。


    $ ansible-playbook -i hosts setup-pg-prmary.yml

プライマリPostgreSQLをセットアップします。


    $ ansible-playbook -i hosts setup-pg-standby.yml

スタンバイPostgreSQLをセットアップします。


    $ ansible-playbook -i hosts setup-pgpool.yml

2台組 Pgpool-II をセットアップします。
files/ifupdown.sh を修正すれば AWS用を別の環境用に変更できます。


## vars

    pgdg_url: "https://download.postgresql.org/pub/repos/yum/11/redhat/rhel-7-x86_64/pgdg-centos11-11-2.noarch.rpm"
    pgver: 11

使用する PostgreSQL の PGDGリポジトリを指定します。


    subnet: "172.31.16.0/20"
    pg1_host: pg1
    pg1_addr: 172.31.17.213
    pg2_host: pg2
    pg2_addr: 172.31.25.63
    pg3_host: pg3
    pg3_addr: 172.31.31.31
    pool1_host: pg4
    pool1_addr: 172.31.28.143
    pool2_host: hpg5
    pool2_addr: 172.31.25.240

テスト用のホストとサブネットを設定します。


    trusted_servers: 172.31.30.9
    delegate_ip: 172.31.25.241
    wd_authkey: hello
    if_cmd_param1: "172.31.25.241 20 eni-045049b95775cd5dc eth0 us-west-2"
    if_cmd_param2: "172.31.25.241 20 eni-05271115f9a4aff67 eth0 us-west-2"
    pcp_pass: pass

Pgpool-II用の設定を与えます。
if_cmd_param1, 2 は AWS用で、files/ifupdown.sh に与えるパラメータです。

