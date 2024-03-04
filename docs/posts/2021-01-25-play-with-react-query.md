---
date: 2021-01-25 09:40:34
category:
  - dev
tag:
  - react
---

# play with react-query

[react-query](https://github.com/tannerlinsley/react-query) 是一个轻量级的状态管理工具，注重于解决异步请求数据的缓存问题，相比于使用 redux 来管理请求数据的状态/缓存，react-query 更加轻便简单，只需要使用 useQuery 就可以自动实现数据的缓存和管理, 还可以使用 useIsFetching 来监听全局加载状态

## 解决什么问题

- 缓存请求的数据
- 将多个重复请求合并成一个，避免多余的网络开销
- 自动更新过期缓存数据
- 知道什么时候缓存数据过期了
- 快速更新缓存数据
- 管理服务端状态的内存/垃圾回收
- 结构化存储请求的数据，根据 key-value 取值

<!--more-->

## show me the code

```js
import {QueryClient, QueryClientProvider, useQuery} from 'react-query';

const queryClient = new QueryClient();

export default function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <Example />
    </QueryClientProvider>
  );
}

function Example() {
  const {isLoading, error, data} = useQuery('repoData', () =>
    fetch('https://api.github.com/repos/tannerlinsley/react-query').then(
      (res) => res.json()
    )
  );

  if (isLoading) return 'Loading...';

  if (error) return 'An error has occurred: ' + error.message;

  return (
    <div>
      <h1>{data.name}</h1>
      <p>{data.description}</p>
      <strong>👀 {data.subscribers_count}</strong>{' '}
      <strong>✨ {data.stargazers_count}</strong>{' '}
      <strong>🍴 {data.forks_count}</strong>
    </div>
  );
}
```

## 有趣的 devtool

会在页面上添加一个浮动按钮，点击可以查看 query 的记录，好比一个简化的 redux 的 devtool

```js
import {ReactQueryDevtools} from 'react-query/devtools';

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      {/* The rest of your application */}
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  );
}
```

[更多文档](https://react-query.tanstack.com/overview)
