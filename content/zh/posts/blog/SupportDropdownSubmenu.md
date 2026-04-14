---
title: 五分钟搭建个人博客：支持多语言选择和下拉菜单
date: 2024-05-07T06:47:40.000Z
draft: false
description: ' 在PaperMod主题中自定义头部导航，添加了多语言选择和下拉菜单功能。通过修改头部文件，实现了多语言的聚合展示和下拉菜单的子菜单导航，同时解决了滚动条和选项高度问题。最终效果可在链接查看。'
---

之前在设置分类、系列入口的时候，发现算是类似的功能，一股脑都放在头部导航，似乎没这么必要，选择展示哪个，也都差不多，而且还有“关于”这类地址。就想着可否有下拉菜单的支持，配置文件倒是支持子菜单的配置信息，只需要增加`parent`字段即可。
```yaml
- identifier: categories
  parent: 更多
  name: 类别
  url: /categories/
  weight: 100
```
可是PaperMod本身不支持下拉菜单配置，不想覆盖主题文件，遂作罢。但不曾想，人工翻译了两篇文章之后，累了🤦🏻‍♀️，就想使用Openai api去机器翻译，既然都机器翻译，何不都支持了，结果在添加更多语言配置的时候。发现header布局是这样的，横向滚动排列，需要占用头部非常多的空间。

![default i18N header 2024-05-07 21.26.11.png](https://s2.loli.net/2024/05/07/CcubVagk9GMzBT5.png)

最终还是得自己处理PaperMod头部文件，添加语言选择和Dropdown下拉菜单。

## 修改头部导航样式
Hugo 在处理layout的时候，会优先自定义的，再到主题，如果需要修改头部导航，只能整个部分全部覆盖。咱们搭建博客的时候，对主题文件使用的是git module的方式引入（正好今天也更新了次PaperMod主题），为了保持风格统一，直接复制[hugo papermod源码](https://github.com/adityatelange/hugo-PaperMod/blob/master/layouts/partials/header.html)，在此基础上修改。

下面分享这个过程，是怎么做的，<a href="https://github.com/hawkeye-xb/blog/blob/main/layouts/partials/header.html" style="color:blue;">最终代码</a>，<a href="https://hawkeye-xb.xyz/" style="color:blue;">最终效果</a>如下图：
![the final header 2024-05-07 21.32.50.png](https://s2.loli.net/2024/05/07/AoKSfviGaWzgdM4.png)

### 添加多语言选择
如果不清楚到底使用的什么数据，可以在[Hugo docs](https://gohugo.io/methods/site/languages/)先行查看。
直接搜索Language 得到下面相关代码，HTML和hugo语法的混合体。
```html
<ul class="lang-switch">
  {{- if $separator }}<li>|</li>{{ end }}
  {{- range . -}}
  {{- if ne $lang .Lang }}
  <li>
      <a href="{{- .Permalink -}}" title="{{ .Language.Params.languageAltTitle | default (.Language.LanguageName | emojify) | default (.Lang | title) }}"
          aria-label="{{ .Language.LanguageName | default (.Lang | title) }}">
          {{- if (and site.Params.displayFullLangName (.Language.LanguageName)) }}
          {{- .Language.LanguageName | emojify -}}
          {{- else }}
          {{- .Lang | title -}}
          {{- end -}}
      </a>
  </li>
  {{- end -}}
  {{- end}}
</ul>
```
大致意思也都能猜出来，循环`.`这个数据的信息，判断Lang不是当前选择的，则以某种信息先后顺序展示。展示的具体规则就没必要过多研究，保持统一就好。

我们将多语言的选择聚合起来，在语言切换入口展示的内容就需要改变，如果展示【语言切换↓】，咱们得给每个语言兼容这个“短语”（i18N目录正好发挥作用😁），就是略麻烦。所以我选择展示当前语种的缩写，比如：zh、en。

这里添加了默认选中的内容。
```html
<option
    selected
    title="{{ $.Site.Language.LanguageName | default ($.Site.Language.Lang) }}"
>{{ $.Site.Language.LanguageName | default ($.Site.Language.Lang) }}</option>
```
然后将a标签的参数直接转换成为option的，就完成多语言选择的基础功能。
```html
<select class="custom-select" id="language-select" onchange="window.location.href=this.value;">
    <option
        selected
        title="{{ $.Site.Language.LanguageName | default ($.Site.Language.Lang) }}"
    >{{ $.Site.Language.LanguageName | default ($.Site.Language.Lang) }}</option>
    {{- range . -}}
    {{- if ne $lang .Lang }}
    <option
        value="{{- .Permalink -}}"
        title="{{ .Language.Params.languageAltTitle | default (.Language.LanguageName | emojify) | default (.Lang | title) }}"
    >
            {{- if (and site.Params.displayFullLangName (.Language.LanguageName)) }}
            {{- .Language.LanguageName | emojify -}}
            {{- else }}
            {{- .Lang | title -}}
            {{- end -}}
    </option>
    {{- end -}}
    {{- end}}
</select>
```
#### CSS
再添加上支持亮暗色的样式属性。在devtool中，找到:root定义的变量，或者[hugo css变量定义文件](https://github.com/adityatelange/hugo-PaperMod/blob/master/assets/css/core/theme-vars.css)查看使用（并不是很齐全，比如hover、active就不知道该选什么变量，或许后续我可以写个脚本，像Flutter一样一键生成、导出Material3规范的颜色，方便在Web管理颜色主题。 ）。这里尴尬的是bg颜色使用，目前是：
  - 亮色、首页使用的是--code-bg变量
  - 而暗色、首页用了--theme
  - 亮色、文章页使用的是--theme
  - 暗色、文章页用了--theme
```css
.custom-select {
  color: var(--content);
  outline: none;
  border: none;
  box-shadow: none;
  background-color: var(--theme);
}
.list .custom-select {
  background-color: var(--code-bg);
}

.dark .custom-select{
  background-color: var(--theme);
}

.custom-select:focus {
  outline: none;
  /* border-color: #a9a9a9; */
}

.custom-select::-ms-expand {
  display: none; /* 针对 IE/Edge */
}
```


### 增加下拉菜单
多语言设置完成之后，接下来给头部增加下拉菜单。同样的([hugo docs](https://gohugo.io/methods/menu-entry/haschildren/))，先找到菜单数据代码，如下：

```html
<ul id="menu">
    {{- range site.Menus.main }}
    {{- $menu_item_url := (cond (strings.HasSuffix .URL "/") .URL (printf "%s/" .URL) ) | absLangURL }}
    {{- $page_url:= $currentPage.Permalink | absLangURL }}
    {{- $is_search := eq (site.GetPage .KeyName).Layout `search` }}
    <li>
        <a href="{{ .URL | absLangURL }}" title="{{ .Title | default .Name }} {{- cond $is_search (" (Alt + /)" | safeHTMLAttr) ("" | safeHTMLAttr ) }}"
        {{- cond $is_search (" accesskey=/" | safeHTMLAttr) ("" | safeHTMLAttr ) }}>
            <span {{- if eq $menu_item_url $page_url }} class="active" {{- end }}>
                {{- .Pre }}
                {{- .Name -}}
                {{ .Post -}}
            </span>
            {{- if (findRE "://" .URL) }}&nbsp;
            <svg fill="none" shape-rendering="geometricPrecision" stroke="currentColor" stroke-linecap="round"
                stroke-linejoin="round" stroke-width="2.5" viewBox="0 0 24 24" height="12" width="12">
                <path d="M18 13v6a2 2 0 01-2 2H5a2 2 0 01-2-2V8a2 2 0 012-2h6"></path>
                <path d="M15 3h6v6"></path>
                <path d="M10 14L21 3"></path>
            </svg>
            {{- end }}
        </a>
    </li>
    {{- end }}
</ul>
```
代码循环的是`site.Menus.main`数据，对search和其它做了单独处理。

和多语言选项刚好相反，子菜单是路由导航类型，没有当前状态，父菜单分类名称不会动态改变，所以select和option就不合适，仅下拉点击就满足。

纠结了下是否引入一些UI库，感觉没几行代码，不值得增加依赖文件，影响打开速度，就自己动手了。

增加子元素的判断，如果有子元素，则渲染下拉菜单，否则照旧。
```html
{{- if .HasChildren }}
<!-- dropdown code -->
{{- else }}
<!-- original code -->
{{- end }}
```
添加的代码如下，注释的a标签代码直接复制过来。
```html
<ul id="menu" style="overflow: visible;">
    <!-- other code -->
    <!-- {{- $is_search := eq (site.GetPage .KeyName).Layout `search` }} -->
    <li>
        {{- if .HasChildren }}
        <ul class="dropdown">
            <span style="font-size: 16px;">{{ .Title | default .Name }} {{- cond $is_search (" (Alt + /)" | safeHTMLAttr) ("" | safeHTMLAttr ) }}</span>
            <div class="dropdown-content">
                {{- range .Children }}
                <div class="dropdown-item">
                    <!-- a tag code -->
                </div>
                {{- end }}
            </div>
        </ul>
        {{- else }}
        <!-- a tag code -->
        {{- end }}
    </li>
</ul>
```
再自信的增加自定义下拉框CSS。

#### 滚动条
如没意外，肯定就有意外。下拉框展开的时候，菜单列表部分出现了滚！动！条！

<!-- 【表情包：jelly躺倒】 -->

通过devtool选中滚动条，发现是ul标签出现的滚动条，是因为设置了`overflow-x: auto`属性，目前还不清楚出于什么考虑，我认为菜单一级展示部分，只应该留给重要的导航，也不应该出现滚动条，所以直接`overflow: visible` /doge。

#### line-height
处理完滚动条，又发现下拉菜单的选项高度，无论怎么都无法调小，想到在为了语义化重复使用了li和a标签，必是有公共样式影响了。果不其然，看到了将`line-height: var(--header-height)`的代码。

### 最后
最后总结下今天的五分钟，使用了select、option给PaperMod主题语言列表替换为选择的方式，遇到了颜色变量的问题；然后自定义了下拉选框(dropdown)，简单的支持子菜单导航，过程遇到了滚动条和选项高度问题。

<a href="https://github.com/hawkeye-xb/blog/blob/main/layouts/partials/header.html" style="color:blue;">最终代码</a>，<a href="https://hawkeye-xb.xyz/" style="color:blue;">最终效果</a>

今天这五分钟，希望对您有帮助/doge。

