---
title: "Starting With Hugo"
date: 2021-01-31T18:55:00+01:00
isCJKLanguage: true
---

## Introduction

예전에 야심찬 목표를 가지고 블로그를 만든 적이 있었다. 어딘가 기시감이 느껴지는 시작이긴 하지만... 신기하고 새로운 것을 배워보고자 하는 마음에, 남들이 다들 하는 [Ruby][ruby]로 된 [Jekyll][jekyll] 기반이 아니라 [NodeJS][nodejs] 기반 [11ty][11ty]를 써보기로 하고는 야심차게 시작했었다.

하지만 육아와 이직준비, 기타등등의 여러가지 이유로 인해서 얼마 안 하다가 관리를 포기해버렸고, 다시 보려고 봤더니 static site generator와 그 프로젝트 구성, 플러그인, 내용물 등등이 이리저리 얽히고 섥혀서 어마어마한 의존성과 그로 인한 보안 구멍 등등 신경쓸 것들이 너무 많아진 것을 알게 되었다.

그 와중에 예전에 작성했던 [플러그인][11ty-yaml]은 어느샌가 [11ty][11ty] 자체가 제공하는 기능으로 들어가있질 않나, 총체적 난국이었다. 좋아, 그럼 이제 [Go][golang]를 만든 회사에도 왔겠다, [Go][golang] 공부도 해볼겸 [Go][golang]로 작성된 유명한 static site generator인 [hugo][hugo]로 다시 갈아엎어보자! 하는 마음으로 [hugo][hugo]를 알아보기 시작했다.

[jekyll]:    https://jekyllrb.com/
[ruby]:      https://www.ruby-lang.org/
[nodejs]:    https://nodejs.org/
[11ty]:      https://11ty.io/
[11ty-yaml]: https://github.com/gwangyi/eleventy-plugin-yamldata
[golang]:    https://golang.org/
[hugo]:      https://gohugo.io/

## Background

Static Site Generator에 대한 내용은 [이전 블로그 포스팅][eleventy-blog]에서 대략적으로 다뤘으므로, 여기서는 [hugo][hugo]와 [11ty][11ty]의 차이점에 대해서 가볍게 언급하고 지나가고자 한다.

[11ty][11ty]는 앞 문단에서도 가볍게 언급하고 지나갔다시피, 블로그 프로젝트가 곧 node 프로젝트이며, 사용한 [11ty][11ty] 자체가 디펜던시로 물려있고 각 플러그인들도 마찬가지로 물려들어간다. 이건 뭘 의미하냐면, 블로그 프로젝트와 [11ty][11ty] 프레임워크, 그리고 플러그인이 하나의 큰 프로그램으로 묶여서 함께 구성된다는 것을 의미한다. 장점으로는 [11ty][11ty] 내부 구조가 투명히 보이기 때문에 플러그인의 제작 자유도가 높고 컨텐츠도 부분적으로 js로 작성이 가능한 만큼 표현력이 좋았다.

그에 비해 [hugo][hugo]는, [hugo][hugo]가 블로그 프로젝트에 직접적으로 물려있지 않다. 블로그 프로젝트는 [Go][golang] 프로젝트가 아니기 때문에 일반적인 [Go][golang] 프로젝트의 방법으로 빌드되거나 하진 않는다. [11ty][11ty]가 프레임워크 같은 느낌이라면 [hugo][hugo]는 말 그대로 Site Generator로써 동작한다.

[eleventy-blog]: {{< ref "/posts/eleventy-blog.md" >}}

## Installation

[아주 직관적인 튜토리얼][hugo-tutorial]이 공식 사이트에 올라와 있다.

[hugo][hugo]는 네이티브 바이너리를 생성하는 [Go][golang] 언어로 작성된 프로그램 답게, 미리 빌드된 바이너리를 바로 [배포][hugo-release]하고 있다. 맞는 OS와 CPU를 찾아서 다운받아 풀면 간단하게 실행파일을 얻을 수 있다. 몇몇 고급 기능(ex. SCSS)은 기본 빌드에서는 빠져있고 extended 빌드를 받아야 된다는 점을 잊으면 안된다.

[hugo-tutorial]: https://gohugo.io/getting-started/quick-start/
[hugo-release]:  https://github.com/gohugoio/hugo/releases

## Theme

공식 사이트에 [아주 다양한 테마][hugo-themes]들이 올라와 있다. 이 중에서 마음에 드는 것을 아무거나 고르자. 일반적으로, 테마들은 GitHub에서 배포되고 있으며 README에 어떻게 테마를 설정해줘야 하는지, 그리고 기본적인 설정파일의 골격은 어떻게 하면 되는지 안내되어 있으므로 참고해서 내 사이트에 맞게 수정해서 쓰면 된다.

참고로 이 사이트는 [etch][etch] 테마를 사용하고 있다. JS 디펜던시를 최소화하고 심플하며 dark 모드를 지원하는 것을 골랐다. 거기에 약간의 커스터마이즈가 더 붙어 있다.

테마에서 재미있는 점은, 테마를 사용하지 않고도 사이트를 구성할 수 있다는 점이다. 먼저 프로젝트가 자체적으로 가지고 있는 레이아웃이나 숏코드 등을 뒤져본 뒤에 지정된 테마들을 [둘러보면서 찾도록][hugo-lookup-order] 되어 있다. 즉, 여러 개의-0개를 포함하여- 테마들을 동시에 적용할 수 있다는 것이다! 물론 우선순위가 있어서 무작성 섞여 나오지는 않지만 일부 기능만 가진 테마를 다른 테마와 겹쳐 사용하는 것이 가능하다. 예를 들면 이 블로그가 사용하고 있는 [mdi][hugo-mdi]테마가 그런 종류의 테마로, material design icons 프로젝트에서 제공하는 아이콘을 삽입하는 숏코드만 제공하고 있다.

[hugo-themes]:       https://themes.gohugo.io/
[etch]:              https://github.com/LukasJoswiak/etch
[hugo-lookup-order]: https://gohugo.io/templates/lookup-order/
[hugo-mdi]:               https://github.com/mochaaP/hugo-shortcode-mdi

## How to Post

[QuickStart][hugo-tutorial] 문서에 있는 것처럼 `hugo new posts/my-first-post.md` 명령의 형태로 새로운 포스트를 등록할 수 있다. 일반적인 SSG들과 비슷하게 [front matter 헤더][hugo-front-matter]가 맨 앞에 위치한다. 지원되는 포맷은 yaml 외에도 toml, json 등도 가능하다.

[hugo-front-matter]: https://gohugo.io/content-management/front-matter/

## Publishing as GitHub Pages via GitHub Actions

그동안 추가된 [GitHub Actions][github-actions]는 서드파티 CI 툴 없이도 간편하게 CI 툴을 사용할 수 있게 해준다. 예전 블로그는 [Travis CI][travis-ci]를 이용해서 [블로그 빌드를 구성][old-blog-build]했었는데 이번에는 그 유명한 [GitHub Actions][github-actions]를 써보기로 했다.

이미 [GitHub Actions][github-actions]가 설정되지 않은 리포지토리라면, 상단 탭의 Actions를 누르면 추천하는 [Workflow][ga-workflow]들을 리스트로 보여준다. 이 중 하나를 눌러서 필요한 정보를 대충 채워넣으면 간단하게 CI를 구성할 수 있다.

[GitHub Actions][github-actions]는 대략적으로 설명하면, 미리 [Workflow][ga-workflow]를 구성해 놓으면 [Workflow][ga-workflow]에서 정의한 트리거가 발동될 때 미리 정의된 [Action][ga-action]이 순서대로 실행되는 서비스라고 볼 수 있겠다. 다른 CI툴과 달리 기본이 되는 이미지는 아무 이미지나 사용할 수 없는 것 같고, 미리 제공되는 몇 개의 이미지 중에서 골라야 한다.

[hugo][hugo]가 유명한 툴인만큼, 미리 정의된 템플릿은 GitHub에서 제공해주지 않지만 누군가가 [action][actions-hugo]을 만들어서 올려두었다. [mdi][hugo-mdi] 테마는 단순 clone만으로 동작하지 않고 추가 작업이 필요하기 때문에 액션을 추가해 줄 필요가 있었다.

{{< gist-it "gwangyi/gwangyi.github.io" "main" ".github/workflows/gh-pages.yml" >}}

[github-actions]: https://github.com/features/actions
[travis-ci]:      https://travis-ci.org/
[old-blog-build]: https://github.com/gwangyi/gwangyi.github.io/blob/source/.travis.yml
[ga-workflow]:    https://docs.github.com/en/actions/learn-github-actions/introduction-to-github-actions#workflows
[ga-action]:     https://docs.github.com/en/actions/learn-github-actions/introduction-to-github-actions#actions
[actions-hugo]:   https://github.com/peaceiris/actions-hugo

## Conclusion

[11ty][11ty]는 당시 나에게 여러모로 좋은 블로깅 툴이었던 것은 맞았다. 하지만 시간이 지나고 이것저것 환경이 바뀐 뒤에는, 역시 여유가 없을 땐 사람들이 많이 쓰는 것을 쓰는게 정신건강상 최고라는 것을 다시 한번 느꼈다.
