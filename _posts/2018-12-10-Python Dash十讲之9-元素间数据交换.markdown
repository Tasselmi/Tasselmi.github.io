---
layout:     post
title:      "Python Dash十讲之9 - 元素间数据交换"
subtitle:   "————用hidden div来存储中间数据"
date:       2018-12-10 11:00:00
author:     "LiangFan"
header-img: "img/my-post/07.jpg"
tags:
    - Python
    - Dash
    - 数据可视化
---

设想如下1个需求：
- 从数据库读取数据，数据量较大；然后将数据以Excel表格形式呈现在网页上
- 数据表格可下载原始数据
- 将其中部分数据或者全部数据以可视化图形方式呈现在网页上，需要呈现多个图形
- 要求所有图形和表格按照一定频率定时自动刷新，下载的原始数据也必须是最新的数据

> 需求分析：因为需要定时刷新，所以需要用到上面讲的监听 `n_intervals` 来实现（如果不监听`Interval`元素的 `n_intervals` 属性的话，那么读取数据并存为变量，只会在app启动的时候io一次，后面这个变量一直存储在内存中不会改变），每一个需要自动刷新的图形都要监听 `n_intervals` ；但是如果每个回调函数的input都是n_intervals的话，那么每个函数中都需要从数据库中读取数据，而这个数据量又非常大，如果每个函数都重复IO就非常耗时。因此需要转变解决问题的思路。

1个可能的解决方案如下：
- 用一个div元素来存储中间数据，这个div的数据实现实时刷新，通过监听n_intervals来实现
- 这个div元素设置为在前端不显示，只作为存储中间数据用
- 这个div元素中的数据以json形式存储
- 其它需要用到数据的元素从div元素中读取json，可以读取一部分或者全部读取，看使用需求情况
- div中的中间数据是基于用户当前浏览器会话的，不同用户间不共享，也就说每个用户打开页面都要IO一次数据，打开新的浏览器窗口也会重新IO一次数据
- 该方案减少的是同一次会话中的重复IO时间，但是会增加不同回调函数间数据交换的时间，回调函数间的数据交换是通过网络传输实现的，因此会增加网络消耗


代码示例如下：

```python
# -*- coding: utf-8 -*-
import dash
import dash_core_components as dcc
import dash_html_components as html
from datetime import datetime
import numpy as np
import pandas as pd
import pymysql
import urllib
from dash.dependencies import Output, Input, State, Event
import dash_table
import ssl
ssl._create_default_https_context = ssl._create_unverified_context

def select_mysql_data(db_config, sql_sentence):
    '''
    # 连接mysql数据库查询信息
    :param db_config:
    :param sql_sentence:
    :return:
    '''
    # 连接数据库
    connection = pymysql.connect(
        host=db_config['host'],
        user=db_config['user'],
        password=db_config['password'],
        port=3306,
        charset='utf8'
    )
    cursor = connection.cursor()
    cursor.execute(sql_sentence)
    data = cursor.fetchall()
    col_name = np.array(cursor.description)[:, 0]
    connection.close()
    if len(data) > 0:
        df = pd.DataFrame(np.array(data), columns=col_name)
    else:
        print('sql sentence matches no values.')
        df = None
    return df

reports_mysql_config = {
    'host': '127.0.0.1',
    'user': 'your-user',
    'password': 'your-pwd',
    'port': '3306',
    'database': 'db-name',
    'table': 'tb-name',
}
sent = "SELECT * FROM {}.{}".format(reports_mysql_config['database'] ,reports_mysql_config['table'])

app = dash.Dash(__name__)

app.layout = html.Div([
    html.Div(
        id='div-store-data',
        style={'display': 'none'}
    ),
    dash_table.DataTable(
        id='data-table',
        style_as_list_view=True,
        style_header={
            'backgroundColor': '#3D9970',
            'fontWeight': 'bold',
            'textAlign': 'center',
            'color': 'white',
            'padding': 10
        },
        style_cell={'textAlign': 'center'},
        pagination_settings={
            'current_page': 0,
            'page_size': 15  # 每页行数
        },
        pagination_mode='be',
        sorting='be',
        sorting_type='single',
        sorting_settings=[],
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
       style={'margin-top': 30}
    ),
    dcc.Interval(
        id='interval-component',
        interval=60*60*1000,   # in milliseconds / refresh the web-page in every 60 minutes
        n_intervals=0
    ),
    html.Div(
        id='listen-download-link',
        style={'display': 'none'}
    )
], style={'margin-left': 50, 'margin-right': 50, 'margin-top': 20})

# 监听n_intervals，实现数据自动刷新，并将新数据存入一个hidden的div中
@app.callback(Output('div-store-data', 'children'), [Input('interval-component', 'n_intervals')])
def update_and_store_data(n):
    channelDailyData = select_mysql_data(reports_mysql_config, sent).sort_values(by=['日期'], ascending=False)
    channelDailyData['日期'] = channelDailyData['日期'].astype(str)
    data = channelDailyData.to_json(orient='split')
    return data

@app.callback(Output('download-link', 'href'), [Input('div-store-data', 'children')])
def update_download_link(data):
    df = pd.read_json(data, orient='split')
    csv_string = df.to_csv(index=False, encoding='utf-8')
    csv_string = "data:text/csv;charset=utf-8," + urllib.parse.quote(csv_string)
    return csv_string

@app.callback(Output('data-table', 'columns'), [Input('div-store-data', 'children')])
def update_table(data):
    df = pd.read_json(data, orient='split')
    return [{'id': c, 'name': c} for c in df.columns]

@app.callback(
    Output('data-table', 'data'),
    [Input('data-table', 'pagination_settings'), Input('data-table', 'sorting_settings'),
     Input('div-store-data', 'children')]
)
def update_graph(pagination_settings, sorting_settings, data):
    df = pd.read_json(data, orient='split')
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

# 监听是否有人下载了数据
@app.callback(Output('listen-download-link', 'children'), [Input('download-link', 'n_clicks')])
def listen_download_link(n):
    if n > 0:
        print('\n{}  someone download data in {} times.'.format(datetime.now(), n))
    return n

if __name__ == '__main__':
    app.run_server(host="127.0.0.1", port=8050, debug=True)
```

&nbsp;
&nbsp;















