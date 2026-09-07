---
title: react router
source: https://www.yuque.com/mook-mvqcp/wc9wsu/fpx82a4q4gz677kp
created: 2026-05-22
updated: 2026-05-22
tags:
  - React
---

React Router有三种编写模式，分别是声明式、数据式和框架模式。

### 一、声明式导航快速入门

声明式Router最简单，适合快速入门。

**1.安装React Router并导入相关API**

npm install react-router-dom是安装指令。

import { BrowserRouter, Routes, Route } from 'react-router-dom'导入声明式路由的API

**2.配置**

```jsx

// main.jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import Home from './pages/Home';
import About from './pages/About';
import NotFound from './pages/NotFound';

ReactDOM.createRoot(document.getElementById('root')).render(
  <BrowserRouter>
    <Routes>
      <Route path="/" element={<Home />} />
      <Route path="/about" element={<About />} />
      <Route path="*" element={<NotFound />} />  {/* 404 路由 */}
    </Routes>
  </BrowserRouter>
);

```

<BrowserRouter>用于监听URL变化，必须放在最外层。

<Routes 会匹>配当前 URL，并渲染第一个 `path` 匹配的 `<Route>`。

**3.声明式导航**

使用<Link>代替<a>标签，实现无刷新跳转

```jsx

// 在任何组件中
import { Link } from 'react-router-dom';

function Navbar() {
  return (
    <nav>
      <Link to="/">首页</Link>
      <Link to="/about">关于</Link>
    </nav>
  );
}

```

**4.嵌套子页面**

通过在父路由内部再写<Route>实现嵌套。注意父组件中要使用<Outlet>指定子路由的渲染位置。

**5.动态路由参数-useParams**

```jsx

// 路由定义：<Route path="products/:productId" element={<ProductDetail />} />
// ProductDetail.jsx
import { useParams } from 'react-router-dom';

function ProductDetail() {
  const { productId } = useParams();
  return <h2>正在查看商品 ID：{productId}</h2>;
}

```

**6.编程式导航-useNavgate**

```jsx

import { useNavigate } from 'react-router-dom';

function LoginButton() {
  const navigate = useNavigate();
  const handleLogin = () => {
    // 登录逻辑...
    navigate('/dashboard');  // 跳转到仪表盘
  };
  return <button onClick={handleLogin}>登录</button>;
}

```

**7.路由守卫**

React Router的声明式路由模式没有内置的路由守卫，但可以通过条件渲染+重定向组件实现。

（1）方法一：封装ProtectedRoute组件

创建一个组件，内部判断是否满足条件（如登录状态），不满足则重定向。

```jsx

// components/ProtectedRoute.jsx
import { Navigate } from 'react-router-dom';

function ProtectedRoute({ children, isAllowed, redirectTo = '/login' }) {
  if (!isAllowed) {
    return <Navigate to={redirectTo} replace />;
  }
  return children;
}

export default ProtectedRoute;

```

在路由定义中使用，ProtectedRoute包裹受保护的组件（children）。

```jsx

// App.jsx
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import ProtectedRoute from './components/ProtectedRoute';
import Dashboard from './pages/Dashboard';
import Login from './pages/Login';

const isAuthenticated = false; // 可从 context / store 中获取

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/login" element={<Login />} />
        <Route
          path="/dashboard"
          element={
            <ProtectedRoute isAllowed={isAuthenticated}>
              <Dashboard />
            </ProtectedRoute>
          }
        />
      </Routes>
    </BrowserRouter>
  );
}

```

（2）方法二：在路由元素中直接中断

```jsx

<Route
  path="/dashboard"
  element={
    isAuthenticated ? <Dashboard /> : <Navigate to="/login" replace />
  }
/>

```

**8.路由懒加载**

react-router-dom中的lazy()方法可以懒加载组件，参数是一个loader（例如，()=>import('path')）

注意，使用了懒加载的路由必须用<Suspence>组件包裹。
