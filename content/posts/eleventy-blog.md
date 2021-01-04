---
title: Eleventy로 블로그 만들기 (1)
date: 2018-10-29 22:01:00
tags:
  - eleventy
  - github-pages
series: eleventy-blog
isCJKLanguage: true
---

## Introduction

예전에 야심찬 목표를 가지고 블로그를 만든 적이 있었다. 당시 블로그 작성을 위해 [Jekyll][jekyll]을 사용했는데, [Ruby][ruby]를 쓰던 관성이 조금 남아있기도 했고, static site generator에 대해 다양하게 알지 못하던 때였다.

최근에 블로그를 다시 살리려고 알아보는데, [NodeJS][nodejs] + [NPM][npm]의 의존성 관리에 비하면 [RubyGems][rubygems]는 [RVM][rvm]을 쓰지 않는 이상 시스템 라이브러리를 건드려야 되는 등 지저분하게 느껴져서, [Jekyll][jekyll] 외의 다른 static site generator를 찾다가 [Eleventy][eleventy]를 발견하게 됐다.

그리하여, 아직 완전히 익숙해지지 못한 [NodeJS][nodejs] 기반 static site generator인 [Eleventy][eleventy]를 이용해 블로그를 만들어보자는 대장정이 시작된 것이다.

[jekyll]:   https://jekyllrb.com/
[ruby]:     https://www.ruby-lang.org/
[nodejs]:   https://nodejs.org/
[npm]:      https://www.npmjs.com/
[rubygems]: https://rubygems.org/
[rvm]:      https://rvm.io/
[eleventy]: https://11ty.io/

## Background

Static Site Generator라고 불리는 일련의 툴들은, 일반적으로 웹 사이트 제작에 사용하는 [HTML][html]보다 조금 더 간결한 [Markdown][markdown]과 같은 마크업 언어를 사용해서 컨텐츠를 작성하면, [Jinja][jinja]나 [Liquid][liquid] 등의 템플릿 엔진을 통해 사이트를 _build_ 하는 기능을 수행한다. [WordPress][wordpress] 등의 CMS와의 차이점은, 요청이 있을 때마다 DB를 참조하여 보여줄 페이지의 내용을 생성하는 CMS와는 달리 static site generator는 전체 사이트 구조를 미리 생성해 두고 요청이 있을 때는 미리 생성해둔 페이지를 서비스한다는 차이가 있다. 미리 생성된 페이지를 서비스하기 때문에 잦은 업데이트가 없는 부분에서 유리하다.

이 바닥에서 제일 유명한 툴이 [GitHub Pages][github-pages]에서 지원해주는 [Jekyll][jekyll]로, [Ruby][ruby] 기반으로 [Markdown][markdown]과 [Liquid][liquid]를 이용해 사이트를 생성할 수 있다. 이 외에도 [Python][python] 기반의 [Hyde][hyde], [Go][golang] 기반의 [Hugo][hugo] 등이 있다. 이 외에도 [StaticGen](https://staticgen.com/) 사이트에서 다양한 static site generator들을 접할 수 있다.

이 중에서 특별히 [Eleventy][eleventy]를 선택한 이유는 다음과 같다.
* [NodeJS][nodejs]를 사용한다.
* 다양한 마크업 언어와 템플릿 엔진을 사용할 수 있다.
* 쓰는 사람이 _아직_ 없어 보인다.

[html]:         https://www.w3.org/html/
[markdown]:     https://www.markdownguide.org/
[jinja]:        http://jinja.pocoo.org/
[liquid]:       https://shopify.github.io/liquid/
[wordpress]:    https://wordpress.org/
[github-pages]: https://pages.github.com/
[python]:       https://www.python.org/
[hyde]:         https://hyde.github.io/
[golang]:       https://golang.org/
[hugo]:         https://gohugo.io/

## Installation

[Eleventy][eleventy]는 site generator일 뿐으로, 블로그처럼 쓰기 위해서는 골격을 짜 줄 필요가 있다. 다만 우리는 바쁜 사람들로, 골격을 처음부터 짜려면 [Eleventy][eleventy]에 대해 심도있는 공부를 할 필요가 있기 때문에 미리 만들어진 골격으로부터 시작할 것이다.

[Eleventy][eleventy] 개발자 측에서 제공하는 간단한 골격이 이미 만들어져 있으므로 가져다 쓰도록 하자. 저장소는 [eleventy-base-blog](https://github.com/11ty/eleventy-base-blog)에서 볼 수 있다.

제일 먼저 해야 할 일은 [NodeJS][nodejs]와 [Yarn][yarn]을 설치하는 일이다. 저장소에 있는 README 파일에는 [NPM][npm]을 쓰라고 되어 있긴 하지만, NPM에 비해 Yarn이 훨씬 빠르고 편리하기 때문에 Yarn을 사용하도록 하자.
* [NodeJS Download](https://nodejs.org/en/download/)
* [Yarn Download](https://yarnpkg.com/en/docs/install)

설치가 끝나면 저장소를 클론할 차례다. 다음 명령어를 순차적으로 수행하고 나면 블로그 내용을 브라우저에서 볼 준비가 끝난다.

```bash
git clone https://github.com/11ty/eleventy-base-blog my-blog
cd my-blog
yarn
yarn build --serve
```

그러면 창에 다음과 비슷한 메시지들이 출력될 것이다.

```bash
yarn run v1.10.1
$ eleventy --serve
Writing _site/sitemap.xml from ./sitemap.xml.njk.
Writing _site/feed/.htaccess from ./feed/htaccess.njk.
Writing _site/feed/feed.xml from ./feed/feed.njk.
Writing _site/index.html from ./index.njk.
Writing _site/posts/firstpost/index.html from ./posts/firstpost.md.
Writing _site/posts/fourthpost/index.html from ./posts/fourthpost.md.
Writing _site/posts/secondpost/index.html from ./posts/secondpost.md.
Writing _site/posts/thirdpost/index.html from ./posts/thirdpost.md.
Writing _site/about/index.html from ./about/index.md.
Writing _site/tags/index.html from ./tags-list.njk.
Writing _site/posts/index.html from ./archive.njk.
Writing _site/404.html from ./404.md.
Writing _site/tags/another-tag/index.html from ./tags.njk.
Writing _site/tags/second-tag/index.html from ./tags.njk.
Writing _site/tags/number-2/index.html from ./tags.njk.
Writing _site/tags/tagList/index.html from ./tags.njk.
Copied 2 items and Processed 16 files in 0.69 seconds (43.1ms each)
Watching…
[Browsersync] Access URLs:
 ----------------------------
 Local: http://localhost:8080
 ----------------------------
    UI: http://localhost:3001
 ----------------------------
[Browsersync] Serving files from: _site
```

Local 옆에 있는 주소를 브라우저에 입력하면 블로그 내용을 볼 수 있다. `from` 오른편에 있는 파일 이름이 생성된 파일에 대한 소스 파일이고, 왼쪽에 있는 파일이 소스 파일로부터 생성된 웹페이지 파일이다. 궁금하니 이것 저것 열어서 어떤 구조로 되어 있는지 잘 살펴보자.

[yarn]: https://yarnpkg.com/

## How to Post

전체 구조는 천천히 살펴보기로 하고, 먼저 포스팅을 하려면 어떻게 해야할지를 살펴보자.

`./posts` 디렉토리를 살펴보면, 미리 만들어져 있는 4개의 포스팅이 보인다. 열어보게 되면 맨 앞엔 static site generator 대부분이 사용하는 [front matter][front-matter] 포맷의 헤더가 위치하고 있고, 그 밑에는 markdown 파일인 만큼 [Markdown][markdown]으로 작성된 포스트의 본체가 있는 것을 볼 수 있다.

간단하게 사용하려면 front matter의 `title` 필드와 `date` 필드를 적절하게 고치고, `tags` 필드에 필요한 태그를 달아준 뒤 내용을 써 보도록 하자. 처음 eleventy를 실행할 때 `--serve` 옵션을 줬기 때문에, 수정 후 저장만 하면 전체 사이트가 다시 빌드될 것이다. 그리고 열어놓은 브라우저도 Browsersync를 통해 자동으로 업데이트 될 것이다.

[front-matter]: https://www.11ty.io/docs/data-frontmatter/

## Future Work

다음 포스팅에서는, 포스트를 제외한 나머지 부분을 적당히 손보는 방법에 대해서 다루고자 한다.

## Conclusion

요즘에는 옛날에 비해 홈페이지를 만드는 방법에 있어서 어마어마한 선택지를 놓고 고를 수 있게 되었다. 그 중에서 요즘 널리 쓰이는 Modern Javascript도 배우고 문서 작업에 편리한 Markdown도 배울 수 있는 Eleventy로 사이트를 구축하는 법에 대해 맛을 보았다. 앞으로 블로그 업데이트와 함께 계속해서 쓸모없는 정보들을 포스팅해보도록 하겠다.
