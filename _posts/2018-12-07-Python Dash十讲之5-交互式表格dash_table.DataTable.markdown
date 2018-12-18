---
layout:     post
title:      "Python Dash十讲之5 - 交互式表格dash_table.DataTable"
subtitle:   "————排序、筛选、单元格控制"
date:       2018-12-07 12:00:00
author:     "LiangFan"
header-img: "img/my-post/03.jpg"
tags:
    - Python
    - Dash
    - 数据可视化
---

> `DataTable`是dash中近来发布的独立的表格元素，功能极其强悍。上面提到的`dcc.Graph`中的table可以理解为是一种图形，而`DataTable`是纯天然的html表格，由`tr`、`td`组成，可以对表格的单元格、元素等实现高度控制，因而能够实现更为复杂的表格格式和交互式体验。


`DataTable`可以实现的功能如下：

- 表头/表标题格式控制/多行表头
- 单元格格式控制：对齐、颜色、背景色、按照条件格式着色、缩进、按行控制格式、按列控制格式等
- 可编辑表格
- 数据排序
- 数据筛选
- 数据删除
- 表格分页



代码示例如下：

```python
# pip install dash-table==3.1.6
import dash
import dash_table
import dash_html_components as html
import dash_core_components as dcc
from dash.dependencies import Output, Input, State
import pandas as pd
import ssl
ssl._create_default_https_context = ssl._create_unverified_context

df = pd.read_csv('https://raw.githubusercontent.com/plotly/datasets/master/gapminder2007.csv')

app = dash.Dash(__name__)

app.layout =html.Div([

    dash_table.DataTable(
        id='datatable-interactivity',
        # table-head
        columns=[{'id': c, 'name': c} for c in df.columns],
        # table-row
        data=df.to_dict('rows'),
        # table-grid
        style_as_list_view=True,
        # common header-style
        style_header={
            'backgroundColor': 'blue',
            'fontWeight': 'bold',
            'textAlign': 'center',
            'color': 'white'
        },
        # common cell-style
        style_cell={'padding': '5px'},
        # special cell-style
        style_cell_conditional=
        [
            {'if': {'column_id': c}, 'textAlign': 'left'} for c in df.columns
        ]
        +
        [
            {'if': {'column_id': 'State'}, 'width': '30%'},
            {'if': {'column_id': 'Number of Solar Plants'}, 'width': '30%'},
        ],
        # special row/column style
        style_data_conditional=[
            {"if": {"row_index": 4}, "backgroundColor": "#3D9970", 'color': 'white'},
            {'if': {'column_id': 'country'}, 'backgroundColor': '#3D9970', 'color': 'white'},
        ],
        pagination_settings={
            'current_page': 0,
            'page_size': 5
        },
        pagination_mode='be',
        sorting='be',
        sorting_type='single',
        sorting_settings=[]
    ),

])

@app.callback(
    Output('datatable-interactivity', 'data'),
    [Input('datatable-interactivity', 'pagination_settings'),
     Input('datatable-interactivity', 'sorting_settings')])
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

if __name__ == '__main__':
    app.run_server(debug=True)
```











