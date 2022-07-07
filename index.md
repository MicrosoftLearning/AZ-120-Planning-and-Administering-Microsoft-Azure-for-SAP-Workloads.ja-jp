---
title: オンライン ホステッド インストラクション
permalink: index.html
layout: home
ms.openlocfilehash: 56285fb7ae88b2fc1b606d089ce74bd77ab72b2a
ms.sourcegitcommit: eb50c95898e7261da18334a990e37721be6cb9a6
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/19/2022
ms.locfileid: "145883912"
---
# <a name="content-directory-az-120-planning-and-administering-microsoft-azure-for-sap-workloads"></a>コンテンツ ディレクトリ:ring0_addr:SAP ワークロードのための Microsoft Azure の計画と管理

必要なラボ ファイルは、[ここからダウンロードできます](https://github.com/MicrosoftLearning/AZ-120-Planning-and-Administering-Microsoft-Azure-for-SAP-Workloads /archive/master.zip)

各ラボの演習とデモへのハイパーリンクを以下に示します。

## <a name="labs"></a>ラボ

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/'" %}
| モジュール | ラボ |
| --- | --- | 
{% for activity in labs %}| {{ activity.lab.module }} | [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}
