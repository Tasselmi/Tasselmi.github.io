---
layout:     post
title:      "Python Dash十讲之7 - 实时刷新页面元素"
subtitle:   "————控制页面元素刷新频率"
date:       2018-12-09 10:00:00
author:     "LiangFan"
header-img: "img/my-post/05.jpg"
tags:
    - Python
    - Dash
    - 数据可视化
---


> dash app的实时刷新功能主要依赖于 `dash_core_components.Interval`， 将`Interval`元素作为回调函数的输入，需要实时刷新的元素的相关属性值作为输出，通过监听`Interval`元素的 `n_intervals` 属性从而实现APP的动态自动更新。`Interval`元素的`interval`属性可以控制刷新频率，该属性单位为毫秒，取整数值。例如需要APP每分钟刷新一次，可以设置` interval = 60*1000`。

下面为一个每一分钟读取远程数据并提供数据下载功能的APP代码示例：

```python
import dash
import pandas as pd
import dash_core_components as dcc
import dash_html_components as html
from dash.dependencies import Output, Input, State
import urllib
import ssl
ssl._create_default_https_context = ssl._create_unverified_context

app = dash.Dash()
app.title = 'my-dash-app'

app.layout = html.Div([

    dcc.Graph(
        id='my-table',
        config={'displayModeBar': False}
    ),

    html.Div(
       html.A(
           '下载原始数据',
           id='download-link',
           download="raw-data.csv",
           href="",
           target="_blank",
           n_clicks=0
       ),
       style={"margin-left": 80, 'margin-top': 20}
    ),
    
    dcc.Interval(
        id='interval-component',
        interval=60*1000,   # in milliseconds / refresh the webpage in every 60 seconds
        n_intervals=0
    ),

])

@app.callback(Output('my-table', 'figure'), [Input('interval-component', 'n_intervals')])
def update_table(n):
    df = pd.read_csv('https://raw.githubusercontent.com/plotly/datasets/master/gapminder2007.csv')[:15]
    figure = {
        'data': [{
            'type': 'table',
            'columnwidth': 0.3,
            'header': {
                'values': [['<b>{}</b>'.format(i)] for i in df.columns],
                'font': {'size': 13, 'color': 'white', },
                'align': 'center',
                'height': 30,
                'fill': {'color': '#0076BA'}
            },
            'cells': {
                'values': df.values.T,
                'line': {'color': 'rgb(50, 50, 50)'},
                'align': 'center',
                'height': 27,
                'fill': {'color': ['#56C1FF', '#f5f5fa']},
            },
        }],
        'layout': {
            'height':len(df)*30+50,
            'margin': {'l': 80, 'r': 80, 't': 20, 'b': 20},
        }
    }
    return figure

@app.callback(Output('download-link', 'href'), [Input('interval-component', 'n_intervals')])
def update_download_link(n):
    df = pd.read_csv('https://raw.githubusercontent.com/plotly/datasets/master/gapminder2007.csv')
    csv_string = df.to_csv(index=False, encoding='utf-8')
    csv_string = "data:text/csv;charset=utf-8," + urllib.parse.quote(csv_string)
    return csv_string

if __name__ == '__main__':
    app.run_server(host='127.0.0.1', port='8050', debug=True)
```















