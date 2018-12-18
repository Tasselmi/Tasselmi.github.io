---
layout:     post
title:      "Python Dash十讲之1 — Python Dash概述"
subtitle:   "————初识Python数据可视化前端框架Dash"
date:       2018-12-05
author:     "LiangFan"
header-img: "img/post-bg-re-vs-ng2.jpg"
tags:
    - Python
    - Dash
    - 数据可视化
---

> Python Dash是html网页元素和图表的python实现，它通过将html元素简化成python对象，使得作者可以通过python代码优雅的编写网页，从而能够更加专注于网页排版及网页内容而不是元素实现本身。非常适合做在线数据可视化和自动化报表。


Python Dash有两个主要构成元素：`Dash Core Components`和`Dash HTML Components`， `Dash Core Components`是核心元素（底层通过React 和 JavaScript实现），它是tab、graphs、slider、dropdown等元素的抽象类和集合，如果想在页面添加一个下拉框，那么实例化一个dcc.Dropdown对象即可；Dash HTML Components是html元素，它是div、p、h1-h6等html元素（各类html元素可自行到[https://www.w3cschool.cn/html/](https://www.w3cschool.cn/html/)探索）的抽象类和集合，如果想在页面添加一个div容器，那么实例化一个html.Div对象即可。一个网页的所有内容都是由这两类核心元素的实例化对象组成的。


一个简短的dash app实现如下：

```python
# pip3 install dash dash_core_components dash_html_components
import dash
import dash_core_components as dcc   #引入dash核心要素
import dash_html_components as html   #引入dash html要素

# 实例化一个dash app对象
app = dash.Dash()

# 编写app的框架和内容layout，layout是由dcc和html组成的列表填充的div
app.layout = html.Div([

    # 实例化一个html p标签
    html.P(
        children='I am Chinese, I love my motherland.',
        id='my-1st-p-label',
        className='p-label'
    ),

    # 实例化一个html 按钮
    html.Button(
        children='点击提交',
        id='my-button'
    ),

    # 实例化一个滑动器
    dcc.Slider(
        min=0,
        max=9,
        marks={i: 'Label {}'.format(i) for i in range(10)},
        value=5,
    )

])

# 运行app
if __name__ == '__main__':
    app.run_server(host='127.0.0.1', port='8050', debug=True)
```

更多 `Dash Core Components` 和`Dash HTML Components` 可自行根据官方文档学习探索。
