---
title: Eleventy로 블로그 만들기 (3)
date: 2018-11-24 12:01:00
tags:
  - eleventy
  - github-pages
series: eleventy-blog
isCJKLanguage: true
---

## Introduction

지난시간까지 CSS까지 입혀서 블로그를 이쁘게 꾸며보는데까지 진행했다. 이제는 블로그 글에 대한 신빙성을 높여주고 읽는 사람에게 나를 어필할 수 있는 [About]({{< ref "/about" >}}) 페이지를 만들어보도록 하자.

## Background

About 페이지를 만드는 것도 여러 방식이 있을 수 있겠지만, 결국 About 페이지에 담기는 것은 term과 그 term에 대한 설명들이 될 것이므로, 어느 정도 정형화된 형태를 가질 것이라는 것을 예상해볼 수 있다.

정형화된 틀에 맞춰서 데이터 중심으로 페이지를 설계하는 것이 바로 template이고, eleventy는 바로 그 template 기능을 주로 사용하는 툴이므로 아주 적합한 사용방법이 아닐 수 없다.

### Definition List

HTML에는 term과 description의 pair들을 모아 list로 만들어주는 태그가 존재한다. 바로 [`<dl>`][dl] 라는 태그인데, 자식 노드로 `<dt>`, `<dd>` 태그를 통해서 definition list를 구성할 수 있다. 하지만 곤란한 점이 있는데, [markdown][markdown]의 기본 정의에는 definition list를 지원하지 않는다는 점이다.
물론 용도가 확실하고 수요가 있는 태그들이라 여러 커스텀 마크다운 해석기에서 지원하기도 하는데, 제일 보편적인 형태는 다음과 같이 정의하는 방식을 사용한다.

```markdown
term
: definition
```

이렇게 사용할 경우 다음처럼 표시된다.

> term
> : definition

다만, eleventy에서 기본적으로 사용하고 있는 마크다운 해석기인 [markdown-it][markdown-it]은 가장 기본적인 markdown 문법만을 지원하고, definition list같은 확장 문법은 플러그인을 통해 활성화시켜야 한다. 해당 플러그인의 이름은 [`markdown-it-deflist`][markdown-it-deflist]이며, `npm`이나 `yarn`을 통해 설치할 수 있다.

```bash
yarn add markdown-it-deflist
```

```javascript {hl_lines=[3,17]}
  /* Markdown Plugins */
  let markdownIt = require("markdown-it");
  let markdownItAnchor = require("markdown-it-anchor");
  let markdownItDeflist = require("markdown-it-deflist");
  let options = {
    html: true,
    breaks: true,
    linkify: true
  };
  let opts = {
    permalink: true,
    permalinkClass: "direct-link",
    permalinkSymbol: "#"
  };

  eleventyConfig.setLibrary("md", markdownIt(options)
    .use(markdownItAnchor, opts)
    .use(markdownItDeflist)
  );
```

`.eleventy.js`의 대략 43번째줄 가량부터 [markdown-it][markdown-it]의 플러그인을 설정하고 있는 부분을 볼 수 있다. 원래는 [markdown-it-anchor][markdown-it-anchor] 플러그인만 추가되어 있지만, definition list를 지원하는 플러그인인 [markdown-it-deflist][markdown-it-deflist]를 추가한 것이다. 이렇게 바꾸고 나면 그 뒤부터는 definition list를 사용할 수 있다.

[dl]:                  https://www.w3.org/TR/html50/grouping-content.html#the-dl-element
[markdown]:            https://www.markdownguide.org/
[markdown-it]:         https://markdown-it.github.io/
[markdown-it-anchor]:  https://github.com/valeriangalliat/markdown-it-anchor
[markdown-it-deflist]: https://github.com/markdown-it/markdown-it-deflist

### YAML as a data file

eleventy는 template에 사용되는 [데이터파일을 템플릿과 별개로 관리][eleventy-data]할 수 있다. 일반적으로는 템플릿의 front matter로 기록하지만, 동일한 템플릿을 서로 다른 데이터로 사용하기 위해 [분리할 수 있도록][eleventy-template-data-file] 되어 있다. 다만, 지원하는 데이터 파일의 포맷은 `.json` 파일과 `.js` 파일이 전부인 상태로, 특별히 확장할 수 있는 API를 현재로썬 지원하고 있지 않다.
front matter는 de facto standard인 YAML로 작성할 수 있게 한다 쳐도, 템플릿 데이터를 YAML로 사용할 수 없는 것은 상당히 불편한 일이다. 물론 `.js`는 evaluate되므로 `.js`로 YAML을 읽어와서 반환하는 방식으로 템플릿 데이터를 만들 수도 있겠지만, 불필요한 코드가 들어가고 watch를 위한 수정사항 tracking도 통하지 않으므로 기각하였다.
그래서 eleventy의 내부 동작을 잘 살펴보고 적당한 [monkey-patching](https://en.wikipedia.org/wiki/Monkey_patch)을 통해 `.yaml` 파일도 template data file 로 사용할 수 있도록 해 주는 플러그인을 제작했다.

```bash
yarn add eleventy-plugin-yamldata
```

```javascript {hl_lines=[4,10]}
const { DateTime } = require("luxon");
const pluginRss = require("@11ty/eleventy-plugin-rss");
const pluginSyntaxHighlight = require("@11ty/eleventy-plugin-syntaxhighlight");
const pluginTOC = require('eleventy-plugin-toc');
const pluginYamldata = require('eleventy-plugin-yamldata');

module.exports = function(eleventyConfig) {
  eleventyConfig.addPlugin(pluginRss);
  eleventyConfig.addPlugin(pluginSyntaxHighlight);
  eleventyConfig.addPlugin(pluginTOC);
  eleventyConfig.addPlugin(pluginYamldata);
```

다음에 기회가 있으면 플러그인을 만드는 과정에 대해서도 포스팅하도록 하겠다.

[eleventy-data]: https://www.11ty.io/docs/data/
[eleventy-template-data-file]: https://www.11ty.io/docs/data-template-dir/

## 'About page' design

About 페이지는 여러 개의 섹션으로 나누고, 각 섹션마다 주제와 설명, 그리고 세부 항목들을 나열하려고 한다. 즉, 다음과 같은 형식이 될 것이다.

> **Section**
>
> term
> : _description_ &horbar; period
>   * item
>   * item

참고로, Section은 `## Section` 이 되어야 하겠으나, 이 문서의 구조를 깨버리기 때문에 강조하는 것으로 대체했다.

이러한 점을 감안해서, about page의 데이터는 다음과 같이 형태를 갖추도록 구성할 것이다.

```yaml
sections:
  - title: Section 1
    entries:
      - title: Term 1
        description: Description 1
        period: 2016 to Present
        items:
          - Item 1
          - Item 2
      - title: Term 2
        description: Description 2
        period: 2010 to 2016
  - title: Section 2
    entries:
      - title: ...
```

`sections` 키 안에 모든 섹션을 리스트로 넣고, 각 섹션은 `title`과 `entries`를 갖는다. `entries`는 term과 description을 가지며, 섹션과 비슷한 방식으로 `term`이 정의된다.

그러면 이걸 표현할 nunjucks + markdown 템플릿을 구성해보면 다음처럼 쓸 수 있을 것이다.

```markdown
{% for section in sections %}
## {{ section.title }}
{% for entry in section.entries %}
{% if entry.title -%}
{{ entry.title }}
{% if entry.description %}: _{{ entry.description }}_{% if entry.period %} &horbar; {{ entry.period }}{% endif %}{% endif -%}
{% if entry.score %}: {% for i in range(5) %}{% if i < entry.score %}&starf;{% else %}&star;{% endif %}{% endfor %}{% endif %}
{%- for item in entry.items %}
  * {{ item }}
{%- endfor %}
{% else -%}
* {{ entry | safe }}
{% endif %}
{% endfor %}
{% endfor %}
```

구체적인 태그들에 대한 설명은 [Nunjucks Templating Docs][nunjucks-template]를 참조하자.

[nunjucks-template]: https://mozilla.github.io/nunjucks/templating.html

## Future Work

문서 도중에 언급한 yaml data plugin의 구조에 대해서 포스팅 할 것이 남아 있다. 그리고 지금껏 만든 블로그를 [github pages][github-pages] 서비스를 통해 발행하는 방법까지 하고 나면 블로그 시리즈는 대단원의 막을 내릴 수 있을 것 같다.

[github-pages]: https://pages.github.com

## Conclusion

About page에 한정시켜서 만든 것이지만, 정형화된 구조를 가진 데이터를 특정 형태에 맞게 출력하는데 유리한 template과 template data file을 이용해 페이지를 만들어 보았다. 이를 통해서 보여주는 부분과 실제 데이터를 분리했기 때문에 추후 리뉴얼이나 다른 곳에 적용시킬 때 활용도가 높도록 나누어주는 것을 할 수 있을 것이다.
