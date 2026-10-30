# Prowler Project

Prowler 오픈소스의 AWS 보안 검사 기능을 분석하고 기여한 내용을 정리하는 개인 기록입니다.

## 기여 목차

| # | 기여 내용 | Issue | Pull Request | 상태 |
|---|---|---|---|---|
| 1 | 포트별 Public ALB ingress 미탐 수정 | [#13209](https://github.com/prowler-cloud/prowler/issues/13209) | [#13210](https://github.com/prowler-cloud/prowler/pull/13210) | 관리자 리뷰 대기 중 |

## 1. 포트별 Public ALB ingress 미탐 수정

### 문제

`elbv2_internet_facing` 검사가 인터넷에 공개된 Application Load Balancer(ALB)를 검사할 때, `0.0.0.0/0` 또는 `::/0`에 TCP 443처럼 특정 포트만 열린 보안 그룹 규칙을 탐지하지 못했습니다.

### 원인

보안 그룹 검사 함수에 `all_ports=True`가 전달되지 않아, 포트 범위가 지정된 공개 TCP 규칙이 검사 대상에서 빠졌습니다. 모든 프로토콜을 허용하는 규칙은 기존 테스트를 통과했기 때문에 이 문제가 드러나지 않았습니다.

### 변경 사항

- `prowler/providers/aws/services/elbv2/elbv2_internet_facing/elbv2_internet_facing.py`
  - 공개 TCP ingress 검사에 `all_ports=True`를 추가했습니다.
- `tests/providers/aws/services/elbv2/elbv2_internet_facing/elbv2_internet_facing_test.py`
  - IPv4 `0.0.0.0/0` 및 IPv6 `::/0` 공개 TCP 규칙 테스트를 추가했습니다.
  - 사설망 `10.0.0.0/8` 규칙이 오탐되지 않는지 검증했습니다.
  - 테스트가 최신 EC2 보안 그룹 정보를 사용하도록 보완했습니다.
- `prowler/changelog.d/elbv2-public-tcp-ingress.fixed.md`
  - 변경 이력을 추가했습니다.

### 검증 결과

- 관련 검사 및 보안 그룹 테스트: `54 passed`
- 전체 ELBv2 테스트: `96 passed`
- Black, Flake8 및 `git diff --check` 통과
- CodeRabbit 검토 통과 및 Git 충돌 없음

### 현재 상태

PR은 열려 있으며 Prowler 관리자의 CI 승인과 코드 리뷰를 기다리고 있습니다. 병합되면 Issue #13209가 자동으로 종료됩니다.

## 관련 링크

- [Prowler](https://github.com/prowler-cloud/prowler)
- [Issue #13209](https://github.com/prowler-cloud/prowler/issues/13209)
- [Pull Request #13210](https://github.com/prowler-cloud/prowler/pull/13210)
