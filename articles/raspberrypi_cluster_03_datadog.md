---
title: "Raspberry Piによるクラスター構築（DataDog編）" # 記事のタイトル
emoji: "🍓" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["raspberrypi", "ansible", "datadog"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---
「[Raspberry Piによるクラスター構築（Ansible + k3s編）](https://zenn.dev/akrisn/articles/raspberrypi_cluster_02_ansible)」の続き。

前回のAnsibleでのk3sのインストールに続き、DataDogの設定を行い、Raspberry Piクラスターの監視を行う。

## DataDogアカウントの作成

DataDogのアカウントがなくては始まらないので、[DataDogのサイト](https://www.datadoghq.com/ja/)に行き、右上の「無料で試す」からアカウントを作成する。登録にクレジットカードは不要で、2週間はフルの機能が使え、2週間経った後も5台までは無料ということから、今回のRaspberry Pi 4台は無料の範囲で使える。（そもそも、Raspberry PiはIoTデバイス扱い）

>  今回は無料の範囲内の話ですが、個人的にはいいサービスにはお金を払うべきだと思っています。

### APIキーの取得

1. ログインしたら、左下のTeamを選択し、
2. 上のタブから[Application Keys](https://app.datadoghq.com/access/application-keys)を選ぶ。
3. +New Keyボタンを押し、APIキーを作成する。


## AnsibleによるDataDog Agentのインストール。

DataDogからansible用のrole（[DataDog/ansible-datadog](https://github.com/DataDog/ansible-datadog)）が公開されているので、Ansible Galaxy経由でインストールする。

前回作成したk3s用のAnsibleディレクトリーのトップで（あるいは、コピーをとってrolesの中を空にしてから）

```bash
ansible-galaxy install datadog.datadog
```

`roles/datadog.datadog` にダウンロードされる。

DataDog用のプレイブック `datadog.yml` を次のように作成。

```yaml
---

- hosts: k3s_cluster
  become: yes
  roles:
    - datadog.datadog
  vars:
    datadog_api_key: "YOUR_DATADOG_API_KEY"
    datadog_agent_flavor: "datadog-iot-agent"
```

`YOUR_DATADOG_API_KEY`は上記「APIキーの取得」で作成したもの。

作成したプレイブックをプレビジョンする。

```bash
ansible-playbook datadog.yml
```

## DataDog Dashboard

DataDogへ行き、左のメニューから[Dashboards > Dashboards list](https://app.datadoghq.com/dashboard/lists)を選ぶ。

プリセットされたダッシュボードのSystem - Metricsを開くと、次のようなダッシュボードを見ることができる。

![](/images/articles/DataDogSystemMetrics.png)
