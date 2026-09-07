---
title: 自定义HOOK
source: https://www.yuque.com/mook-mvqcp/wc9wsu/uhbch9e7b0v77dx1
created: 2026-05-27
updated: 2026-05-27
tags:
  - 面试准备
---

**1.useBacktest**

运行回测涉及选择策略、配置参数、指定测试文件和模型文件等多个步骤，每一步都会改变组件状态，且这些状态之间存在联动关系（如切换策略需重置参数、运行成功后需触发图表刷新）。如果将这些状态和逻辑散落在组件中，会导致组件臃肿且容易遗漏联动逻辑。因此将其抽象为 `useBacktest` 自定义 hook，把回测这一业务域的所有状态与操作内聚在一起，组件层只需调用 `useBacktest()` 解构出所需的状态和方法，专注于消费结果和渲染 UI，无需关心内部的状态管理细节。

Situation（情境）

交易回测涉及多个可变维度：策略类型（均线交叉、RSI、MACD 等）、每种策略有不同的参数、可选绑定已训练模型、可选指定数据文件。用户操作流程是"选策略 → 调参数 → 点运行 → 看结果"，还要能从历史记录点进去回看之前的回测详情。这些状态——策略名、参数对象、运行中标识、返回结果、错误信息——原本都是散落的 useState，和训练逻辑、视图切换逻辑全挤在 DashboardLayout 一个文件里。

Task（任务）

把回测的全部状态和操作封装成一个独立 Hook，DashboardLayout 只做"组装配件"，不碰回测的内部细节。核心要覆盖三条链路：运行新回测、从历史记录加载回测详情、切换策略时自动重置参数。

Action（行动）

第一个链路——运行回测：handleRunBacktest 收集当前的策略类型 + 参数 + 可选模型 ID + 可选数据文件，组装成请求体调 POST /api/backtest，拿到结果后同时更新 backtestResult（展示）和 refreshTrigger（触发历史列表自动刷新）
第二个链路——加载历史详情：handleBacktestHistorySelect(id) 调 GET /api/backtest/:id，拿到完整结果后把 strategy_type、strategy_params、回测数据一起塞进 state，同时切换到"运行回测"视图展示——这样用户从历史列表点进去，右侧面板的参数和图表都自动还原到当时的状态
第三个细节——策略切换时自动重置参数：handleStrategyChange(key) 不光是改 selectedStrategy，还会把 strategyParams 重置为该策略的默认值并清空 error，防止上一个策略的参数残留在新策略里导致请求失败
整组状态（8 个）和 4 个操作函数对外暴露为一个对象，DashboardLayout 一行 const backtest = useBacktest() 全部拿到
Result（结果）

DashboardLayout 删掉了 ~70 行回测相关逻辑。RightSidebar 只负责渲染参数面板的 UI，不管理状态——它拿到的 selectedStrategy、strategyParams、onStrategyChange、onRunBacktest 全部来自 Hook。三条链路（新回测 / 看历史 / 切策略）各自独立、互不干扰，新增策略类型时只需在 constants/strategies.js 加配置，Hook 逻辑零改动。

**2.useTrainning**

这个hook的实现思路和useBacktest一样，只不过能力不同。

useTraining——WebSocket 实时训练监控
Situation（情境）

模型训练是一个异步长任务，后端通过 WebSocket 推送每个 epoch 的 loss、val_loss、best_epoch 等数据。前端需要展示实时训练曲线、进度条，且要处理用户中途取消、训练正常完成、训练失败三种终态。原本这堆逻辑全在 DashboardLayout 里，和视图切换、回测逻辑搅在一起。

Task（任务）

把训练全生命周期——参数管理、发起请求、WebSocket 实时流、取消、完成/失败处理——封装进 useTraining，对外只暴露状态和操作方法。

Action（行动）

用户点"开始训练"后，先 POST /api/training 拿到 job ID，立刻创建 WebSocket 连接
onEpoch 回调里把每条 epoch 日志 push 进 liveTraining.epochLogs，驱动 Recharts 实时绘制 loss 曲线
onCompleted → 关闭 WebSocket、刷新模型列表、清除 loading 态；onFailed → 展示错误信息
用户点"取消"时调用 wsRef.current.close() 断开连接，setIsTraining(false) 重置所有状态——这里用 useRef 存 WebSocket 实例而非 state，因为 WS 对象不需要触发重渲染
对外暴露 trainingParams、liveTraining、trainingResult、selectedModelId 等，DashboardLayout 不关心训练内部怎么跑
Result（结果）

DashboardLayout 删掉了 ~80 行训练相关逻辑，训练功能变成一个可插拔模块。TrainingContent 和 TrainingRightPanel 各自只从 Hook 拿自己需要的状态，职责清晰。WebSocket 三种终态都有对应的 UI 反馈，取消训练不会残留连接。

**3.usePaginatedList**

基础设施型hook，提供通用的分页列表的能力。

usePaginatedList——消灭重复的列表加载代码
Situation（情境）

回测历史、模型列表、数据集列表三个页面，每个都要写一遍同样的逻辑：useState 存 list/total/loading/error，useEffect 调 API，最后 try/catch/finally 处理三种状态。三份代码结构完全一样，只是调用的 API 函数不同。

Task（任务）

抽象一个通用的分页列表 Hook，约定统一的 API 返回格式 { items, total }，让所有列表页面只关心"调哪个 API"这一个变量。

Action（行动）

写了 usePaginatedList(apiFn)，接收 API 函数作为参数——fetchList 内部调用 apiFn(0, pageSize)，拿到结果后统一塞进 list 和 total
加了一个 refreshTrigger 参数：外部组件跑完回测或训练后 setRefreshTrigger(prev => prev + 1)，Hook 内部的 useEffect 监听到变化自动重新请求，不需要外部手动调 fetchList
配套写了 useDeleteConfirm——封装 window.confirm → 调删除 API → 成功后回调的完整流程
Result（结果）

三个列表页面各自从 ~50 行削减到 ~15 行。新加一个需要分页列表的页面，只要传一个 API 函数给 Hook 就行。refreshTrigger 机制让"操作完成后刷新列表"变成自动行为，不用在回调里手动调。

**4.useSearchDropdown**

useSearchDropdown——搜索防抖 + Portal 浮层
Situation（情境）

项目有两个页面（数据获取页、预测页）都需要股票搜索功能——用户输入关键词、实时展示搜索建议、选中后填入表单。两个页面的逻辑几乎一模一样：调 API、展示下拉框、防抖、点击外部关闭、滚动时跟随定位。如果不抽象，要写两遍。

Task（任务）

把搜索的全部状态和交互逻辑封装成一个可复用的 Hook，两个页面只负责渲染 UI，不碰逻辑细节。同时解决一个 CSS 层面的硬伤：搜索框被包裹在侧边栏或主内容区的 overflow: hidden 容器里，下拉框正常渲染会被裁切。

Action（行动）

写了 useSearchDropdown Hook，内部用 useRef 存定时器 ID 实现 300ms 防抖——每次按键先 clearTimeout，用户停止输入 300ms 后才发 API 请求；空内容直接清空，不发请求
用 createPortal 把下拉框渲染到 document.body，彻底脱离父容器的 CSS 限制；定位通过 inputRef.current.getBoundingClientRect() 拿到输入框的屏幕坐标，设 position: fixed + z-index 9999 贴在输入框正下方
在 window 上挂 scroll 监听（捕获阶段），页面滚动时实时重新计算位置防止脱节；在 document 上挂 mousedown 监听，点击输入框和下拉框之外的区域自动关闭
对外暴露 keyword、searchResults、inputRef、dropdownRef、dropdownStyle 等，组件拿到直接解构绑定
Result（结果）

两个页面的搜索相关代码从各 ~40 行削减到 ~15 行——只负责调 Hook 和在 JSX 里绑 ref。Hook 作为一个完整的独立模块，后续加第三个搜索场景时零改动直接复用。下拉框不受任何父容器裁剪，固定在输入框下方，响应流畅
