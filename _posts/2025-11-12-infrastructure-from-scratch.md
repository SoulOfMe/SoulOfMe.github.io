---
layout: default
title: 从零开始搭建软硬件基础设施
excerpt: 架构设计、硬件清单、网络规划、基础服务、监控与安全的落地路线
tags: [Infra, Network, DevOps]
---

## 目标与范围

以中小团队为场景，搭建稳定、可扩展、可观测的软硬件基础设施，覆盖网络、身份、代码托管与交付、容器编排、监控日志、备份与安全。

## 总体架构

核心分层：接入层（网关与防火墙）、网络层（交换与 VLAN）、服务层（身份、DNS/DHCP/NTP）、应用层（代码托管与 CI/CD、制品仓库、镜像与容器）、可观测层（监控日志告警）、数据层（备份与恢复）。

## 硬件清单建议

- 网关/防火墙：具备千兆交换与策略路由，支持 VPN
- 管理交换机：L2/L3 管理型，支持 VLAN、链路聚合
- 服务器：2 台以上 x86，32G 内存，SSD/NVMe，冗余电源
- 存储：NAS，支持快照与远程备份（如 ZFS/Btrfs）
- 机柜与配电：UPS，合理布线与标识

## 网络规划

- 地址段：区分管理、服务器、办公、访客 VLAN
- DNS/DHCP：统一管理与保留地址池
- NTP：统一时间源，日志与证书依赖
- 访问控制：最小权限原则，明确南北向与东西向流量策略

## 基础服务

- 身份认证：AD/LDAP，集中账号与权限，接入 GitLab/Jenkins/Registry
- 名称解析与服务发现：DNS，内部域与记录规范
- 时间服务：NTP，高可用与上游冗余

## 代码托管与 CI/CD

- GitLab：项目管理、Issue、Merge Request 与权限模型
- Runner：Docker 执行环境，隔离构建与部署
- 制品仓库：Registry/Nexus，固化构建产物与依赖
- 流水线：分层构建、测试、发布与部署，区分环境并设立门禁

## 容器与编排

- 本地与准生产：Docker Compose，快速迭代与一致性
- 小规模编排：Kubernetes 或 K3s，带来服务发现、滚动更新与 HPA
- 入口网关：Traefik/Caddy/Ingress，证书自动化与按域路由

## 监控与日志

- 指标：Prometheus + Grafana，节点与应用指标采集与可视化
- 日志：Loki/ELK，集中式日志与检索
- 告警：Alertmanager，与 IM 或邮件集成

示例 Docker Compose：

```yaml
services:
  prometheus:
    image: prom/prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"
    depends_on:
      prometheus:
        condition: service_started
```

## 备份与容灾

- 策略：3-2-1 原则，多副本、多介质、异地备份
- 级别：数据库、对象存储、文件系统与配置
- 恢复演练：定期校验与演练，缩短 RTO 与 RPO

## 安全与零信任

- 身份为中心：SSO、多因素认证与细粒度授权
- 网络分段：VLAN 与安全域边界，微分段控制东西向流量
- 证书与密钥：集中管理与自动续期，密钥最小暴露面
- 审计：登录、变更、发布与访问的审计闭环

## 落地步骤与时间表

1. 第 1 周：网络与基础服务上线（DNS/DHCP/NTP/LDAP）
2. 第 2 周：GitLab 与 Runner，制品仓库与镜像仓库
3. 第 3 周：监控与日志栈，统一告警通道与看板
4. 第 4 周：应用容器化与编排，分环境发布与灰度
5. 第 5 周：备份与容灾策略落地，演练与文档化

## 成本估算与规模扩展

硬件一次性投入与持续电力/机房成本；软件可选社区或企业版。随团队规模与应用复杂度，逐步引入编排与服务网格等能力。

## 结语

从零到一的基础设施建设核心在于标准化与可观测性，先打好网络与基础服务，再逐步引入 CI/CD 与容器化，最终以监控、日志与安全闭环保障稳定运营。
