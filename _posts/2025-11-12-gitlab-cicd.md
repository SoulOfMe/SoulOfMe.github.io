---
layout: default
title: GitLab CICD
excerpt: Runner、Stages、Artifacts、变量与部署的系统实践
tags: [GitLab, CI, CD]
---

## 概述

GitLab CI/CD 以 `.gitlab-ci.yml` 描述流水线，通过 Runner 执行 Job。合理设计阶段与制品可实现构建、测试、发布的自动化闭环。

## Runner 与执行环境

Runner 类型：Shell、Docker、Kubernetes。推荐使用 Docker Runner 以隔离环境。

注册：

```bash
gitlab-runner register
```

示例配置（Docker Runner）：

```toml
[[runners]]
  name = "docker-runner"
  url = "https://gitlab.com/"
  token = "TOKEN"
  executor = "docker"
  [runners.docker]
    image = "docker:24"
    privileged = true
    volumes = ["/var/run/docker.sock:/var/run/docker.sock","/cache"]
```

## `.gitlab-ci.yml` 结构

关键元素：`stages`、`jobs`、`image`、`services`、`artifacts`、`cache`、`rules`、`only/except`、`variables`。

```yaml
stages:
  - build
  - test
  - release

variables:
  DOCKER_DRIVER: overlay2

build:
  stage: build
  image: node:20-alpine
  script:
    - npm ci
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 week

test:
  stage: test
  image: node:20-alpine
  needs: ["build"]
  script:
    - npm test -- --ci
  dependencies:
    - build

release:
  stage: release
  image: docker:24
  services:
    - docker:24-dind
  rules:
    - if: $CI_COMMIT_TAG
  script:
    - docker build -t registry.example.com/app:$CI_COMMIT_TAG .
    - docker push registry.example.com/app:$CI_COMMIT_TAG
```

## 变量与密钥管理

在项目或组的 CI/CD Settings 中设置 `Variables`，区分 `Protected` 与 `Masked`。在流水线中以环境变量读取。

示例：`REGISTRY_USER`、`REGISTRY_PASSWORD`、`KUBE_CONFIG`。

## 缓存与制品

`cache` 适合依赖缓存（如 `node_modules`）；`artifacts` 用于跨 Job 传递构建产物。

```yaml
cache:
  paths:
    - node_modules/
  key: ${CI_COMMIT_REF_SLUG}
```

## Docker 镜像构建与安全

在 Docker Runner 中通过 `docker:dind` 构建镜像；建议固定基础镜像版本并启用镜像扫描。

```yaml
release:
  stage: release
  image: docker:24
  services:
    - docker:24-dind
  variables:
    DOCKER_TLS_CERTDIR: ""
  script:
    - docker login -u $REGISTRY_USER -p $REGISTRY_PASSWORD registry.example.com
    - docker build -t registry.example.com/app:$CI_COMMIT_SHA .
    - docker push registry.example.com/app:$CI_COMMIT_SHA
```

## 部署策略

环境与手动门禁：

```yaml
deploy_staging:
  stage: release
  when: manual
  environment:
    name: staging
    url: https://staging.example.com
  script:
    - ./scripts/deploy-staging.sh

deploy_prod:
  stage: release
  when: manual
  environment:
    name: production
    url: https://example.com
  script:
    - ./scripts/deploy-prod.sh
```

## 触发规则与分支策略

使用 `rules` 精确控制触发：

```yaml
rules:
  - if: $CI_PIPELINE_SOURCE == "merge_request_event"
  - if: $CI_COMMIT_BRANCH == "main"
```

## 自动化测试与报告

可生成 JUnit 或覆盖率报告并在 GitLab 显示：

```yaml
test:
  stage: test
  script:
    - npm test -- --ci --reporter=junit --coverage
  artifacts:
    reports:
      junit: junit.xml
    paths:
      - coverage/
```

## 常见问题与最佳实践

- 固化镜像与依赖版本，避免不可控的 `latest`
- 使用 `needs` 减少不必要的拉取与等待
- 明确分支与标签策略，区分预发布与正式发布
- Runner 权限最小化，敏感变量设为 `Protected` 并仅在受保护分支使用

## 模板示例

```yaml
stages: [build, test, release]

.default_job:
  image: node:20-alpine
  cache:
    paths: [node_modules/]

build:
  stage: build
  extends: [.default_job]
  script: ["npm ci","npm run build"]
  artifacts:
    paths: [dist/]

test:
  stage: test
  extends: [.default_job]
  needs: [build]
  script: ["npm test -- --ci"]

release:
  stage: release
  image: docker:24
  services: ["docker:24-dind"]
  rules:
    - if: $CI_COMMIT_TAG
  script:
    - docker login -u $REGISTRY_USER -p $REGISTRY_PASSWORD registry.example.com
    - docker build -t registry.example.com/app:$CI_COMMIT_TAG .
    - docker push registry.example.com/app:$CI_COMMIT_TAG
```

## 总结

通过合理的 Runner 配置、阶段划分、变量与制品管理，GitLab CI/CD 可实现稳定、可审计的持续交付流水线，并在多环境部署中保持一致性与安全性。
