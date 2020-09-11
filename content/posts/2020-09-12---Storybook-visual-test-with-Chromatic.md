---
title: "[Storybook] Storybook visual test with Chromatic"
date: "2020-09-12 00:11:00"
template: "post"
draft: false
category: "Storybook"
tags:
  - "Storybook"
  - "test"
  - "Chromatic"
description: "Chromatic을 이용해 Storybook을 클라우드에서 테스트하는 방법을 적어보았습니다."
---

### 하라는 컴포넌트화는 안하고 테스팅만 찾아보는 당신은 대체...

Storybook Tutorial에서 제시하는 테스트는 크게 세가지가 있다.
1. Visual tests : 직접 보고 의도대로 모양이 나왔는지 체크하는 방법
2. Snapshot tests : Storybook이 렌더된 markup의 snapshot을 찍어서 비교한다.
3. Unit tests : Jest를 사용해 같은 인풋에 대해 같은 아웃풋이 나오는지 체크한다.
 
 하지만 앞서 언급했던 방법들은 UI bug를 감지하기 힘들다. UI bug는 경우에 따라 굉장히 미미한 차이일 수도 있기 때문에, Visual tests는 너무 수동이고, Snapshot tests는 UI에 적용했을 때 너무 많은 false positive(실제론 false인데 positive라고 나오는)를 발생시키며, pixel-level unit tests는 의미가 없다. 그래서 Storybook은 `visual regression tests`를 제안한다.

## Visual regression testing for Storybook
`Visual regression tests`는 렌더링된 모양의 차이를 잡아내도록 디자인되었다. 모든 스토리의 스크린샷을 찍어서, 커밋별 변화점을 잡아낸다. 많은 visual regression testing 도구가 있지만, Storybook에선 클라우드에서 테스트를 할 수 있는 `Chromatic`을 제안한다!

## Setup visual regression testing

### Bring git up to date
먼저 storybook project의 git 저장소가 있어야 합니다! 기존에 만들었으면 상관없겠죠?

### Get Chromatic

- chromatic을 설치해줍니다.
```bash
yarn add -D chromatic
```

- chromatic에서 git으로 로그인을 하고, repository를 추가합니다.  
[Chromatic](https://www.chromatic.com/start?invite-token=dnjk1zik&utm_source=https://learnstorybook.com&utm_medium=tutorial&utm_campaign=learnstorybook)

- 그러면 `project-token`을 받을 수 있고, 해당 명령어를 실행해 deploy합니다.
```bash
npx chromatic --project-token=<project-token>
```

<details>
<summary>package.json에 build-storybook이 없다고 한다면?</summary>

```json
"scripts": {
	"build-storybook": "build-storybook"
},
```
</details>

그러고 나서 chromatic에 들어가면 정상적으로 배포된 것을 볼 수 있습니다.
![image](https://user-images.githubusercontent.com/32301380/92944506-03d4cd80-f48f-11ea-8b6b-9a6ea1befa5b.png)

진짜 chromatic의 기능은 여기부터 시작입니다. 아무 변화나 주고 다시 배포를 해봅시다. 저는 버튼의 background-color를 파랑에서 빨강으로 바꿔봤습니다.
그럼 다음처럼 변경점이 감지되었음을 보여줍니다. Review changes 버튼을 클릭해서 들어가봅시다.
![image](https://user-images.githubusercontent.com/32301380/92947467-ff121880-f492-11ea-9062-499e544c7400.png)

다음 처럼 UI 변경점을 보여줍니다. 바뀐 지점이 연두색으로 덮혀있습니다. DOM에서도 바뀐게 감지되었죠.
![image](https://user-images.githubusercontent.com/32301380/92947491-0afdda80-f493-11ea-8b97-186ef3ddd49a.png)

이렇게 배포할 때마다 UI의 변경점을 감지하고, 둘 중에 하나를 선택해서 적용할 수 있습니다. 내가 작업한 결과물이 기존것과 잘 매치되는지, 잘 변경되는지 등을 손쉽게 테스트할 수 있어 정말 유용한 서비스 같습니다.

## Manage
PR을 보냈을 때에도 똑같이 테스트를 해 볼 수 있네요! 정말 유용합니다.
![image](https://user-images.githubusercontent.com/32301380/92945175-e0f6e900-f48f-11ea-998c-0c75244dc55a.png)

브라우저별 테스트를 제공합니다. 제일 말썽인 IE가 포함되어 있네요. IE를 누르면 다음 가격 정책이 뜹니다.
![image](https://user-images.githubusercontent.com/32301380/92944752-5910df00-f48f-11ea-966b-49d3bec1e4b4.png)

그렇다면 가격 정책에 대해서 알아봅시다!
![image](https://user-images.githubusercontent.com/32301380/92944786-65953780-f48f-11ea-81bb-6642f8f3a1eb.png)
그만 알아봅시다 ㅠㅠ 너무 비싸네요. 그런데 Free인 5000 snapshots도 다 못 채울 것 같습니다.
