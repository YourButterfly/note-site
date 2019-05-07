# Image

图片有关优化

## 图片缩放

simpleLightbox下载链接

```txt
https://github.com/dbrekalo/simpleLightbox/archive/2.1.0.tar.gz
```

jss,css放置，参考

```shell
$ tree docs
docs/
├── javascripts
│   ├── custom.js
│   ├── jquery-3.4.1.min.js
│   └── simpleLightbox.min.js
└── stylesheets
    ├── custom.css
    └── simpleLightbox.min.css
```

custom.css

```css
/* custom.css */
a.boxedThumb {
    display: block;
    padding: 4px;
    line-height: 20px;
    border: 1px solid #ddd;
    -webkit-border-radius: 4px;
    -moz-border-radius: 4px;
    border-radius: 4px;
    -webkit-box-shadow: 0 1px 3px rgba(0, 0, 0, 0.055);
    -moz-box-shadow: 0 1px 3px rgba(0, 0, 0, 0.055);
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.055);
    -webkit-transition: -webkit-transform .15s ease;
    -moz-transition: -moz-transform .15s ease;
    -o-transition: -o-transform .15s ease;
    -ms-transition: -ms-transform .15s ease;
    transition: transform .15s ease;
  }

  a.boxedThumb:hover {
    -webkit-transform: scale(1.5);
    -moz-transform: scale(1.5);
    -o-transform: scale(1.5);
    -ms-transform: scale(1.5);
    transform: scale(1.5);
    z-index: 5;
  }
```

custom.js

```js
$(document).ready(function () {
    let productImageGroups = []
    $('.img-fluid').each(function () {
      let productImageSource = $(this).attr('src')
      let productImageTag = $(this).attr('tag')
      let productImageTitle = $(this).attr('title')
      if (productImageTitle) {
        productImageTitle = 'title="' + productImageTitle + '" '
      }
      else {
        productImageTitle = ''
      }
      $(this).
          wrap('<a class="boxedThumb ' + productImageTag + '" ' +
              productImageTitle + 'href="' + productImageSource + '"></a>')
      productImageGroups.push('.' + productImageTag)
    })
    jQuery.unique(productImageGroups)
    productImageGroups.forEach(productImageGroupsSet)

    function productImageGroupsSet (value) {
      $(value).simpleLightbox()
    }
  })
```

mkdocs.yml配置

```yml
# 自定义css
extra_css:
- 'stylesheets/custom.css'
- 'stylesheets/simpleLightbox.min.css'
# 自定义js
extra_javascript:
- 'javascripts/jquery-3.4.1.min.js' #注意顺序
- 'javascripts/simpleLightbox.min.js'
- 'javascripts/custom.js'
# 一些扩展
markdown_extensions:
- markdown.extensions.attr_list # 允许在MarkDown链接/图片后用括号指明任意标签的字段
- admonition
- codehilite:
    guess_lang: false
    linenums: false
- toc:
    permalink: true
- footnotes
- meta
- def_list
- pymdownx.arithmatex
- pymdownx.betterem:
    smart_enable: all
- pymdownx.caret
- pymdownx.critic
- pymdownx.details
- pymdownx.inlinehilite
- pymdownx.magiclink
- pymdownx.mark
- pymdownx.smartsymbols
- pymdownx.superfences
- pymdownx.tasklist
- pymdownx.tilde
```