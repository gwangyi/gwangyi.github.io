---
title: Eleventy로 블로그 만들기 (2)
date: 2018-11-05 21:54:00
tags:
  - eleventy
  - github-pages
series: eleventy-blog
isCJKLanguage: true
---

## Introduction

지난 시간에 글만 쓸 수 있을만한 간단한 블로그를 실제로 만들어 보았다. 지난 시간에 예고했던 대로 기본 틀의 구조를 이곳저곳 살펴보고, 제일 먼저 블로그의 전체적인 분위기를 바꿔보도록 하자.

## Background

구조를 살펴보기 전에, 쓸만한 CSS 프레임워크들을 뒤져볼 것이다. 블로그 테마를 직접 꾸밀 수 있다면 아쉬울 것이 없겠지만, 나같이 디자인과는 담을 쌓은 사람들은 직접 만드는 것보단 많은 사람들이 쌓아올린 금자탑을 가져다 쓰는 편이 더 생산적이고 중요한 일에 집중하는데 도움을 줄 것이다. 무엇보다, 웹 기술이 온 세상을 집어삼키는 이 시점에서 CSS 프레임워크 쓰는 법을 조금만 배워놓으면, 앞으로의 삶이 편리해지리라는 확신을 가지고서 뒤져보도록 하자.

### Awesome CSS Frameworks

[Github][github]에는 워낙 다양한 오픈소스 프로젝트들이 널려 있어서 그 중에 마음에 드는 것을 찾을 수만 있다면 최고의 보물창고임에는 이론의 여지가 없다. 하지만 먼저 말했듯이, 너무 다양하기 때문에 원하는 것을 찾기가 쉽지 않다는 점에 착안, 특정 주제에 관련된 유명한 프로젝트들을 큐레이트해서 보여주는 다양한 저장소들 또한 존재한다. 아마도 최초의 큐레이팅 저장소의 이름이 그랬기 때문일 것 같지만, 그런 저장소들은 대부분 `awesome-something` 과 같은 이름을 하고 있는 경우가 많다.

Github에서 awesome css frameworks를 검색해보면, [{{% mdi `github-circle` %}}troxler/awesome-css-frameworks](https://github.com/troxler/awesome-css-frameworks/) 저장소가 제일 위에 나온다. 이 중에서 [Bootstrap][bootstrap], [Milligram][milligram], [Bulma][bulma] 같은 프레임워크들을 볼 수 있을 것이다.

[Bootstrap][bootstrap]은 보편적으로 많이 쓰이는 프레임워크이고, 당시 나왔을 때 나같은 디알못도 유려한 사이트를 만들 수 있도록 컴포넌트들을 미리 다 정의해 둠으로써 디자인보다는 내용에 집중할 수 있게 해줘서 엄청난 반향을 불러일으켰던 프레임워크이다. 다만 단점이 있다면, Javascript 프레임워크 중 하나인 [jQuery][jquery]와 강하게 결합되어 HTML/CSS만으로는 사이트를 제작할 수 없다는 것과, 이번 [Eleventy][eleventy] 웹사이트 제작과는 큰 상관이 없지만 다른 종류의 Javascript framework와 섞어 쓰기가 곤란하다는 점이 있다.

[Milligram][milligram]은 상당히 작은 footprint를 가진 프레임워크로, CSS 파일의 용량이 적기 때문에 빠르게 로딩되고, 지저분하게 태그마다 클래스 이름을 갖다붙이지 않아도 적절하게 처리해준다는 점은 장점이지만, 아무래도 전체적인 레이아웃을 잡는데는 기능이 좀 부족하다.

[Bulma][bulma]는 CSS 만으로 구성된 프레임워크이고, [SASS][sass]나 [LESS][less] 코드가 제공되어 커스터마이징하기 편리하며, 모듈화가 되어 있어 필요한 모듈만 가져다 빌드할 수도 있는 특징을 가지고 있다. [Bootstrap][bootstrap]에 비해 지원하는 컴포넌트들이 그다지 부족하지 않고, Javascript를 배제하여 특정 Javascript framework에 대한 의존성이 없어 원하는 Javascript framework를 선택할 수 있는 자유도를 갖는다. 그에 더해서, 특정 태그를 부여하면 일정 영역에 대해 [Milligram][milligram]처럼 태그마다 클래스 이름을 붙이지 않더라도 미려한 디자인으로 구성할 수 있기 때문에, 개인적인 판단으로는 [Bootstrap][bootstrap] 비슷한 거대한 CSS framework와 [Milligram][milligram] 같은 가벼운 CSS framework의 장점을 모았다고 생각한다.

원래, 최근 [Electron][electron]에 요즘 핫하다는 [Vue.js][vuejs]를 끼얹어서 뭘 만들어보려는데, [Vue.js][vuejs]는 [Bootstrap][bootstrap]과 달리 디자인 요소가 단 하나도 없다는 것을 알고서 [Bootstrap][bootstrap]을 끼얹으려고 했었다. 그런데 [Bootstrap][bootstrap]은 앞에서 언급했듯 [jQuery][jquery]와 강하게 결합되어 있어 [Vue.js][vuejs]와 함께 쓰기에는 모양새가 별로 좋지 않았다. 이를 대체할 수 있는 CSS만으로 된 framework를 찾다가 발견한게 [Bulma][bulma]였었다. 이번에 블로그를 만들면서 [Bulma][bulma]를 계속 쓸지 다른걸 찾아볼지 고민을 좀 했었는데, 아무리 찾아봐도 [Bulma][bulma]만한 것을 찾기 힘들어서 도로 돌아오게 되었다.

여담으로, [Bulma][bulma]라는 이름은 드래곤볼에 나오는 부르마의 영어식 이름이라고 한다.

[github]:    https://github.com
[bootstrap]: https://getbootstrap.com
[jquery]:    https://jquery.com
[eleventy]:  https://11ty.io
[milligram]: https://milligram.io
[bulma]:     https://bulma.io
[sass]:      https://sass-lang.com
[less]:      https://lesscss.org
[electron]:  https://electronjs.org
[vuejs]:     https://vuejs.org

### Template engine

[Eleventy][eleventy]는 static site generator의 일종으로, 문서를 작성하면 미리 만들어둔 적절한 템플릿과 함께 사이트 전체를 빌드해내는 것을 목적으로 한 프로그램이다. 이는 즉, [Eleventy][eleventy]를 잘 쓰려면 템플릿 엔진에 대한 이해도 어느 정도 필요하다는 말이 된다. [Eleventy][eleventy]가 지원하는 템플릿 언어는 몇 가지가 있는데, 내장된 템플릿 엔진은 [EJS][ejs], [Handlebars][handlebars], [Liquid][liquid], [Mustache][mustache], [Nunjucks][nunjucks] 등이 들어있다. 그 외에도 몇 가지 더 있는데, HTML 문법에서 많이 벗어나는 것은 일단 건너뛰도록 하자.

[EJS][ejs]는 [Ruby][ruby]의 [ERB][erb]에서 영감을 받은 것으로, 특별한 별도의 템플릿 문법 없이 [php][php]와 비슷한 느낌으로 Javascript를 이용해 템플릿을 만들 수 있게 해준다. [Handlebars][handlebars], [Liquid][liquid], [Mustache][mustache], [Nunjucks][nunjucks]는 서로서로 비슷하게 생겨서 영향을 받았지만, 키워드 면에서 조금씩 차이가 있다. 이 중 [Liquid][liquid]는 static site generator의 시초격 되는 [Jekyll][jekyll]에서 채택한 것이고, [Nunjucks][nunjucks]는 [Python][python]계에서 널리 쓰이는 템플릿 언어인 [Jinja][jinja]의 Javascript 포팅 버전이다. 한번 배운건 배우는데 들인 시간이 아까워서라도 마르고 닳도록 써먹어야 되므로, [Python][python]에서도 쓸 일이 있을 [Nunjucks][nunjucks]를 기본 템플릿 엔진으로 써보도록 하자.

한가지 주의할 것은, 앞에서 참조했던 [{{% mdi `github-circle` %}}11ty/eleventy-base-blog](https://github.com/11ty/eleventy-base-blog)의 기본 설정으론 html 파일은 [Nunjucks][nunjucks]를 쓰지만 md 파일은 [Liquid][liquid]로 지정되어 있다. 템플릿 엔진을 이것저것 쓰면 정신사나우니 이걸 [Nunjucks][nunjucks]로 바꿔서 쓰도록 하자. 바꾸고 싶을 때는 `.eleventy.js` 파일을 수정하면 된다.

```javascript
return {
...
    markdownTemplateEngine: 'njk',
...
};
```

원래 코드의 73번행 근처를 보면 이렇게 template engine을 바꿀 수 있도록 되어 있을 것이다. 이 때 주의할점은, [Nunjucks][nunjucks]나 [Liquid][liquid]나 비슷한 문법을 쓰기 때문에 많은 경우 특별히 손 대지 않아도 호환이 되지만, 몇 가지의 문법은 호환되지 않아 단순히 템플릿 엔진을 바꾸기만 하면 사이트가 만들어지지 않을 것이다. 이 글이 써지는 현재 주석 구문이 좀 달라서 수정할 필요가 있다. `404.md` 파일 안에 사용중이다. [Liquid][liquid]의 주석은 `{% comment %}} ... {% endcomment %}` 방식으로 걸지만, [Nunjucks][nunjucks]는 `{# ... #}` 을 사용한다.

[ejs]:        https://ejs.co
[php]:        https://www.php.net/
[handlebars]: https://handlebarsjs.com
[liquid]:     https://shopify.github.io/liquid
[mustache]:   https://mustache.github.io
[nunjucks]:   https://mozilla.github.io/nunjucks
[ruby]:       https://ruby-lang.org/
[erb]:        https://ruby-doc.org/stdlib-2.5.3/libdoc/erb/rdoc/ERB.html
[python]:     https://python.org/
[jinja]:      https://jinja.pocoo.org

## Applying Bulma to my blog

이제 [Bulma][bulma]를 블로그에 붙일 차례다. 아무 포스트나 잡고 열어보면, front matter에 `layout` 필드가 있는 것을 볼 수 있다. 포스트라면 `layouts/post.njk`라고 적혀있을텐데, 해당 파일이 일반적인 포스트의 껍데기 역할을 한다. 그런데 실제로 열어보면 내용물이 그다지 많지 않고 또 `layout` 필드로 다른 레이아웃을 지정하고 있는 것을 알 수 있다. 이렇게 타고 올라가다보면 보통 하나의 레이아웃으로 모이게 될텐데, 바로 `layout/base.njk` 파일이다.

`layout/base.njk`를 열어보면 전체 HTML 구조를 볼 수 있다. 수정 방향은, 이 구조를 최대한 살리면서 [Bulma][bulma]를 적용한 미려한 사이트를 만드는 것이다.

`layout/base.njk` 파일을 열어보면, css 파일과 feed를 지정하는 태그가 들어 있고, 본체인 `<body>` 태그 안에는 네비게이션 바가 들어 있는 `<header>`, 본문이 들어가는 `<main>`, 꼬릿말이 들어가는 `<footer>` 세 태그로 구성되어 있는 것을 볼 수 있다.

### Stylesheets

제일 먼저 해야할 일은, 기존 css를 비활성화시키고 [Bulma][bulma]를 추가하는 일이다.


```html {hl_lines=["4-5"]}
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>{{ renderData.title or title or metadata.title }}</title>
        <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/bulma/0.7.2/css/bulma.min.css" integrity="sha256-2pUeJf+y0ltRPSbKOeJh09ipQFYxUdct5nTY6GAXswA=" crossorigin="anonymous" />
        <link rel="stylesheet" href="https://unpkg.com/@mdi/font/css/materialdesignicons.min.css"/>
        <link rel="alternate" href="{{ metadata.feed.path | url }}" type="application/atom+xml" title="{{ metadata.title }}">
    </head>
```


[cdnjs][cdnjs]가 다양한 Javascript 라이브러리들을 CDN으로 제공해주긴 하지만, 없는 것들도 있다. [Bulma][bulma]는 [cdnjs][cdnjs]에서 구할 수 있지만, [MDI][mdi]는 [cdnjs][cdnjs]에서 못 찾아서 [unpkg][unpkg]에서 링크했다. [unpkg][unpkg]는 [npm][npm]에 업로드되어 있는 Javascript 라이브러리를 CDN 형식으로 서비스해주는 사이트이다. 이렇게 적용하고 새로고침을 해보면, 기존 css가 다 벗겨지고 [Bulma][bulma]의 기본적인 CSS만 적용되어 봐주기 힘든 모양으로 바뀔 것이다. 이제 여기서 HTML을 조금씩 수정해, 보기 좋은 형태로 순차적으로 고쳐나가 보자.

[cdnjs]: https://cdnjs.com/
[mdi]:   https://materialdesignicons.com/
[unpkg]: https://unpkg.com/
[npm]:   https://npmjs.com/

### HTML

이제 HTML 구조를 변경할 시간이다. header, main, footer 세 단계로 나뉘어진 구조를 유지하면서 [Bulma][bulma] 스타일을 씌우는 쪽으로 수정하자.

#### header

네비게이션 바 모양을 그대로 살려서 작업할 수도 있겠지만, 네비게이션 바는 [Bulma][bulma]에서 그다지 의미적으로 배치된 요소가 아니라는 단점이 있다. 타이틀도 들어가고, `<ul>` 기반의 네비게이션 바를 같이 쓸 수 있는 예제가 [Bulma][bulma] 구성요소 중에 있는데, [Fullheight hero in 3 parts](https://bulma.io/documentation/layout/hero/#fullheight-hero-in-3-parts)가 바로 그것이다. 이를 사용해보도록 하자. 우리가 쓸 부분은 제목 부분과 하단 탭 부분만 쓰고, `hero-head` 부분에 들어가는 네비게이션은 쓰지 않을 것이다.

```html
        <header>
            <h1 class="home"><a href="{{ '/' | url }}">{{ metadata.title }}</a></h1>
            <ul class="nav">
                {%- for nav in collections.nav | reverse -%}
                <li class="nav-item{% if nav.url == page.url %} nav-item-active{% endif %}"><a href="{{ nav.url | url }}">{{ nav.data.navtitle }}</a></li>
                {%- endfor -%}
            </ul>
        </header>
```

위와 같은 코드를 다음처럼 수정한다.

```html
        <header class="hero is-primary">
            <div class="hero-body">
                <h1 class="title home"><a href="{{ '/' | url }}">{{ metadata.title }}</a></h1>
                <h2 class="subtitle">{{ metadata.feed.subtitle }}</h2>
            </div>
            <div class="hero-foot">
                <nav class="tabs is-boxed">
                    <div class="container">
                        <ul>
                            {%- for nav in collections.nav | reverse -%}
                            <li{% if nav.url == page.url %}  class="is-active"{% endif %}><a href="{{ nav.url | url }}">{{ nav.data.navtitle }}</a></li>
                            {%- endfor -%}
                        </ul>
                    </div>
                </nav>
            </div>
        </header>
```

`<header>` 태그 전체가 hero가 될 것이므로 그에 맞는 클래스를 추가했다. 사이트 제목 부분이 hero의 title이 되었고, feed에서 쓰는 부제목을 추가했다. 이 두 부분은 hero의 body 부분이므로 `hero-body`로 싸여졌다. 네비게이션 부분은 hero의 탭으로 만들었는데, 이 것은 hero의 아랫부분에 들어가야 되므로 `hero-foot`으로 먼저 쌌고, 탭으로 만들기 위해 `tabs` 클래스가 추가된 `<nav>` 태그로 쌌다. `is-boxed` 클래스를 추가해 탭의 모양을 잡아주었다. `<ul>` 태그를 `container`로 감싸줘서 탭이 중앙으로 적당히 정렬되도록 했으며, `nav` 클래스가 [Bulma][bulma]에서 의미를 갖기 때문에 클래스를 빼주었다. 마지막으로, 활성화된 탭은 `is-active` 클래스를 사용해야하므로 `nav-item-active`에서 `is-active`로 바꿔주었다.

#### main

본문 부분은 다음과 같이 수정했다.

```html
        <main{% if templateClass %} class="{{ templateClass }}"{% endif %}>
            {{ content | safe }}
        </main>
```

```html
        <main class="section{% if templateClass %} {{ templateClass }}{% endif %}">
            <div class="container content">
                {{ content | safe }}
            </div>
        </main>
```


단순히 [`section`](https://bulma.io/documentation/layout/section) 클래스를 부여한 뒤 [`container`](https://bulma.io/documentation/layout/container) 클래스가 부여된 `<div>` 태그로 감쌌다. 안에는 본문만 들어갈 것이므로 [`content`](https://bulma.io/documentation/elements/content)를 부여했다.

#### footer

꼬릿말에는 저작권 표시와 나의 깃헙 프로필 페이지를 추가할 것이다. 기존의 `<footer>` 태그는 내용이 비어 있으니 지우고 다음 내용으로 대체하자.  [Bulma][bulma]는 [`footer`](https://bulma.io/documentation/layout/footer) 구성요소를 가지고 있다.

```html
        <footer class="footer">
            <div class="content has-text-centered">
                <p>
                Copyright &copy; {{ metadata.author.name }}
                {% if metadata.author.github %}
                    <a href="https://github.com/{{ metadata.author.github }}">
                        <span class="icon">
                            <i class="mdi mdi-github-circle"></i>
                        </span>
                    </a>
                {% endif %}
                </p>
            </div>
        </footer>
```


여기서 특이한 사항은, `_data/metadata.json` 파일에 `author` 필드 안에 `github` 필드가 있는지를 보고 분기하는 것인데, `_data/metadata.json`에 깃헙 정보를 추가하지 않았다면 깃헙 링크를 스킵하고 보여주지 않도록 처리했다.

### Post

기본 post 레이아웃은 포스트 내용을 보여주는데는 부족함이 없지만, 몇 가지 아쉬운 점이 있다. 먼저 포스트가 작성된 시각이 보이지 않고, 포스트에 달린 태그가 보이지 않는다. 또, 긴 포스팅일 경우 목차가 자동으로 작성되면 좋을 것이다. post 레이아웃은 `_includes/layouts/post.njk` 안에 들어 있다.

태그 목록과 포스팅 날짜는 post 레이아웃에는 존재하지 않지만, 포스트 목록에 대한 템플릿인 `_includes/postslist.njk` 파일에 나와 있으므로 이를 참조해서 진행하도록 하자.

#### Tag list

제목 바로 밑에는 해당 포스트에 달린 태그 리스트를 보여주도록 하자. 포스트 안으로 들어왔기 때문에 `tags` 변수를 사용하면 태그 리스트를 출력할 수 있다. 그런데 몇 페이지는 `tags`가 정의되어 있지 않을 수 있기 때문에, 조건부로 넣으면 될 것이다.

```html
{%- if tags %}
<p>
{%- for tag in tags -%}
{%- if tag != 'post' -%}
    {% set tagUrl %}/tags/{{ tag }}/{% endset %}
    <a href="{{ tagUrl | url }}" class="tag">{{ tag }}</a>
{%- endif %}
{%- endfor %}
</p>
{% endif -%}
```


[Bulma][bulma]의 `tag` 구성요소를 사용해서 태그 모양으로 달았다.

#### Posted date

`<time>` 태그를 이용해서 의미론적으로 구성하도록 하자. 다음과 같은 내용을 제목 밑에 넣으면 된다.


```html
{%- if date -%}
<time datetime="{{ date | htmlDateString }}">{{ date | readableDate }}</time>
{% endif %}
```


#### Table of Contents

목차는 추가 플러그인을 이용하는 것이 편리하다. `eleventy-plugin-toc` 플러그인이 이미 있으므로 이를 이용하도록 하자.

```bash
yarn add eleventy-plugin-toc
```

플러그인을 설치했으면 이용하도록 해 주어야 한다. `.eleventy.js` 파일을 열어서 다음 부분을 추가하도록 하자.

```js {hl_lines=[3, 8]}
const { DateTime } = require("luxon");
const pluginRss = require("@11ty/eleventy-plugin-rss");
const pluginSyntaxHighlight = require("@11ty/eleventy-plugin-syntaxhighlight");
const pluginTOC = require('eleventy-plugin-toc')

module.exports = function(eleventyConfig) {
  eleventyConfig.addPlugin(pluginRss);
  eleventyConfig.addPlugin(pluginSyntaxHighlight);
  eleventyConfig.addPlugin(pluginTOC);
```

이제 `toc` 필터를 이용할 수 있게 되었다. 다음 코드를 `_includes/layouts/post.njk`에 추가하도록 하자. `{{ content | safe }}` 위에 넣어주어야 목차가 본문 위에 위치할 수 있으므로 참고하자.


```html
{%- if 'post' in tags -%}
<nav class="box">
    <strong> Table of Contents </strong>
    {{ content | toc | safe }}
</nav>
{%- endif -%}
```


목차는 포스팅에만 나오게 하기 위해 `tags`에 `post`가 있을 때만 보여주도록 했다.

## Future Work

이제 사이트의 틀을 대강 갖추었으므로, 나의 소개 페이지를 만들어 볼 차례이다. 소개 페이지는 대강의 틀을 먼저 갖춘 뒤, 내용은 별도의 데이터파일로 작성 후 템플릿을 통해 모양새를 갖추는 식으로 진행할 예정이다.

## Conclusion

이번 포스팅을 통해서는, [Eleventy][eleventy]의 템플릿 구조를 간단하게 살핀 후 [Bulma][bulma]를 적용하여 사이트를 미려하게 꾸며보았다. 잘 되어 있는 오픈소스를 가져다 쓰면, 그럴듯한 사이트를 만드는 것도 그렇게 어려운 일이 아니라는 것을 볼 수 있었다.

분량 조절에 실패해 글의 내용이 너무 길어졌다. 다음에는 분량을 잘 조절할 필요가 있겠다.
