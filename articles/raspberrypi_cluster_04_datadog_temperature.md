---
title: "Raspberry Piによるクラスター構築（温度監視）" # 記事のタイトル
emoji: "🍓" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["raspberrypi", "ansible", "datadog"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---
「[Raspberry Piによるクラスター構築（Datadog編）](https://zenn.dev/akrisn/articles/raspberrypi_cluster_03_datadog)」の続き。

前回のAnsibleでのDatadogの設定に続き、Raspberry PiのCPU温度を監視する。

## Raspberry PiのCPU温度

Raspberry PiのCPU温度は `/sys/class/thermal/thermal_zone0/temp` を見ると分かります。

```bash
$ cat /sys/class/thermal/thermal_zone0/temp
34563
```

1000倍された摂氏の値になっていることが分かります。

## DatadogへCLIで値を送る

Pythonのdatadogパッケージをインストールすると、`dog`コマンドがインストールされます。

`dog`コマンドを使うには `~/.dogrc`の設定が必要です。

```
[Connection]
apikey = DatadogのAPIキー
appkey = DatadogのAPPキー
```

これらは前回作成したと思いますので説明は省きます。

`dog`コマンドを用いると、次のようにしてカスタムメトリクスをDatadogに送ることができます。

```bash
$ dog metric post your.metric.name 1234
```

そこで、Raspberry PiのCPU温度を送るには次のようなスクリプトを用意すればよさそうです。

```bash
#!/bin/bash

TEMP=$(cat /sys/class/thermal/thermal_zone0/temp)
dog metric post system.cpu.temp $TEMP
```

## Ansibleによるサービスの設定

上記のスクリプトをクラスターにインストールして、サービスによって定期実行します。
前回作成したdatadog-ansibleディレクトリーをまるっとコピーして再利用します。

```bash
$ cp -R datadog-ansible send-temp-ansible
$ cd send-temp-ansible
$ rm -rf roles  # rolesの下は作り直します
$ mkdir roles
```

クラスターに配置するファイルを準備します。まず、上記で説明したファイルから

roles/send-temp/templates/dogrc.j2
```
[Connection]
apikey = {{ datadog_api_key }}
appkey = {{ datadog_app_key }}
```

roles/send-temp/templates/temp2datadog.sh.j2
```bash
#!/bin/bash

TEMP=$(cat /sys/class/thermal/thermal_zone0/temp)
dog metric post system.cpu.temp $TEMP
```

サービス定義を次のように

roles/send-temp/templates/temp2datadog.service.j2
```
[Unit]
Description=Send CPU temperature to DataDog
After=network-online.target

[Service]
Type=onehsot
ExecStart=/usr/local/bin/temp2datadog.sh

[Install]
WantedBy=multi-user.target
```

一定時間毎に起動する設定も

roles/send-temp/templates/temp2datadog.timer.j2
```
[Unit]
Description=Send CPU temperature to DataDog

[Timer]
OnCalendar=*-*-* *:*:00
Unit=temp2datadog.service

[Install]
WantedBy=timers.target
```

これらのファイルを配置するとともにsystemdを設定します。

roles/send-temp/tasks/main.yaml
```yaml
---

- name: Install Python3
  apt:
    name:
      - python3
      - python3-pip
    
- name: pip datadog
  pip:
    executable: /usr/bin/pip3
    name:
      - datadog

- name: Copy .dogrc
  template:
    src: "dogrc.j2"
    dest: "/root/.dogrc"
    owner: root
    group: root
    mode: 0600

- name: Copy temp2datadog
  template:
    src: "temp2datadog.sh.j2"
    dest: "/usr/local/bin/temp2datadog.sh"
    owner: root
    group: root
    mode: 0755
  
- name: Copy temp2datadog service file
  template:
    src: "temp2datadog.service.j2"
    dest: "{{ systemd_dir }}/temp2datadog.service"
    owner: root
    group: root
    mode: 0644

- name: Copy temp2datadog timer file
  template:
    src: "temp2datadog.timer.j2"
    dest: "{{ systemd_dir }}/temp2datadog.timer"
    owner: root
    group: root
    mode: 0644

- name: Reload systemd
  systemd:
    daemon_reload: yes

- name: Enable a service unit for temp2datadog
  systemd:
    name: temp2datadog.service
    enabled: yes

- name: Enable a timer unit for temp2datadog
  systemd:
    name: temp2datadog.timer
    enabled: yes

- name: Start a service unit for temp2datadog
  systemd:
    name: temp2datadog.service
    state: started

- name: Start a timer unit for temp2datadog
  systemd:
    name: temp2datadog.timer
    state: started
```

最後に、このroleを用いるためのplaybookを

main.yml
```
---

- hosts: k3s_cluster
  become: yes
  roles:
    - send-temp
  vars:
    datadog_api_key: "DatadogのAPIキー"
    datadog_app_key: "DatadogのAPPキー"
```

前回からのコピーを含めて次のようなファイルツリーができていると思います。

```
$ tree .
.
├── LICENSE
├── README.md
├── ansible.cfg
├── inventory
│   ├── my-cluster
│   │   ├── group_vars
│   │   │   └── all.yml
│   │   └── hosts.ini
│   └── sample
│       ├── group_vars
│       │   └── all.yml
│       └── hosts.ini
├── main.yml
└── roles
    └── send-temp
        ├── tasks
        │   └── main.yml
        └── templates
            ├── dogrc.j2
            ├── temp2datadog.service.j2
            ├── temp2datadog.sh.j2
            └── temp2datadog.timer.j2
```

では、実行してみます。

```bash
$ ansible-playbook main.yml

PLAY [k3s_cluster] ****************************************************************

TASK [Gathering Facts] ************************************************************
Saturday 16 October 2021  15:47:48 +0900 (0:00:00.022)       0:00:00.022 ******
ok: [rasp1]
ok: [rasp2]
ok: [rasp3]
ok: [rasp4]

TASK [send-temp : Install Python3] ************************************************
Saturday 16 October 2021  15:48:02 +0900 (0:00:13.437)       0:00:13.459 ******
ok: [rasp2]
ok: [rasp4]
ok: [rasp3]
ok: [rasp1]

TASK [send-temp : pip datadog] ****************************************************
Saturday 16 October 2021  15:48:05 +0900 (0:00:03.140)       0:00:16.600 ******
ok: [rasp3]
ok: [rasp4]
ok: [rasp2]
ok: [rasp1]

TASK [send-temp : Copy .dogrc] ****************************************************
Saturday 16 October 2021  15:48:11 +0900 (0:00:06.480)       0:00:23.080 ******
ok: [rasp2]
ok: [rasp4]
ok: [rasp3]
ok: [rasp1]

TASK [send-temp : Copy temp2datadog] **********************************************
Saturday 16 October 2021  15:48:13 +0900 (0:00:01.930)       0:00:25.010 ******
changed: [rasp1]
changed: [rasp3]
changed: [rasp4]
changed: [rasp2]

TASK [send-temp : Copy temp2datadog service file] *********************************
Saturday 16 October 2021  15:48:15 +0900 (0:00:01.921)       0:00:26.932 ******
changed: [rasp1]
changed: [rasp2]
changed: [rasp4]
changed: [rasp3]

TASK [send-temp : Copy temp2datadog timer file] ***********************************
Saturday 16 October 2021  15:48:17 +0900 (0:00:01.738)       0:00:28.670 ******
changed: [rasp1]
changed: [rasp2]
changed: [rasp4]
changed: [rasp3]

TASK [send-temp : Reload systemd] *************************************************
Saturday 16 October 2021  15:48:19 +0900 (0:00:01.840)       0:00:30.510 ******
ok: [rasp3]
ok: [rasp2]
ok: [rasp4]
ok: [rasp1]

TASK [send-temp : Enable a service unit for temp2datadog] *************************
Saturday 16 October 2021  15:48:22 +0900 (0:00:03.280)       0:00:33.791 ******
changed: [rasp1]
changed: [rasp2]
changed: [rasp4]
changed: [rasp3]

TASK [send-temp : Enable a timer unit for temp2datadog] ***************************
Saturday 16 October 2021  15:48:25 +0900 (0:00:03.105)       0:00:36.897 ******
changed: [rasp1]
changed: [rasp2]
changed: [rasp4]
changed: [rasp3]

TASK [send-temp : Start a service unit for temp2datadog] **************************
Saturday 16 October 2021  15:48:28 +0900 (0:00:03.116)       0:00:40.014 ******
changed: [rasp1]
changed: [rasp2]
changed: [rasp4]
changed: [rasp3]

TASK [send-temp : Start a timer unit for temp2datadog] ****************************
Saturday 16 October 2021  15:48:30 +0900 (0:00:01.547)       0:00:41.561 ******
changed: [rasp1]
changed: [rasp2]
changed: [rasp3]
changed: [rasp4]

PLAY RECAP ************************************************************************
rasp1                      : ok=12   changed=7    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
rasp2                      : ok=12   changed=7    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
rasp3                      : ok=12   changed=7    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
rasp4                      : ok=12   changed=7    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

Saturday 16 October 2021  15:48:31 +0900 (0:00:01.401)       0:00:42.963 ******
===============================================================================
Gathering Facts ----------------------------------------------------------- 13.44s
send-temp : pip datadog ---------------------------------------------------- 6.48s
send-temp : Reload systemd ------------------------------------------------- 3.28s
send-temp : Install Python3 ------------------------------------------------ 3.14s
send-temp : Enable a timer unit for temp2datadog --------------------------- 3.12s
send-temp : Enable a service unit for temp2datadog ------------------------- 3.11s
send-temp : Copy .dogrc ---------------------------------------------------- 1.93s
send-temp : Copy temp2datadog ---------------------------------------------- 1.92s
send-temp : Copy temp2datadog timer file ----------------------------------- 1.84s
send-temp : Copy temp2datadog service file --------------------------------- 1.74s
send-temp : Start a service unit for temp2datadog -------------------------- 1.55s
send-temp : Start a timer unit for temp2datadog ---------------------------- 1.40s
```

少ししてからDatadogへ行き、[Metrics Explorer](https://app.datadoghq.com/metric/explorer)を開き、Graph:に`system.cpu.temp`を入力します。One graph per:に`host`を入力するとホストごとの温度が送られているのが確認できます。

![](/images/articles/DatadogTemp.png)
