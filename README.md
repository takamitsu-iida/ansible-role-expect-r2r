expect-r2r
=========

expectスクリプトを使ってネットワーク機器から情報を収集するansibleロールです。

ネットワークの構成によっては装置に直接接続することはできず、ホップバイホップで接続することを余儀なくされることもあります。
そして往々にしてそのような環境に置かれている装置はtelnetでしか接続できなかったりしますね。

そのような直接乗り込めない環境の装置からansible単独で情報を収集するのは困難です。

装置への接続とコマンドの打ち込みはexpectスクリプトで行い、スクリプトの起動と結果の処理をansibleで行ったほうが生産性は良さそうです。

Requirements
------------

ローカルホストにexpectがインストールされている必要があります。

expectスクリプトが正常に動作することを確認してください。

expectスクリプトには実行権限が必要です。

```bash
r2r.expect \
  踏み台のIP \
  踏み台の接続ユーザ名 \
  踏み台のパスワード \
  踏み台の管理者パスワード\
  ターゲットのIP \
  ターゲットの接続ユーザ名 \
  ターゲットのパスワード \
  '実行したいコマンド'
```

expectスクリプトの実行例です。。

10.35.185.2のCiscoルータに一度ログインして、そのあと172.20.0.21のCiscoルータに乗り込んで、show versionを実行します。

接続関連の処理は画面に表示されません。showコマンドの結果だけが表示されます。

```bash
iida$ ./files/r2r.expect 10.35.185.2 cisco cisco123 cisco123 172.20.0.21 cisco cisco123 cisco123 'show version'

r1#show version
Cisco IOS XE Software, Version 16.03.05
Cisco IOS Software [Denali], CSR1000V Software (X86_64_LINUX_IOSD-UNIVERSALK9-M), Version 16.3.5, RELEASE SOFTWARE (fc1)
Technical Support: http://www.cisco.com/techsupport
Copyright (c) 1986-2017 by Cisco Systems, Inc.
Compiled Thu 05-Oct-17 02:38 by mcpre


Cisco IOS-XE software, Copyright (c) 2005-2017 by cisco Systems, Inc.
All rights reserved.  Certain components of Cisco IOS-XE software are
licensed under the GNU General Public License ("GPL") Version 2.0.  The
software code licensed under GPL Version 2.0 is free software that comes
with ABSOLUTELY NO WARRANTY.  You can redistribute and/or modify such
GPL code under the terms of GPL Version 2.0.  For more details, see the
documentation or "License Notice" file accompanying the IOS-XE software,
or the applicable URL provided on the flyer accompanying the IOS-XE
software.


ROM: IOS-XE ROMMON

r1 uptime is 8 weeks, 5 days, 21 hours, 40 minutes
Uptime for this control processor is 8 weeks, 5 days, 21 hours, 41 minutes
System returned to ROM by reload
System image file is "bootflash:packages.conf"
Last reload reason: reload



This product contains cryptographic features and is subject to United
States and local country laws governing import, export, transfer and
use. Delivery of Cisco cryptographic products does not imply
third-party authority to import, export, distribute or use encryption.
Importers, exporters, distributors and users are responsible for
compliance with U.S. and local country laws. By using this product you
agree to comply with applicable laws and regulations. If you are unable
to comply with U.S. and local laws, return this product immediately.

A summary of U.S. laws governing Cisco cryptographic products may be found at:
http://www.cisco.com/wwl/export/crypto/tool/stqrg.html

If you require further assistance please contact us by sending email to
export@cisco.com.

License Level: ax
License Type: Default. No valid license found.
Next reload license Level: ax

cisco CSR1000V (VXE) processor (revision VXE) with 2043432K/3075K bytes of memory.
Processor board ID 9T8873IY5FA
4 Gigabit Ethernet interfaces
32768K bytes of non-volatile configuration memory.
3985216K bytes of physical memory.
7774207K bytes of virtual hard disk at bootflash:.
0K bytes of  at webui:.

Configuration register is 0x2102

r1#
r1#
r1#
iida$
```

Role Variables
--------------

ロール変数はdefaults/main.ymlに記載しています。

- EXPECT_SCRIPT_PATH: expectスクリプトのパスです。デフォルトは'/files/r2r.expect'です。
- NTC_PATH: Ciscoデバイスのshow interacesコマンド出力を加工するためのNTCテンプレートです。デフォルトは'files/cisco_ios_show_interfaces.template'です。

Dependencies
------------

他のロールへの依存関係はありません。

Example Playbook
----------------

このロールでは、Ciscoルータを踏み台にして別のCiscoルータに接続し、show interfacesを実行します。
その出力をNTCテンプレートで処理して、結果を表示します。

tests/test.ymlがプレイブックです。vars.ymlから変数を読み込むようにしています。

```yml
- name: GATHER SHOW COMMANDS OUTPUT
  hosts: r1, r2
  vars_files: vars.yml
  gather_facts: false
  serial: 1

  tasks:
    - import_role:
        name: ansible-role-expect-r2r
```

tests/vars.ymlには、ターゲット利用する踏み台に関する情報を記述します。
ターゲット装置ごとに踏み台を変えないといけない場合は、インベントリのhost_varsに定義した方が良いでしょう。

```yml
# expectで踏み台にする装置の情報
JUMP_HOST: 10.35.185.2
JUMP_USERNAME: cisco
JUMP_PASSWORD: cisco123
JUMP_ENABLE: cisco123
```

プレイブックの実行例。

```bash
iida$ ansible-playbook tests/test.yml

PLAY [GATHER SHOW COMMANDS OUTPUT] ***************************************************************************

TASK [FAIL SCRIPT IF REQUIRED TARGET_HOSTS PARAMETER IS MISSING] *********************************************
skipping: [r1]

TASK [ansible-role-expect-r2r : create log directory if not exists] ******************************************
ok: [r1 -> localhost]

TASK [ansible-role-expect-r2r : get current time] ************************************************************
ok: [r1]

TASK [ansible-role-expect-r2r : run expect script on localhost] **********************************************
changed: [r1 -> localhost]

TASK [ansible-role-expect-r2r : save result to file] *********************************************************
changed: [r1 -> localhost]

TASK [ansible-role-expect-r2r : parse by ntc-templates] ******************************************************
ok: [r1]

TASK [ansible-role-expect-r2r : display interface speed and mtu] *********************************************
ok: [r1] => {}

MSG:

interface = GigabitEthernet1
  speed = 1000Mbps
  mtu = 1500

interface = GigabitEthernet2
  speed = 1000Mbps
  mtu = 1500

interface = GigabitEthernet3
  speed = 1000Mbps
  mtu = 1512

interface = GigabitEthernet4
  speed = 1000Mbps
  mtu = 1500

interface = Loopback0
  speed =
  mtu = 1514


PLAY [GATHER SHOW COMMANDS OUTPUT] ***************************************************************************

TASK [FAIL SCRIPT IF REQUIRED TARGET_HOSTS PARAMETER IS MISSING] *********************************************
skipping: [r2]

TASK [ansible-role-expect-r2r : create log directory if not exists] ******************************************
ok: [r2 -> localhost]

TASK [ansible-role-expect-r2r : get current time] ************************************************************
ok: [r2]

TASK [ansible-role-expect-r2r : run expect script on localhost] **********************************************
changed: [r2 -> localhost]

TASK [ansible-role-expect-r2r : save result to file] *********************************************************
changed: [r2 -> localhost]

TASK [ansible-role-expect-r2r : parse by ntc-templates] ******************************************************
ok: [r2]

TASK [ansible-role-expect-r2r : display interface speed and mtu] *********************************************
ok: [r2] => {}

MSG:

interface = GigabitEthernet1
  speed = 1000Mbps
  mtu = 1500

interface = GigabitEthernet2
  speed = 1000Mbps
  mtu = 1500

interface = GigabitEthernet3
  speed = 1000Mbps
  mtu = 1500

interface = GigabitEthernet4
  speed = 1000Mbps
  mtu = 1500

interface = Loopback0
  speed =
  mtu = 1514


PLAY RECAP ***************************************************************************************************
r1                         : ok=6    changed=2    unreachable=0    failed=0
r2                         : ok=6    changed=2    unreachable=0    failed=0

iida$
```

License
-------

BSD

Author Information
------------------

@takamitsu-iida

メモ
----

expectスクリプトの全文は以下のようになっています。

Ciscoルータを前提に書いたものですので、enableやterminal length 0を打ち込んでいます。
Cisco以外の装置の場合は適宜書き換えてご利用ください。

```tcl
#!/usr/bin/expect -f
#!/usr/local/bin/expect -f

#
# ルータを踏み台にして、別のルータに乗り込むためのexpectスクリプトです。
# 2019/08/20
# takamitsu-iida
#

# 実行例
# $ ./r2r.expect 10.35.185.2 cisco cisco123 cisco123 172.20.0.21 cisco cisco123 cisco123 "show ip route" "show process cpu | i CPU"

if {$argc < 8} {
  puts "Usage: $argv0 jump_host jump_user jump_pass jump_enable target_host target_user target_pass target_enable command command ..."
  exit
}

# 引数から情報を取得
#   第1引数： 踏み台のホスト名(IPアドレス)
#   第2引数： 踏み台のユーザ名
#   第3引数： 踏み台のパスワード
#   第4引数： 踏み台の管理者パスワード
#   第5引数： ターゲット装置のホスト名(IPアドレス)
#   第6引数： ターゲット装置のユーザ名
#   第7引数： ターゲット装置のパスワード
#   第8引数： ターゲット装置の管理者パスワード
#   第9引数以降はターゲット装置で実行したいコマンドです。

set JUMP_HOST [lindex $argv 0]
set JUMP_USERNAME [lindex $argv 1]
set JUMP_PASSWORD [lindex $argv 2]
set JUMP_ENABLE [lindex $argv 3]

set REMOTE_HOST [lindex $argv 4]
set REMOTE_USERNAME [lindex $argv 5]
set REMOTE_PASSWORD [lindex $argv 6]
set REMOTE_ENABLE [lindex $argv 7]

# 20秒間応答がなければ終了
set timeout 20

# 接続時に期待するプロンプト(正規表現)
# motdを表示する装置の場合、いきなりこれにヒットしてしまうかもしれない
# その場合はもっと厳密にプロンプトを定義しないといけない
set INITIAL_PROMPT "\[#$>\]"

# 画面表示を停止(0で停止、1で再開)
log_user 0

# telnet または ssh を起動
spawn telnet ${JUMP_HOST}

# Username: または Login: が返ってきたら
expect -re "\[Uu\]sername:|\[Ll\]ogin:" {

  # ユーザ名を送信して
  send -- "${JUMP_USERNAME}\r"

  # Password: が戻ってきたら
  expect -re "\[Pp\]assword:" {

    # パスワードを送信
    send -- "${JUMP_PASSWORD}\r"
    expect -re ${INITIAL_PROMPT}

  # プロンプトが返ってきたら、パスワードなしでログインできたということ
  } ${INITIAL_PROMPT} {
    # ログイン完了
  }

# Password: が返ってきたら
} "\[Pp\]assword:" {

  # パスワードを送信
  send -- "${JUMP_PASSWORD}\r"
  expect -re "${INITIAL_PROMPT}"
  # ログイン完了

# プロンプトが返ってきたら、ユーザ名もパスワードもなしにログインできたということ
} ${INITIAL_PROMPT} {
  # ログイン完了
} default {
  exit 2
} timeout {
  exit 2
}

# 次のルータに接続

send -- "telnet ${REMOTE_HOST}\r"

# Username: または Login: が返ってきたら
expect -re "\[Uu\]sername:|\[Ll\]ogin:" {

  # ユーザ名を送信して
  send -- "${REMOTE_USERNAME}\r"

  # Password: が戻ってきたら
  expect -re "\[Pp\]assword:" {

    # パスワードを送信
    send -- "${REMOTE_PASSWORD}\r"
    expect -re ${INITIAL_PROMPT}

  # プロンプトが返ってきたら、パスワードなしでログインできたということ
  } ${INITIAL_PROMPT} {
    # ログイン完了
  }

# Password: が返ってきたら
} "\[Pp\]assword:" {

  # パスワードを送信
  send -- "${REMOTE_PASSWORD}\r"
  expect -re "${INITIAL_PROMPT}"
  # ログイン完了

# プロンプトが返ってきたら、ユーザ名もパスワードもなしにログインできたということ
} ${INITIAL_PROMPT} {
  # ログイン完了
} default {
  exit 2
} timeout {
  exit 2
}

#
# Cisco機器を前提とした処理
#

# enableコマンドで特権モードに遷移
send -- "enable\r"
expect -re "\[Pp\]assword:" {
  send -- "${REMOTE_ENABLE}\r"
  expect -re "${INITIAL_PROMPT}"
} ${INITIAL_PROMPT} {
}

# terminal length 0 を実行してMOREを抑制
send -- "terminal length 0\r"
expect -re "${INITIAL_PROMPT}"

#
# ここから指定されたコマンドを実行
#

# 画面表示を開始
log_user 1

# プロンプトを設定
send -- "\r"
expect -re "${INITIAL_PROMPT}"
set CURRENT_PROMPT [string trim $expect_out(buffer)]

# プロンプトが何に設定されたのか確認するための画面表示
# puts "${CURRENT_PROMPT}"
# exit 0

# コマンドを実行する
# 引数の先頭8個は接続に必要な情報で、実行したいコマンドはその後ろにある
set i 8
while { [lindex $argv $i ] != "" } {
  # 引数からコマンドを取り出す
  set CMD [lindex $argv $i ]

  # コマンドを打ち込む
  send -- "${CMD}\r"
  expect -re ${CURRENT_PROMPT}
  incr i

  # command output is in $expect_out(1,string)
  # expectでログファイルに残すなら、ここで。

  send -- "\r"
  expect -re ${CURRENT_PROMPT}

}

# 画面表示を停止
log_user 0

#
# ログアウト処理
# Cisco機器が前提なので、exitコマンドで切断する。
#
send -- "exit\r"
expect -re "${INITIAL_PROMPT}"

#
# 踏み台からログアウト
# Cisco機器が前提なので、exitコマンドで切断する。
#
send -- "exit\r"

# 終了
expect eof
```
