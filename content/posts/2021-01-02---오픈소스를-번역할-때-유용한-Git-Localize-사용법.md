---
title: "오픈소스를 번역할 때 유용한 Git Localize 사용법"
date: "2021-01-02 00:48:00"
template: "post"
draft: false
category: "Tools"
tags:
  - "Tools"
  - "Git Localize"
  - "Git 번역"
  - "오픈소스 번역"
  - "오픈소스"
  - "translation"
description: "오픈소스 번역을 도와주는 Git Localize의 사용법을 다룹니다."
---
![Git Localize](https://user-images.githubusercontent.com/32301380/103441476-9e048f80-4c91-11eb-92b2-90674cde683c.png)

### Git Localize는 Git Repository의 내용 번역을 쉽게 도와주는 사이트입니다. [Git Localize](https://gitlocalize.com/)

로그인을 해서 `Add Repository`를 누르면 번역할 대상을 등록할 수 있습니다.

![image](https://user-images.githubusercontent.com/32301380/103441527-005d9000-4c92-11eb-839f-7b103eb0a26d.png)
![image](https://user-images.githubusercontent.com/32301380/103441536-1703e700-4c92-11eb-9b6b-2c55980ddf92.png)

다음으로 번역할 Repository를 고르는데, 접근 권한 설정이 되어 있어야 보이겠죠?
Repository를 고른 뒤에는, 대상 Branch를 작성합니다. 저는 번역용 Branch를 따로 생성해서 PR을 생성할 것이기 때문에 제가 새로 만든 Branch이름을 적었습니다.

`default branch`는 `master`인데, master를 main으로 바꾸신 분들은 main을 적어주세요!

![image](https://user-images.githubusercontent.com/32301380/103441547-2daa3e00-4c92-11eb-8ec3-892e576fe574.png)
![image](https://user-images.githubusercontent.com/32301380/103441603-706c1600-4c92-11eb-8bed-44568d5c06cd.png)

`Project Paths` 항목에서는 Repository 안에서 `번역할 대상을 지정`할 수 있습니다.
`Directory`로 설정하면 해당 Directory에 있는 모든 파일이 대상이 되며, `File`을 선택하면 특정 File만 선택할 수 있습니다.

`%lang%`이라는 placeholder는 번역 세팅에 따라 `target language`로 자동 변환됩니다.
예를 들어서, `target language`를 `ko`로 설정한다면 `Microsoft-Web-Dev-For-Beginners`에 있는 파일들을 번역하면 `Microsoft-Web-Dev-For-Beginners/ko`에 저장됩니다.

![image](https://user-images.githubusercontent.com/32301380/103441644-b628de80-4c92-11eb-8831-e0b19a209bca.png)


저는 모든 파일들을 번역할 것이 아니라 특정 파일들만 번역할 예정이라 `File`로 Path들을 추가해나갔습니다.
`%lang%` 부분이 `target language`로 자동 변환된다고 했죠? 제가 설정한걸 예로 들어보면 
`assignment.md` 파일을 번역하면 `./translations/assignment.ko.md`파일로 저장된다고 볼 수 있습니다!

![image](https://user-images.githubusercontent.com/32301380/103441685-04d67880-4c93-11eb-8172-a961ca59c94c.png)


그 다음엔 `Project languages` 설정입니다. 제가 이 글을 쓰게 된 이유이기도 합니다.
`ko`라고 검색하면 두개가 나오는데 [ko-KR]을 선택하고 제출하면, **계속 로딩만 되고 아무것도 안됩니다.**
**위에 있는 `[ko]`를 선택하셔야 합니다.**
그리고 나서 Submit 버튼을 누르시면 정상적으로 등록됩니다. 😎 파일이 많을수록 로딩이 오래 걸리니 조금만 기다려주시면 됩니다. 준비가 되면 이메일로 알려줍니다.

![image](https://user-images.githubusercontent.com/32301380/103441755-4bc46e00-4c93-11eb-8326-a4ab650983df.png)


등록이 되면 리스트에 등록한 결과가 나오며, 여러가지 메뉴가 나옵니다.
Overview 메뉴에서는 번역 상태를 볼수 있고 번역 언어도 관리할 수 있습니다.

![image](https://user-images.githubusercontent.com/32301380/103441817-d7d69580-4c93-11eb-9c0c-dfea934aea23.png)

Team 메뉴에서는 함께 작업할 유저들을 초대하고 관리할 수 있습니다.

![image](https://user-images.githubusercontent.com/32301380/103441820-e0c76700-4c93-11eb-9485-7998c08219c2.png)

Settings 메뉴에서는 Project Paths를 수정할 수 있으며 Sync Repository, Repository 제거 등을 할 수 있습니다.

![image](https://user-images.githubusercontent.com/32301380/103441821-e58c1b00-4c93-11eb-9e28-2b63d79cbf04.png)

Badge 메뉴에서는 프로젝트의 README.md에 추가할 수 있는 귀여운 배지를 제공해주네요!

![image](https://user-images.githubusercontent.com/32301380/103441822-e8870b80-4c93-11eb-8a84-3c55ee287fb8.png)

제가 준비한 내용은 여기까지입니다. 이제는 번역하러 가볼까요? 읽어주셔서 감사하고 저처럼 세팅할 때 삽질하시는 분이 없길 바랍니다. 

