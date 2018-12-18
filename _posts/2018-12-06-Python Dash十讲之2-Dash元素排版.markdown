---
layout:     post
title:      "Python Dash十讲之2 — Dash元素排版"
subtitle:   "————页面元素/图表局部、CSS、Style"
date:       2018-12-06 10:00:00
author:     "LiangFan"
header-img: "img/post-bg-2018-12-06.jpg"
tags:
    - Python
    - Dash
    - 数据可视化
---


> dash中的dcc.Graph 也就是图形元素只能通过图形自带的参数来调整形状、颜色、位置等。其它元素既可以通过dash内置参数来调整，也可以通过外置的CSS文件来控制。

通过内置的style属性来控制元素格式：

```python
html.P(
    children='I am Chinese, I love my motherland.',
    id='my-1st-p-label',
    className='p-label',
    
    # style键值对来控制元素的格式
    style={'font-size': 20, 'text-align': 'center', 'margin-top': 10, 'color': 'red'}
),
```


通过外置css来控制元素格式，首先需要在 `.py` 所在文件夹中新建一个文件夹命名为 `assets` ，也就是说 .py 文件和assets文件夹在同一目录下。然后在assets文件夹下新建一个 `typography.css` 文件，在该css文件中可以对 .py 文件中的元素加以控制，示例如下：

```css
/*style your html*/
#my-1st-p-label {
    font-size: 40px;
    color: rebeccapurple;
    text-align: left;
}

/*you can use  class-# or id-. to style your html*/
/*.p-label {*/
    /*font-size: 40px;*/
    /*color: blue;*/
    /*text-align: center;*/
/*}*/
```


> 当既有内置代码也有外置css文件且两者冲突时，不会出错，内置代码优先级更高，会优先使用。对于较为复杂的且较多元素复用的APP强烈建议通过外置css文件来控制格式，将内容和排版、格式相分离，易于代码管理和维护，python代码可以专注于内容和图形，css代码控制元素的格式和排版。css相关的知识，同样可以去w3cschool深入了解。


元素间的排版，如多个图形水平横排，多个子元素纵向排列等，可以通过div元素的相互嵌套来排版实现，如将不同的元素放入不同的div容器中，通过排版div容器来实现元素的排版。具体可参考dcc.Graph 中的图形排版。


