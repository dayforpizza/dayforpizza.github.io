---
title: "Stateful vs Stateless"
excerpt: "Stateful과 Stateless는 정책의 순서와 관계가 있다"
header:
  teaser: /assets/posts/images/2023-10-03-10-14-16.png
categories:
  - Cloud Security
tags:
  - firewall
  - network
---

![](/assets/posts/images/2023-10-03-10-14-16.png)

## Stateful
- 정책을 넣는 순서가 영향을 준다.

> 예를 들면 UTM 또는 방화벽 등의 보안 장비는 정책을 적용 시 Top-down 방식으로 순서에 따라 어떻게 접근 제어를 설계하는 지가 중요하다.

- Whitelist 기반, Blacklist 기반 등 방화벽 정책을 다양한 방식으로 접근 제어를 구현할 수 있는 기반이 된다.

## Stateless
- 정책을 넣는 순서가 영향을 주지 않는다.

> 예를 들면 특정한 IP를 허용해주는 정책이 있고 그 위에 Any Deny가 있더라도 해당 IP는 허용이 된다.

- AWS의 Security Group이 대표적이다.