---
layout: single
title: "드림투두 로그인 기능 개편"
categories:
  - "로그인/인증"
tags:
  - "SEO"
  - "드림투두"
---

## 드림투두 로그인 기능 개편

드림투두 백엔드의 로그인 기능을 개편한다.

시나리오별 프론트엔드 / 백엔드 행동을 생각해보았다.

### 시나리오1. 최초에 Token 을 발급받는 경우

<table>
<thead>
<tr>
<th>-</th>
<th>Frontend</th>
<th>Backend</th>
<th>비고</th>
</tr>
</thead>
<tbody>
<tr>
<td></td>
<td>- 로그인을 한다.</td>
<td></td>
<td></td>
</tr>
<tr>
<td></td>
<td></td>
<td>- Access/Refresh Token 을 발급한다.<br>- Refresh Token 과 UserAgent 를 DB 에 저장한다.<br>- 요청을 처리하고 응답한다.</td>
<td></td>
</tr>
<tr>
<td></td>
<td>- Acceess/Refresh Token 을 저장한다.</td>
<td></td>
<td></td>
</tr>
<tr>
<td></td>
<td>- Access Token 으로 요청한다.</td>
<td></td>
<td></td>
</tr>
<tr>
<td></td>
<td></td>
<td>- Access Token 을 확인한다.<br>- 요청을 처리하고 응답한다.</td>
<td></td>
</tr>
</tbody>
</table>

Refresh Token 과 UserAgent 를 DB 에 저장하는 이유는 나중에 Refresh Token 으로 요청이 왔는데, UsetAgent 가 다를 경우 유출이라고 판단하여 삭제처리 하기 위함이다.

### 시나리오2. Access Token 만료 후 접속하는 경우

<table>
<thead>
<tr>
<th>-</th>
<th>Frontend</th>
<th>Backend</th>
<th>비고</th>
</tr>
</thead>
<tbody>
<tr>
<td></td>
<td>- 만료된 Access Token 으로 요청한다.</td>
<td></td>
<td></td>
</tr>
<tr>
<td></td>
<td></td>
<td>- Access Token 이 만료된 토큰이라고 응답한다.</td>
<td></td>
</tr>
<tr>
<td></td>
<td>- Refresh Token 으로 요청한다.</td>
<td></td>
<td></td>
</tr>
<tr>
<td></td>
<td></td>
<td>- Refresh Token 과 UserAgent 를 확인한다.<br>- Access/Refresh Token 을 갱신발급한다.<br>- 요청을 처리하고 응답한다.</td>
<td></td>
</tr>
<tr>
<td></td>
<td>- Acceess/Refresh Token 을 저장한다.</td>
<td></td>
<td></td>
</tr>
<tr>
<td></td>
<td>- Access Token 으로 요청한다.</td>
<td></td>
<td></td>
</tr>
<tr>
<td></td>
<td></td>
<td>- Access Token 을 확인한다<br>- 요청을 처리하고 응답한다.</td>
<td></td>
</tr>
</tbody>
</table>

Access/Refresh Token 을 갱신발급하는 이유는 만료일자를 연장시키기 위함이다.

### 시나리오3. Refresh Token 만료 후 접속하는 경우

<table>
<thead>
<tr>
<th>-</th>
<th>Frontend</th>
<th>Backend</th>
<th>비고</th>
</tr>
</thead>
<tbody>
<tr>
<td></td>
<td>- 만료된 Access Token 으로 요청한다.</td>
<td></td>
<td></td>
</tr>
<tr>
<td></td>
<td></td>
<td>- Access Token 이 만료된 토큰임이라고 응답한다.</td>
<td></td>
</tr>
<tr>
<td></td>
<td>- Refresh Token 으로 요청한다.</td>
<td></td>
<td></td>
</tr>
<tr>
<td></td>
<td></td>
<td>- Refresh Token 이 만료된 토큰이라고 응답한다.</td>
<td></td>
</tr>
<tr>
<td></td>
<td>- 로그인을 한다.</td>
<td></td>
<td></td>
</tr>
</tbody>
</table>

### 참고

- [https://tecoble.techcourse.co.kr/post/2021-10-20-refresh-token/](https://tecoble.techcourse.co.kr/post/2021-10-20-refresh-token/)

## 네이버 사이트 간단 체크

아직 엉성하지만 네이버 사이트 간단 체크 결과가 개선되었다.

Docusaurus Layout 컴포넌트를 이용해서 통과된 것이다.

[https://docusaurus.io/ko/docs/seo](https://docusaurus.io/ko/docs/seo) 에서 SEO 에 대한 설명을 볼 수 있다.
