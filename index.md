---
title: GenAIOps 연습
permalink: index.html
layout: home
---

# 생성형 AI 애플리케이션 운영

다음 빠른 시작 연습은 Microsoft Azure에서 생성형 AI 워크로드를 운영하는 데 필요한 일반적인 작업을 탐색하는 실습 학습 환경을 제공하도록 설계되었습니다.

> **참고**: 연습을 완료하려면 필요한 Azure 리소스와 생성형 AI 모델을 프로비전할 수 있는 충분한 권한과 할당량이 있는 Azure 구독이 필요합니다. 아직 계정이 없는 경우 [Azure 계정](https://azure.microsoft.com/free)에 가입합니다. 첫 30일 동안 사용할 수 있는 신규 사용자용 무료 평가판 옵션이 있으며, 크레딧도 제공합니다.

## 빠른 시작 연습

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions'" %} {% for activity in labs  %}
<hr>
### [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }})

{{activity.lab.description}}

{% endfor %}

> **참고**: 이 연습은 단독으로 완료할 수도 있지만 원래 [Microsoft Learn](https://learn.microsoft.com/training/paths/operationalize-gen-ai-apps/)의 모듈에 대한 보충 자료로 설계되었으며, 여기에서 이 연습의 기반이 되는 몇 가지 기본 개념에 대해 자세히 알아볼 수 있습니다.
