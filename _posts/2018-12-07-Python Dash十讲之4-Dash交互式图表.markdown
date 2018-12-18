---
layout:     post
title:      "Python Dash十讲之4 — Dash交互式图表"
subtitle:   "————回调函数、Input、Output"
date:       2018-12-07 10:00:00
author:     "LiangFan"
header-img: "img/my-post/02.jpg"
tags:
    - Python
    - Dash
    - 数据可视化
---


> 所谓交互式是指，用户对APP施加行为能够影响或改变APP的外观或内容，行为能够得到反馈。比如，鼠标hover时候数据闪现、点击按钮生成新的数据和图表等，都属于交互。

dash中实现交互的逻辑非常简单，即调用回调函数`@app.callback`，在回调函数中 `Input` 为被施加行为的元素， `Output` 为被影响的元素，也就是说Input是因，Output是果，对Input传入的值一番操作后，将值（回调函数返回值）传给Output进行输出。同一个元素只能在Output中出现一次，也就是说同一个元素不能被多个回调函数的Output调用，但一个Input可以调用多个回调函数——控制多个其它元素。如果多个元素同时控制一个元素，那么需要在一个回调函数中，输入多个Input。

State可以记录元素的当前状态或值。比如记录当前图表被鼠标hover时候的值。State只能配合Input一起使用——输入行为的同时记录元素的状态值。

Input使用方法为  `[Input('id', 'property')]` ；Output的使用方法为 `Output('id', 'property')` ；State的使用方法为 `[State('id', 'property'), State('id', 'property')]`

代码示例如下：

```python
import os
from datetime import datetime
import dash
import dash_core_components as dcc   
import dash_html_components as html   
from dash.dependencies import Output, Input, State

app = dash.Dash()
app.layout = html.Div([
    html.Div([
        html.H6('请给我们反馈',
            style={'color': 'red', 'font-size': 30, 'padding-left': 15, 'padding-top': 30, }),
        html.P('1.对于本月的月报，您的满意度评分是？（0-10分，分值越高越满意）'),
        html.Div(
            dcc.Slider(
                id='slider_feedback',
                min=0,
                max=10,
                marks={i: '{}分'.format(i) for i in range(11)},
                value=0
            ),
            style={'padding-left': 15, 'width': '60%'}
        ),
        html.P('2.您觉得月报还有哪些地方需要改进？或者您在下一期月报中还想了解哪些内容？', style={'margin-top': 50}),
        html.Div(
            dcc.Textarea(
                id='textarea_feedback',
                placeholder='不记名、不追踪、不打击报复，请畅所欲言~~~',
                style={'width': '100%', 'height': 80}
            ),
            style={'padding-left': 15, 'width': '60%'}
        ),
        html.Div(
            html.Button('点击提交', id='button_feedback', n_clicks=0),
            style={'padding-left': 15, 'padding-top': 15,},
        ),
        html.Div(
            id='div_feedback_success',
            children=html.P(
                id='p_feedback_success',
                children=None,
                style={'color': 'red', 'font-size': 12, 'padding-left': 15, 'padding-top': 10,},
            ),
            style={},
        ),
        html.Div(
            id='div_feedback_data',
        ),
    ], className='div_feedback'),
])

# 点击按钮返回提交成功
@app.callback(Output('p_feedback_success', 'children'), [Input('button_feedback', 'n_clicks')])
def feedback_success(n_clicks):
    hint = '提交成功，谢谢！'
    if n_clicks > 0:
        return hint

# 点击提交按钮保存数据
@app.callback(Output('div_feedback_data', 'children'), [Input('button_feedback', 'n_clicks')],
              [State('slider_feedback', 'value'), State('textarea_feedback', 'value')])
def feedback_success(n_clicks, score, text):
    path = '/Users/liang/Documents/Python/99Pyproject/channelDailyData'
    if n_clicks > 0:
        with open(os.path.join(path, 'feedback.txt'), 'a', encoding='gbk') as f:
            f.write('"{}", "{}", "{}", "{}" \n'.format(datetime.now(), n_clicks, score, text.strip('\n')))
        print('有人在{}提交了第{}条反馈意见，请快查看！ '.format(datetime.now(), n_clicks))

if __name__ == '__main__':
    app.run_server(host='127.0.0.1', port='8050', debug=True)
```








