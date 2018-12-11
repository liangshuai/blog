title: CSS Modules 简单入门
date: 2016-05-26 16:37:44
tags:
- CSS
- Web组件
- CSS Modules
---

CSS的选择器在全局范围都会生效，比如有时候会有A写了 `btn` 的class定义了一个默认显示蓝色按钮， B之后不小心也写了一个 `btn` 的class， 很不幸，A需要的按钮样式可能被B所写的样式覆盖！ Bootstrap中的类似btn这种情况，在 `btn` 的class 通常定义的是所有按钮通用的样式，并且通过在定义 `btn-success`、`btn-error` 等样式通过选择器优先级来实现不同的样式。但是这种依旧相当于在全局中定义了样式， 无法避免将来有人不经意的又写了`btn-success`这样的class覆盖或者影响了之前的样式。

为了解决这问题，前端大牛们提出了种种的命名约定来避免命名冲突。 比如[OOCSS](http://oocss.org/)、[SMACSS](https://smacss.com/)、[BEM](https://en.bem.info/methodology/naming-convention/)、[SUIT](https://suitcss.github.io/)等等。 以BEM为例，class的名字以`block__element--modifier`这种方式来命名，比如写个Menu， 那么Menu中的item处于active状态就要命名成 `menu__item--active` 这样的写法可能会解决问题，但是写起来总感觉有些反人类。所以有人说[BEM很火,看着就想吐](http://weibo.com/1960954893/Ac2LAgIjn?type=comment) . 并没有在项目中实践BEM这种命名方式，但是它来解决命名冲突的思路还是挺不错的，相当于通过模块命名前缀的方式避开在全局空间下命名，每个组件在它所属的命名空间下定义。

<!-- more -->

直到2014年Christopher Chedeau 在NationJS中所讲的[CSS in JS](http://blog.vjeux.com/2014/javascript/react-css-in-js-nationjs.html), 前端衍生出两个流派，一个主张彻底抛弃CSS，在JS中来写样式，并且从中衍生出[React Style](https://github.com/js-next/react-style)、 [jsxstyle](https://github.com/petehunt/jsxstyle)、 [Radium](https://github.com/FormidableLabs/radium)等实现。

CSS in JS 好处虽多，但是也有一些不便之处，比如无法使用css预处理器、调试困难、无法使用像PostCSS这样的CSS常用工具等。

在此基础上[CSS Modules Team](https://github.com/orgs/css-modules/people) 提出的CSS Modules显的更加实用，我们还可以独立的写CSS文件，通过JS来管理样式依赖。

最近使用Mithril.js写了个[小项目](https://github.com/liangshuai/git-history) 并且尝试实践CSS Modules ，最终的效果如下:

文件结构

```
+---components
|   +---LeftBar
|   |       LeftBar.css
|   |       LeftBar.js
|   +---Nav
|   |       Nav.css
|   |       Nav.js
|   +---...
```

像写普通CSS一样写CSS文件，以Nav.css为例(CSS Modules 推荐驼峰式命名)

```css
.list {
    margin: 0;
    padding: 3px 10px;
    list-style: none;
    line-height: 1.5;
    color: #9c9c9c;
}

.item {
    margin: 0;
    display: inline-block;
    position: relative;
    margin-left: 20px;
}
```

在JavaScript中引用css文件，并且定义元素的class。这里使用了[Webpack](https://webpack.github.io/)，[Browersify](http://browserify.org/)也可以实现。

```javascript
import m from 'mithril';
import styles from './Nav.css';

var Nav = module.exports = {
    controller: function() {
    },

    view: function(ctrl) {
        return m("nav", {class: styles.nav}, [
            m("ul", {class: styles.list}, [
                m('li', {class: styles.item}, [
                    m('i.fa.fa-folder-o'),
                    'Git History'
                ]),
                m('li', {class: styles.item}, [
                    m('i.fa.fa-file-text-o'),
                    'index.js'
                ])
            ])
        ]);
    }
};

```

最终生成的效果如下：


![CSS Module Screenshot](https://ooo.0o0.ooo/2017/03/04/58ba0a05b9c87.png)


其中生成的Class Name依赖于具体的配置规则，示例中的WebPack Loader配置如下：

```js
loaders: [
  { test: /\.js$/, loader: 'babel-loader', exclude: /node_modules/ },
  { test: /\.css$/, loader: ExtractTextPlugin.extract('style-loader', 'css-loader?modules&importLoaders=1&localIdentName=[name]__[local]___[hash:base64:5]!postcss-loader') }
]
```

CSS文件里面的选择器通过在JS中import之后引用，Webpack会自动处理，这样我们就可以既避免BEM那种反人类的写法，又享受到模块化命名。

### 命名方式

CSS Modules推荐驼峰式命名，但是并不是强制的。需要注意的是在写样式的时候不推荐像Bootstrap那样通过选择器优先级来覆盖，一个Class应该包含该Class所有的样式，这样所有的组件、元素都是相对独立的。以btn的不同状态为例

```css
/* BootStrap是这样实现 */
.btn {}
.btn-default {} /* 在`btn`的基础上定义`default`样式，HTML中需要同时引用 `btn` 和 `btn-default` */
.btn-success {} /* 在`btn`的基础上定义`success`样式，HTML中需要同时引用 `btn` 和 `btn-success` */

/* CSS Modules 推荐方式 */

.default {} /* 包含所有default的样式, html中仅仅引用`default` 的就可以，此处的命名也不用加btn前缀 */
.success {} /* 包含所有success的样式, html中仅仅引用`success` 的就可以 */

```

CSS Modules组件里面Class命名不需要再加模块名的前缀，一般Webpack/Browserify配置好之后就能自动生成出来，类似BEM那样，模块名通常都以文件名来命名了。

### Composition

上面说到一个Class应该包含它自身所需要的所有样式，但是并不意味着需要重复大量的代码， 用过Sass的应该比较熟悉@extend，在CSS Modules中也有类似的，叫Composition。

```css
.common {} /* 定义公用样式 */
.default{
    composes: common;
    ...
}
.success{
    composes: common;
    ...
}

```
如果使用了composes的话，必须在其它样式之前声明，composes也可以声明多个，使用 `composes: commonA commonB`


### Global 范围的样式

有时候需要写一些全局范围的样式，比如经常使用的clearfix， CSS Modules提供了:global

```css
:global(.clearfix::after) {
    content: '';
    clear: both;
    display: table;
}
//或者这种
:global .clearfix {
    
}
.menu:global(.active) {
}

```

### 依赖管理

如果需要引入外部文件的样式也可以使用composes

```css
.default{
    composes: common from '../common.css';
}

```
### 定义变量

如果使用了postcss和[postcss-modules-values](https://github.com/css-modules/postcss-modules-values) 就可以像Sass/Less一样使用变量

比如

*fontsize.css*
```css
@value large: 2rem;
@value middle: 1.5rem;
@value small: 1rem;

```

*Nav.css*
```css
@value fontsize: './fontsize.css';
@value large, middle, small from fontsize;

.list {
    font-size: large;
}
```

### 总结

CSS Modules是目前实现CSS组件化最可行的方案之一，相对于其它实现，学习成本也会小很多，同时也能充分结合CSS生态工具(Sass/Less/Postcss等)和JS模块化能力。


