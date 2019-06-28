---
title:  "Identity Provider (WIP)"
classes: wide
date: 2018-02-06T13:15:00+09:00
categories: [devops, identity provider]
tags: [identity provider]
---


# 왜 필요한가?
- 백엔드 사용자와, 지원툴 사용자에게 통합적인 인증/권한 체계가 필요함.
- 백엔드 관리자 페이지, 서비스간 권한 인증, jenkins, sonarqube, kubernetes, spinnaker 등등.
- 왜 새로운 사람이 들어오면, 매번 권한 신청을 시스템별로 각각 해야만 하는가???
- 한군데서 인증과 권한을 관리하면 좋지 아니한가?


# 프로젝트명
CARINA

# 인증 방법 
- OAuth2 / OIDC (OpenId Connect)
- LDAP
- SAML

기본적으로 OIDC를 제공 하고, 과거의 제품들을 지원하기 위해 LDAP과 SAML도 제공하는게 좋다.

![CARINA](assets/img/2018/carina.png)

# 사용자 정보
사용자의 로그인 정보는 데이터베이스를 사용하거나, 다른 LDAP을 사용할 수 있도록 한다.
LDAP을 사용하는 회사는 많이 있다. 하지만 인사적인 측면에서의 LDAP 정보와 개발/제품 측면에서의 LDAP 정보는 다를 수 있기 때문에, 
로그인 정보로만 사용하고, 그룹 정보는 CARINA에서 별도로 정의해서 사용할 수 있도록 하는게 좋다.

# 그룹 정보
그룹 정보는 데이터베이스에 저장한다.
그룹의 멤버는(MEMBER)는 사용자(USER)만 가능하다.
그룹의 멤버로 그룹을 추가하는것이 필요할 수는 있지만, 구조의 복잡성으로 인해 일단은 기능을 제공하지 않는다.
그룹의 멤버의 권한은 소유자(OWNER), 관리자(ADMIN), 멤버(MEMBER)로 정의한다.


# APIs
- Name
- Identifier (Audience)
- Scope (scope, description)
- Consumers(Applications)
- Test

# Applications
- Name
- Type : MTM
- API and scopes
- Domain(?)
- ClientId / Client Secret
- Description

- Endpoint, Grant Types

- OpenID Configuration URL
- JSON Web Key Set
- OAuth Authorization URL
- OAuth Token URL
- OAuth User Info URL


# Claims 커스터 마이징 기능?

# Role 또는 Privileges 기반의 권한 부여 필요.
Group - Roles
User - Roles
Role - Privileges

Role이 많이 지면 JWT의 크기가 커질 수 있다. - 소유한 모든 Role이 아니라, 토큰 생성시 필요한 API 로 필터링 한다면 크기를 줄일 수 있다.

