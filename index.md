---
title: Instruções online hospedadas
permalink: index.html
layout: home
---

# Diretório de Conteúdo: AZ-120: Planejar e administrar cargas de trabalho do Microsoft Azure para SAP

Os arquivos de laboratórios necessários podem ser [baixados aqui](https://github.com/MicrosoftLearning/AZ-120-Planning-and-Administering-Microsoft-Azure-for-SAP-Workloads /archive/master.zip)

Hiperlinks para cada um dos exercícios de laboratório e demonstrações estão listados abaixo.

## Laboratórios

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/'" %}
| Módulo | Laboratório |
| --- | --- | 
{% for activity in labs  %}| {{ activity.lab.module }} | [{{ activity.lab.title }}{% if activity.lab.type %} — {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}
