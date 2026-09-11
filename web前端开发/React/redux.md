---
title: redux
source: https://www.yuque.com/mook-mvqcp/wc9wsu/lmlx6dymegb7klg4
created: 2026-05-11
updated: 2026-05-11
tags:
  - React
---

**createAsyncThunk()**
这个函数有三个参数，分别是typePrefix、payloadCreator、options?，返回一个AsyncThunk action creator（这个返回值是一个函数，执行它返回一个thunkFunction（包含了执行payloadCreator的代码），thunkFuntion会被redux-thunk拦截并执行；里面包含pending、fulfilled和rejected三个action creator，它们就是普通的action creator，在特定阶段被dispatch调用产生action传递给store调用对应的reducer，fulfilled和rejected这两个action creator的payload来自payloadCreator的执行结果。）
