---
title: オンラインでホストされる手順
permalink: index.html
layout: home
---

# コンテンツ ディレクトリ:ring0_addr:SAP ワークロードのための Microsoft Azure の計画と管理

必要なラボ ファイルは、[ここからダウンロードできます](https://github.com/MicrosoftLearning/AZ-120-Planning-and-Administering-Microsoft-Azure-for-SAP-Workloads /archive/master.zip)

各ラボの演習とデモへのハイパーリンクを以下に示します。

## ラボ

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/'" %}
| モジュール | ラボ |
| --- | --- | 
{% for activity in labs %}| {{ activity.lab.module }} | [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}
