---
layout:     post
title:      "Python Dash十讲之6 - 在DASH APP中添加静态元素"
subtitle:   "————添加.html/.png等格式静态文件"
date:       2018-12-08 10:00:00
author:     "LiangFan"
header-img: "img/my-post/04.jpg"
tags:
    - Python
    - Dash
    - 数据可视化
---


> 在dash app中添加静态元素非常简单——将静态元素放在某个目录下并引入dash app中即可。

一般来说，需要在 `.py` 所在文件夹中新建一个文件夹命名为 `assets` ，也就是说`.py` 文件和`assets`文件夹在同一目录下（前面dash元素排版中已经提到过，`typography.css` 文件也在此文件夹下）。当然，也可以遵从自己的个人喜好建立一个文件夹，然后在dash app代码中引入文件的绝对路径即可。


引入html文件或图片文件示例如下：

```python
import dash
import dash_core_components as dcc 
import dash_html_components as html  

app = dash.Dash()
app.title = 'my-dash-app'

app.layout = html.Div([

    html.Iframe(
        id='my-static',
        src='/assets/新增留存.html',
        height=925,
        width='100%',
        draggable=False,
        style={'border': 'none', 'margin-top': 50}
    ),

    html.Div(
        html.Img(
            id='click01_cc',
            src='/assets/creditCard01.png',
            height=600
        ),
        style={
            'margin-left': '50',
        }
    ),

])

if __name__ == '__main__':
    app.run_server(host='127.0.0.1', port='8050', debug=True)
```


- 引入app的icon文件
  - icon是指会出现在浏览器tab前面的图标
  - 直接将icon图片文件命名为 `favicon.ico` 并放置 在`assets` 文件夹下即可，dash app会自动寻找并引用













