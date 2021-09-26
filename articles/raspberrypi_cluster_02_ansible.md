---
title: "Raspberry Piによるクラスター構築（Ansible + k3s編）" # 記事のタイトル
emoji: "🍓" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["raspberrypi", "ansible", "kubernetes", "k3s"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---
「[Raspberry Piによるクラスター構築（ハード編）](https://zenn.dev/akrisn/articles/raspberrypi_cluster_01_hard)」の続き。

ハードが組み上がったので、使えるように設定していく。
今回はAnsibleでk3sをインストールするところまで。

## pssh

Ansibleを使うまででもない作業をさっと実行するのに便利な`pssh`をまずは使えるようにする。

> psshは `brew install pssh` でインストール

sshのkeyを各マシンにコピーしておく。

```bash
$ ssh-copy-id ubuntu@rasp1
$ ssh-copy-id ubuntu@rasp2
$ ssh-copy-id ubuntu@rasp3
$ ssh-copy-id ubuntu@rasp4
```

psshのためのhostsファイルを用意する。

```
ubuntu@rasp1
ubuntu@rasp2
ubuntu@rasp3
ubuntu@rasp4
```

psshを用いてhostnameを確認

```bash
$ pssh -h ./hosts -i hostname
[1] 20:14:32 [SUCCESS] ubuntu@rasp4
rasp4
[2] 20:14:32 [SUCCESS] ubuntu@rasp3
rasp3
[3] 20:14:32 [SUCCESS] ubuntu@rasp2
rasp2
[4] 20:14:32 [SUCCESS] ubuntu@rasp1
rasp1
```

これで、クラスターをまとめて落とすのも1行。
```bash
$ pssh -h ./hosts -i sudo shutdown -h now
```

## Ansibleによるk3sクラスターの構築

[k3s](https://rancher.com/products/k3s/)はRancher社による軽量なKubernetesです。

1台のマシンにインストールするだけなら、スクリプトを用いればいいが、クラスターだとノードの登録などの作業が必要となる。これらを手作業で行うと間違いも起きるためAnsibleを用いる。

k3s公式に[k3s-ansible](https://github.com/k3s-io/k3s-ansible)が提供されているので、それを用いる。

```bash
git clone https://github.com/k3s-io/k3s-ansible.git
```

まず、インベントリーを作成する。

```bash
cp -R inventory/sample invenvtory/my-cluster
```

`inventory/my-cluster/hosts.ini`を編集。

```
[master]
rasp1 ansible_host=192.168.86.121

[node]
rasp2 ansible_host=192.168.86.122
rasp3 ansible_host=192.168.86.123
rasp4 ansible_host=192.168.86.124

[k3s_cluster:children]
master
node
```

`inventory/my-cluster/group_vars/all.yml`中で`k3s_version`が設定されている。
[k3s](https://github.com/k3s-io/k3s)のTagsをみて必要なら新しいバージョンを設定。
また、`ansible_user`を`ubuntu`に。

```
k3s_version: v1.21.5+k3s1
ansible_user: ubuntu
```

`ansible.cfg`のinventoryを編集

```
inventory = inventory/my-cluster/hosts.in
```

以上で、プロビジョニングを始めればクラスターが構築されます。

```bash
ansible-playbook site.yml
```

> cgroupの設定のため(`tasks/prereq/Ubuntu.yml`)、一度、Raspberryのリブートが入りますが、慌てず見守りましょう。

### 実行ログ
```
$ ansible-playbook site.yml

PLAY [k3s_cluster] *************************************************************

TASK [Gathering Facts] *********************************************************
Tuesday 21 September 2021  10:16:31 +0900 (0:00:00.018)       0:00:00.018 *****
ok: [rasp3]
ok: [rasp2]
ok: [rasp4]
ok: [rasp1]

TASK [prereq : Set SELinux to disabled state] **********************************
Tuesday 21 September 2021  10:16:43 +0900 (0:00:12.010)       0:00:12.029 *****
skipping: [rasp1]
skipping: [rasp2]
skipping: [rasp3]
skipping: [rasp4]

TASK [prereq : Enable IPv4 forwarding] *****************************************
Tuesday 21 September 2021  10:16:43 +0900 (0:00:00.222)       0:00:12.252 *****
ok: [rasp2]
ok: [rasp3]
ok: [rasp4]
ok: [rasp1]

TASK [prereq : Enable IPv6 forwarding] *****************************************
Tuesday 21 September 2021  10:16:44 +0900 (0:00:00.829)       0:00:13.081 *****
ok: [rasp1]
ok: [rasp2]
ok: [rasp3]
ok: [rasp4]

TASK [prereq : Add br_netfilter to /etc/modules-load.d/] ***********************
Tuesday 21 September 2021  10:16:44 +0900 (0:00:00.843)       0:00:13.924 *****
skipping: [rasp1]
skipping: [rasp2]
skipping: [rasp3]
skipping: [rasp4]

TASK [prereq : Load br_netfilter] **********************************************
Tuesday 21 September 2021  10:16:45 +0900 (0:00:00.197)       0:00:14.122 *****
skipping: [rasp1]
skipping: [rasp2]
skipping: [rasp3]
skipping: [rasp4]

TASK [prereq : Set bridge-nf-call-iptables (just to be sure)] ******************
Tuesday 21 September 2021  10:16:45 +0900 (0:00:00.192)       0:00:14.314 *****
skipping: [rasp1] => (item=net.bridge.bridge-nf-call-iptables)
skipping: [rasp1] => (item=net.bridge.bridge-nf-call-ip6tables)
skipping: [rasp2] => (item=net.bridge.bridge-nf-call-iptables)
skipping: [rasp2] => (item=net.bridge.bridge-nf-call-ip6tables)
skipping: [rasp3] => (item=net.bridge.bridge-nf-call-iptables)
skipping: [rasp3] => (item=net.bridge.bridge-nf-call-ip6tables)
skipping: [rasp4] => (item=net.bridge.bridge-nf-call-iptables)
skipping: [rasp4] => (item=net.bridge.bridge-nf-call-ip6tables)

TASK [prereq : Add /usr/local/bin to sudo secure_path] *************************
Tuesday 21 September 2021  10:16:45 +0900 (0:00:00.250)       0:00:14.565 *****
skipping: [rasp1]
skipping: [rasp2]
skipping: [rasp3]
skipping: [rasp4]

TASK [download : Download k3s binary x64] **************************************
Tuesday 21 September 2021  10:16:45 +0900 (0:00:00.358)       0:00:14.923 *****
skipping: [rasp1]
skipping: [rasp2]
skipping: [rasp3]
skipping: [rasp4]

TASK [download : Download k3s binary arm64] ************************************
Tuesday 21 September 2021  10:16:46 +0900 (0:00:00.216)       0:00:15.140 *****
changed: [rasp4]
changed: [rasp2]
changed: [rasp3]
changed: [rasp1]

TASK [download : Download k3s binary armhf] ************************************
Tuesday 21 September 2021  10:16:52 +0900 (0:00:06.577)       0:00:21.717 *****
skipping: [rasp1]
skipping: [rasp2]
skipping: [rasp3]
skipping: [rasp4]

TASK [raspberrypi : Test for raspberry pi /proc/cpuinfo] ***********************
Tuesday 21 September 2021  10:16:53 +0900 (0:00:00.410)       0:00:22.127 *****
ok: [rasp2]
ok: [rasp3]
ok: [rasp1]
ok: [rasp4]

TASK [raspberrypi : Test for raspberry pi /proc/device-tree/model] *************
Tuesday 21 September 2021  10:16:54 +0900 (0:00:01.853)       0:00:23.981 *****
ok: [rasp1]
ok: [rasp2]
ok: [rasp4]
ok: [rasp3]

TASK [raspberrypi : Set raspberry_pi fact to true] *****************************
Tuesday 21 September 2021  10:16:56 +0900 (0:00:01.752)       0:00:25.734 *****
ok: [rasp1]
ok: [rasp2]
ok: [rasp3]
ok: [rasp4]

TASK [raspberrypi : Set detected_distribution to Raspbian] *********************
Tuesday 21 September 2021  10:16:56 +0900 (0:00:00.238)       0:00:25.973 *****
skipping: [rasp1]
skipping: [rasp2]
skipping: [rasp3]
skipping: [rasp4]

TASK [raspberrypi : Set detected_distribution to Raspbian (ARM64 on Debian Buster)] ***
Tuesday 21 September 2021  10:16:57 +0900 (0:00:00.241)       0:00:26.214 *****
skipping: [rasp1]
skipping: [rasp2]
skipping: [rasp3]
skipping: [rasp4]

TASK [raspberrypi : Set detected_distribution_major_version] *******************
Tuesday 21 September 2021  10:16:57 +0900 (0:00:00.247)       0:00:26.461 *****
skipping: [rasp1]
skipping: [rasp2]
skipping: [rasp3]
skipping: [rasp4]

TASK [raspberrypi : execute OS related tasks on the Raspberry Pi] **************
Tuesday 21 September 2021  10:16:57 +0900 (0:00:00.267)       0:00:26.729 *****
included: /Users/akira/Works/src/github.com/stn/main/raspberrypi/k3s-ansible/roles/raspberrypi/tasks/prereq/Ubuntu.yml for rasp1, rasp2, rasp3, rasp4 => (item=/Users/akira/Works/src/github.com/stn/main/raspberrypi/k3s-ansible/roles/raspberrypi/tasks/prereq/Ubuntu.yml)

TASK [raspberrypi : Enable cgroup via boot commandline if not already enabled for Ubuntu on a Raspberry Pi] ***
Tuesday 21 September 2021  10:16:58 +0900 (0:00:00.396)       0:00:27.125 *****
changed: [rasp3]
changed: [rasp2]
changed: [rasp4]
changed: [rasp1]

RUNNING HANDLER [raspberrypi : reboot] *****************************************
Tuesday 21 September 2021  10:16:59 +0900 (0:00:00.955)       0:00:28.081 *****
changed: [rasp3]
changed: [rasp2]
changed: [rasp4]
changed: [rasp1]

PLAY [master] ******************************************************************

TASK [Gathering Facts] *********************************************************
Tuesday 21 September 2021  10:17:52 +0900 (0:00:53.591)       0:01:21.672 *****
ok: [rasp1]

TASK [k3s/master : Copy K3s service file] **************************************
Tuesday 21 September 2021  10:18:00 +0900 (0:00:08.180)       0:01:29.852 *****
changed: [rasp1]

TASK [k3s/master : Enable and check K3s service] *******************************
Tuesday 21 September 2021  10:18:02 +0900 (0:00:01.897)       0:01:31.750 *****
changed: [rasp1]

TASK [k3s/master : Wait for node-token] ****************************************
Tuesday 21 September 2021  10:18:47 +0900 (0:00:45.195)       0:02:16.945 *****
ok: [rasp1]

TASK [k3s/master : Register node-token file access mode] ***********************
Tuesday 21 September 2021  10:18:48 +0900 (0:00:01.021)       0:02:17.967 *****
ok: [rasp1]

TASK [k3s/master : Change file access node-token] ******************************
Tuesday 21 September 2021  10:18:50 +0900 (0:00:01.306)       0:02:19.274 *****
changed: [rasp1]

TASK [k3s/master : Read node-token from master] ********************************
Tuesday 21 September 2021  10:18:51 +0900 (0:00:00.893)       0:02:20.167 *****
ok: [rasp1]

TASK [k3s/master : Store Master node-token] ************************************
Tuesday 21 September 2021  10:18:51 +0900 (0:00:00.756)       0:02:20.924 *****
ok: [rasp1]

TASK [k3s/master : Restore node-token file access] *****************************
Tuesday 21 September 2021  10:18:51 +0900 (0:00:00.083)       0:02:21.007 *****
changed: [rasp1]

TASK [k3s/master : Create directory .kube] *************************************
Tuesday 21 September 2021  10:18:52 +0900 (0:00:00.668)       0:02:21.675 *****
ok: [rasp1]

TASK [k3s/master : Copy config file to user home directory] ********************
Tuesday 21 September 2021  10:18:53 +0900 (0:00:00.657)       0:02:22.333 *****
changed: [rasp1]

TASK [k3s/master : Replace https://localhost:6443 by https://master-ip:6443] ***
Tuesday 21 September 2021  10:18:54 +0900 (0:00:00.860)       0:02:23.194 *****
changed: [rasp1]

TASK [k3s/master : Create kubectl symlink] *************************************
Tuesday 21 September 2021  10:18:55 +0900 (0:00:01.648)       0:02:24.842 *****
ok: [rasp1]

TASK [k3s/master : Create crictl symlink] **************************************
Tuesday 21 September 2021  10:18:56 +0900 (0:00:00.776)       0:02:25.619 *****
ok: [rasp1]

PLAY [node] ********************************************************************

TASK [Gathering Facts] *********************************************************
Tuesday 21 September 2021  10:18:57 +0900 (0:00:00.767)       0:02:26.386 *****
ok: [rasp2]
ok: [rasp4]
ok: [rasp3]

TASK [k3s/node : Copy K3s service file] ****************************************
Tuesday 21 September 2021  10:19:08 +0900 (0:00:10.925)       0:02:37.312 *****
changed: [rasp2]
changed: [rasp3]
changed: [rasp4]

TASK [k3s/node : Enable and check K3s service] *********************************
Tuesday 21 September 2021  10:19:09 +0900 (0:00:01.627)       0:02:38.939 *****
changed: [rasp3]
changed: [rasp4]
changed: [rasp2]

PLAY RECAP *********************************************************************
rasp1                      : ok=24   changed=9    unreachable=0    failed=0    skipped=10   rescued=0    ignored=0
rasp2                      : ok=13   changed=5    unreachable=0    failed=0    skipped=10   rescued=0    ignored=0
rasp3                      : ok=13   changed=5    unreachable=0    failed=0    skipped=10   rescued=0    ignored=0
rasp4                      : ok=13   changed=5    unreachable=0    failed=0    skipped=10   rescued=0    ignored=0

Tuesday 21 September 2021  10:19:35 +0900 (0:00:25.394)       0:03:04.334 *****
===============================================================================
raspberrypi : reboot --------------------------------------------------- 53.59s
k3s/master : Enable and check K3s service ------------------------------ 45.20s
k3s/node : Enable and check K3s service -------------------------------- 25.39s
Gathering Facts -------------------------------------------------------- 12.01s
Gathering Facts -------------------------------------------------------- 10.93s
Gathering Facts --------------------------------------------------------- 8.18s
download : Download k3s binary arm64 ------------------------------------ 6.58s
k3s/master : Copy K3s service file -------------------------------------- 1.90s
raspberrypi : Test for raspberry pi /proc/cpuinfo ----------------------- 1.85s
raspberrypi : Test for raspberry pi /proc/device-tree/model ------------- 1.75s
k3s/master : Replace https://localhost:6443 by https://master-ip:6443 --- 1.65s
k3s/node : Copy K3s service file ---------------------------------------- 1.63s
k3s/master : Register node-token file access mode ----------------------- 1.31s
k3s/master : Wait for node-token ---------------------------------------- 1.02s
raspberrypi : Enable cgroup via boot commandline if not already enabled for Ubuntu on a Raspberry Pi --- 0.96s
k3s/master : Change file access node-token ------------------------------ 0.89s
k3s/master : Copy config file to user home directory -------------------- 0.86s
prereq : Enable IPv6 forwarding ----------------------------------------- 0.84s
prereq : Enable IPv4 forwarding ----------------------------------------- 0.83s
k3s/master : Create kubectl symlink ------------------------------------- 0.78s
```

## 動作確認

Kubeconfigを手元にコピーし、

```bash
scp ubuntu@rasp1:~/.kube/config ~/.kube/config
```

`kubectl`で確認。

```bash
$ kubectl top node
NAME    CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
rasp1   321m         8%     1038Mi          13%
rasp2   168m         4%     332Mi           4%
rasp3   196m         4%     361Mi           4%
rasp4   174m         4%     333Mi           4%
```

## k3sのアンインストール

アンインストールも簡単です。

```bash
ansible-playbook reset.yml
```

## 次回

「[Raspberry Piによるクラスター構築（DataDog編）](https://zenn.dev/akrisn/articles/raspberrypi_cluster_03_datadog)」に続く
