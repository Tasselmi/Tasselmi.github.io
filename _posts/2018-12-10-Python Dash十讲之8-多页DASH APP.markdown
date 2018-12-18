---
layout:     post
title:      "Python Dash十讲之8 - 多页DASH APP"
subtitle:   "————页面间切换跳转、工程结构、代码示例"
date:       2018-12-10 10:00:00
author:     "LiangFan"
header-img: "img/my-post/06.jpg"
tags:
    - Python
    - Dash
    - 数据可视化
---

> 上面提到的APP都是单一页面的，即不存在页面之间的跳转，下面看一下多个页面的APP是如何实现的。

- 多页面APP的实现原理
    - dash app的页面间切换，其实可以理解为文件路径的切换
    - 一个` .py `文件为一个页面，每个` .py `文件有自己独立的内容
    - 通过回调函数，将文件的 url 地址作为输入，将文件的内容（layout）作为输出，即可实现页面跳转

- 多页面APP的文件结构
    - `app.py` 文件内代码放置app的实例，以及和app相关的属性配置
    - `index.py` 文件内代码放置索引页的内容，以及分页面文件的路径地址及跳转逻辑，**`索引页是APP的入口`**。
    - `apps` 文件夹放置各分页面的.py文件，一个页面一个.py文件
    - `assets` 文件夹放置`typography.css`文件——app格式控制文件、`favicon.ico`文件——app图标、以及其它需要引用的静态文件


多页面APP文件结构示意如下：

```
- app.py
- index.py
- apps
    |-- __init__.py
    |-- app1.py
    |-- app2.py
- assets
    |-- typography.css
    |-- favicon.ico
    |-- static1.html
    |-- static2.png
```



`app.py`代码示例如下：

```python
import dash

app = dash.Dash()
server = app.server
app.config.suppress_callback_exceptions = True
app.title = 'my-dash-multi-page-app'
```



`index.py`代码示例如下：

```python
import dash_core_components as dcc
import dash_html_components as html
from dash.dependencies import Input, Output, State
from app import app
from apps import app1, app2


app.layout = html.Div([
    dcc.Location(id='url', refresh=False),
    html.Div(id='page-content'),
])

#目录页
index_page = html.Div([

    html.H2(children='my-app-is-running', id='title'),

    html.Ul([
        html.Li(html.A('page-1', href='app1')),
        html.Li(html.A('page-2', href='app2')),
    ]),

])


# Update the index
@app.callback(Output('page-content', 'children'), [Input('url', 'pathname')])
def display_page(pathname):
    if pathname == '/':
        return index_page
    elif pathname == '/app1':
        return app1.layout
    elif pathname == '/app2':
        return app2.layout
    else:
        return 'URL not found'


if __name__ == '__main__':
    app.run_server(debug=True)
```



 `apps-app1.py`代码示例如下：

```python
import dash
import dash_table
import dash_html_components as html
import dash_core_components as dcc
from dash.dependencies import Output, Input, State
import pandas as pd
import ssl
ssl._create_default_https_context = ssl._create_unverified_context
from app import app

df = pd.read_csv('https://raw.githubusercontent.com/plotly/datasets/master/gapminder2007.csv')

layout = html.Div([

    html.P(
        df[:15].to_json()
    ),

    dash_table.DataTable(
        id='data-table',
        columns=[{'id': c, 'name': c} for c in df.columns],
        data=df.to_dict('rows'),
        style_as_list_view=True,
        style_header={
            'backgroundColor': 'blue',
            'fontWeight': 'bold',
            'textAlign': 'center',
            'color': 'white'
        },
        style_cell={'padding': '5px'},
        style_cell_conditional=
        [
            {'if': {'column_id': c}, 'textAlign': 'left'} for c in df.columns
        ]
        +
        [
            {'if': {'column_id': 'gdpPercap'}, 'width': '30%'},
            {'if': {'column_id': 'country'}, 'width': '30%'},
        ],
        style_data_conditional=[
            {"if": {"row_index": 4}, "backgroundColor": "#3D9970", 'color': 'white'},
            {'if': {'column_id': 'country'}, 'backgroundColor': '#3D9970', 'color': 'white'},
        ],
        pagination_settings={
            'current_page': 0,
            'page_size': 15 #每页行数
        },
        pagination_mode='be',
        sorting='be',
        sorting_type='single',
        sorting_settings=[],
        editable=True
    ),

], className='div-sub-page')


@app.callback(
    Output('data-table', 'data'),
    [Input('data-table', 'pagination_settings'),
     Input('data-table', 'sorting_settings')])
def update_graph(pagination_settings, sorting_settings):
    if len(sorting_settings):
        dff = df.sort_values(
            sorting_settings[0]['column_id'],
            ascending=sorting_settings[0]['direction'] == 'asc',
            inplace=False
        )
    else:
        # No sort is applied
        dff = df

    return dff.iloc[
        pagination_settings['current_page']*pagination_settings['page_size']:
        (pagination_settings['current_page'] + 1)*pagination_settings['page_size']
    ].to_dict('rows')
```



`apps-app2.py`代码示例如下：

```python
import pandas as pd
import dash_core_components as dcc   #引入dash核心元素
import dash_html_components as html   #引入dash html元素
from app import app

df = pd.DataFrame([
    {'city': 'hangzhou', 'man': 150, 'woman': 0.523, 'city-level': '2nd'},
    {'city': 'shanghai', 'man': 200, 'woman': 0.26, 'city-level': '1st'},
    {'city': 'shenzhen', 'man': 150, 'woman': 0.37, 'city-level': '1st'},
    {'city': 'wuhan', 'man': 160, 'woman': 0.17, 'city-level': '2nd'},
    {'city': 'ningbo', 'man': 100, 'woman': 0.88, 'city-level': '3rd'},
])

layout = html.Div([

    dcc.Graph(
        id='graph-bar-horizontal2',
        className='my_graph',
        figure={
            'data': [{
                'x': df['man'],
                'y': df['city'],
                'type': 'bar',
                'orientation': 'h'
            }],
            'layout': {
                'title': '各城市男性人口',
                'height': 600,
                'yaxis': {'hoverformat': '.0f'},
                'margin': {'l': 80, 'r': 35, 't': 50, 'b': 80},
            }
        },
        config={
            'displayModeBar': False
        },
    ),

])
```



`index.py`作为app的入口，运行其中的 `app.run_server` 即可。

&nbsp;
&nbsp;















