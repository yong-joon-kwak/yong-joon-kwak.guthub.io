---
title: '블로그 제작 실패기'
description: 'Vue 블로그 제작 실패기'
pubDate: 'Jul 13 2025'
heroImage: '../../assets/blog-placeholder-3.jpg'
tags: ['Blog', 'Vue', 'Vite']
---

## 주의사항

이 블로그 포스트는 어디까지나 **실패기**입니다. 개인적으로 이런 행위는 더이상 하지 않으려고, 또한 왜 실패했는지 남기려고 적는 것 이니 참고하시고 이걸 따라해서 또 실패하지 말아주세요.

## 블로그

시작은 간단하게 개발블로그를 직접만들어보자는 아이디어에서 나왔다.

> 간단한 블로그니까 굳이 DB를 쓰지 않고, 프론트엔드에서만 처리할 수 있지않을까? MD 형식으로 그냥 쓰고 편집한걸 git에 올린다음에, 그걸 GitHub Page에서 보여주면 서버비용도 안들고, 제작도 간단하겠네?

프론트엔드에서 마무리하겠다는 의미로, 최초에는 그냥 Vue3를 이용해보려고 했다. 일단 Vue3 + Vite + TypeScript 환경으로 프로젝트를 시작했다.

## Vue3

간단하게 누구나 알법한 프로젝트를 구성했다. Vue3 와 Vue-rotuer를 이용해 제작했고, Vite를 이용해 빌드하였다. 당연하게도 테스트용 Vue 파일은 모두 성공했다. 이제 블로그 글들을 올릴 MD를 제작하고 이를 보여주면 될 일이었다.

우선 시도한건 Dynamic Import 였다.
>아래 코드는 어디까지나 실패한 코드입니다.
```js
import matter from 'gray-matter'

export function parseMarkdownFiles() {
  const posts = import.meta.glob('/post/*.md', { eager: true })
  
  return Object.entries(posts).map(([path, content]) => {
    const rawContent = content as string
    const { data, content: markdownContent } = matter(rawContent)
    
    return {
      id: path.split('/').pop()?.replace('.md', ''),
      title: data.title || '제목 없음',
      excerpt: data.excerpt || markdownContent.slice(0, 100) + '...',
      date: data.date || new Date().toISOString().split('T')[0],
      tags: data.tags || [],
      content: markdownContent
    }
  })
```

코드를 살펴보면 간단한 코드였다. 우선 MD 상단에 다음과 같이 메타데이터를 남긴다.

```MD
---
title: "Test Post"
date: "2024-01-15"
excerpt: "Test 포스트 입니다."
tags: ["test", "example"]
---
```

이걸 `post` 하위 디렉토리에 둔 뒤, posts를 불러오는 함수에서 import.meta.glob을 이용해 모두 가져온뒤 이걸 파싱하는 것이다. 생각해보면 단순했다. 문제는 여기서 발생한다.

> md 파일을 브라우저가 읽을 수 있는가?

생각해보면 간단한 문제이다. import.meta.glob 즉 Vite 의 Dynamic import로 MD파일을 가져왔다 치자. 그러면 그 다음엔? 이걸 사용자에게 어떻게 보여줄건가?

> string 처리해서 MarkDown 파싱하면 되는거아냐?

맞다. 근데 MD 위에 저런 메타데이터를 넣고 이를 파싱한 후 나머지 컨텐츠는 Html로 파싱하는 과정이 쉽지가 않았다. 더군다나 사용한 [gray-matter](https://www.npmjs.com/package/gray-matter)는 Node.js 전용 패키지였기에, 사용도 불가했다. 

그러면 방법은 다음과 같다.

1. MarkDown MetaData Parser를 직접 제작한다.
2. 나머지 MarkDown을 Html로 파싱하는 작업을 한다.

근데 이쯤되면 이생각이 든다. **굳이?**

굳이 MarkDown으로 적어서, 굳이 문법에 맞지도 않는 메타데이터를 안에 넣어서. 굳이 이걸 파싱해서, HTML로 보여준다? 왜? 이럴거면 걍 HTML로 만드는게 낫지 않나?

이 생각이 들자. 갑자기 머리가 하얘지면서 그냥 싹 지우고 다시 시작했다.

## 실패와 오해

물론 저대로 진행했어도 문제는 없다. 여러가지 해결책이 있었는지, Claude에게 물어봤다.

**나:** 나의 블로그 제작 실패기를 적고있어. 최초에는 Vue3 + Vite로 제작하려다가 MarkDown 형식 포스트를 적는 과정에서 좌절하고 실패한 내용이야. 너가 한번 보고 기술적으로 내가 잘못적은 부분이 있다면 바로 고치지말고 말해줘

**Claude:** 블로그 제작 실패기를 읽어보니, 몇 가지 기술적으로 잘못 이해하신 부분들이 있는 것 같습니다:

### 1. `gray-matter`의 브라우저 호환성

글에서 `gray-matter`가 "Node.js 전용 패키지"라고 하셨는데, 실제로는 브라우저에서도 사용할 수 있습니다. 대부분의 모던 번들러(Vite 포함)가 Node.js 패키지들을 브라우저에서 사용할 수 있도록 polyfill을 제공합니다.

### 2. `import.meta.glob` 반환값 타입

코드에서 `content as string`으로 타입 캐스팅하셨는데, `import.meta.glob`은 실제로는 모듈 객체를 반환합니다. 올바른 사용법은 다음과 같습니다:

```js
const posts = import.meta.glob('/post/*.md', { eager: true, as: 'raw' })
```

여기서 `as: 'raw'` 옵션을 사용하면 파일 내용을 문자열로 가져올 수 있습니다.

### 3. Vite의 Markdown 처리 능력

Vite 환경에서 Markdown 파일을 처리하는 것은 충분히 가능합니다. `@vitejs/plugin-md`나 `vite-plugin-md` 같은 플러그인들이 있어서 Markdown 파일을 Vue 컴포넌트로 변환하거나 HTML로 파싱하는 작업을 간단하게 할 수 있습니다.

### 4. 해결 방법이 존재했음

실제로는 다음과 같은 방법들로 해결할 수 있었습니다:

- Vite 플러그인 사용
- `marked`나 `markdown-it` 같은 브라우저 호환 마크다운 파서 사용
- Vite의 `?raw` 쿼리를 사용한 파일 내용 가져오기

결론적으로 Vue3 + Vite로 마크다운 블로그를 만드는 것은 충분히 가능했는데, 몇 가지 기술적 오해로 인해 포기하신 것 같습니다. 물론 지금 Astro로 만드신 것도 훌륭한 선택이지만요!

---

Claude가 했던 답변을 토대로 또 길을 찾았으면 가능했을지도 모른다. 하지만 그게 과연 성공이었을지도 의문이다. 좀 더 단순한 해결책이 있는지 찾아봤었고, 다른 방법으로 나는 해결했다.
