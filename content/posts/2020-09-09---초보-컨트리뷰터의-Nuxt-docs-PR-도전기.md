---
title: "[OpenSource] 초보 컨트리뷰터의 Nuxt docs PR 도전기"
date: "2020-09-09 22:26:00"
template: "post"
draft: false
category: "OpenSource"
tags:
  - "OpenSource"
  - "Nuxt.docs"
  - "Nuxt.js"
description: "Nuxt docs에 크롬 이슈가 있어 PR을 올려보았습니다."
---

# Nuxt docs가 크롬에서 깨지는 현상
### chrome에서 https://nuxtjs.org/ 를 열면 다음 이미지처럼 보이는 현상이 있었습니다.
 
<img src="https://user-images.githubusercontent.com/32301380/91999436-561f3b80-ed77-11ea-922a-47ed15fb38ad.png">

개발자 도구로 살펴보니, `svg`라는 클래스에 `width: 20px` style이 전역으로 적용되고 있었습니다.

<img src="https://user-images.githubusercontent.com/32301380/91999543-7b13ae80-ed77-11ea-9895-b55a5a93e11a.png">

오픈소스 컨트리뷰톤에서 활동중이던 차에 직접 PR을 올려볼까? 하는 생각이 스쳤습니다. 그래서 다짜고짜 nuxtjs.org repo를 fork했습니다.
dev 모드에선 해당 이슈가 없었는데 prod 모드에서 이슈가 재현되었고, `svg class`가 어디에서 오는건지 찾아봤습니다.
찾아보니, `AppModal.vue`이라는 컴포넌트에 있었습니다. 여기서 주목할만한 점은 `style`이 **전역으로 적용**되고 있었다는 점입니다.
정확한 빌드과정은 모르겠지만, 스타일이 전역으로 적용되면서 `html` 태그안에 `class`로 들어간 것 같았습니다.
그래서 `svg class`가 다른데에서도 쓰이나 전체검색을 해봤는데, `AppModal.vue`에서 밖에 쓰이지 않았습니다. ***(여기서 저는 마음속으로 소리를 질렀습니다.)***
`style`에 `scoped` 속성을 주고 빌드를 해보니 `html` 태그에서 `svg class`가 빠지는 것을 확인할 수 있었습니다.
```vue
./components/global/bases/AppModal.vue

...
<style>
.svg {
  width: 20px;
  color: black;
}
...
```
그렇게 PR을 올릴까 하다가 혹시 몰라서 디렉토리 안의 다른 파일들도 살펴보았는데, `BaseAlert.vue` 컴포넌트도 똑같이 `scoped` 적용이 되어있지 않았습니다.
이것을 바탕으로 다음 PR을 작성했습니다.

https://github.com/nuxt/nuxtjs.org/pull/639

PR template이 따로 없어서, 어떤 현상인지, 어떻게 고쳤는지, 제 생각을 작성했습니다.  
maintainer가 이슈를 재현할 순 없지만, 위의 스타일들이 scoped되어야 하는 것은 동의한다고 merge 해주셨습니다.  
### Vue에서 style에 scoped속성을 적절히 사용하는 것은 Vue의 기본 중 기본입니다. 이런 기본 개념으로 이슈를 해결할 수 있어서 뜻깊었고 기여가 항상 거창해야 하는 것은 아니라는 것을 느꼈습니다. 또한 제가 자주 사용하는 프로젝트에 기여할 수 있어서 좋았습니다.
