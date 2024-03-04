---
date: 2020-09-30 10:40:15
category:
  - dev
tag:
  - react
  - unit-test
---

# play with react-testing-library

## 为什么进行单元测试

- 提高代码质量（写易于测试的代码，模块化，函数化）
- 避免由于修复 bug 而导致新 bug
- 让你重构更有信心

<!--more-->

## 主要工具

- [@testing-library/react](https://www.npmjs.com/package/@testing-library/react)
- [@testing-library/jest-dom](https://www.npmjs.com/package/@testing-library/jest-dom)
- [react-test-renderer](https://www.npmjs.com/package/react-test-renderer)

如果是使用的`create-react-app`创建的项目，自带有 RTL，无需单独配置

只需添加

```bash
yarn add --dev react-test-render
```

## react-testing-library(RTL) vs enzyme

### Enzyme

一个社区主流测试工具

侧重于 dom 测试，测试风格偏向于测试实现的方式，比如 dom 的内容，组件的状态，可能写着写着就变成了重写代码?

### RTL

是一个新兴的测试工具

侧重于 UI 角度的测试，即从用户的视角出发（文案，点击），测试所见所得是否符合预期

### 目录结构

```bash
src/
    components/
        Comp/
            __test__
                Comp.test.js
    pages/
        XXXPage/
            __test__
                XXXPage.test.js
```

### AAA 规则

- Arrange 编排渲染组件 （使用`data*testid`来查询组件）
- Act 执行操作
- Assert 断言

## 测试实践

> 以下代码来自[参考](http://blog.itpub.net/31559758/viewspace-2682344/)，仅用于个人学习

### Snapshot Testing

RTL

```js
import React from 'react';
import {render, cleanup} from '@testing-library/react';
import App from '../App';
afterEach(cleanup); // 每个测试结束后结束组件生命
it('should take a snapshot', () => {
  const {asFragment} = render(<App />);
  expect(asFragment()).toMatchSnapshot();
});
```

Renderer

```js
import React from 'react';
import renderer from 'react-test-renderer';
import App from '../App';
afterEach(cleanup);
it('should take a snapshot', () => {
  const tree = renderer.create(<App />).toJson();
  expect(tree).toMatchSnapshot();
});
```

### DOM Testing

要测试的组件:

```js
// TestElements.js
import React from 'react';
const TestElements = () => {
  const [counter, setCounter] = React.useState(0);
  return (
    <>
      <h1 data-testid="counter">{counter}</h1>
      <button data-testid="button-up" onClick={() => setCounter(counter + 1)}>
        {' '}
        Up
      </button>
      <button
        disabled
        data-testid="button-down"
        onClick={() => setCounter(counter - 1)}
      >
        Down
      </button>
    </>
  );
};
export default TestElements;
```

```js
// TestElements.test.js
import React from 'react';
import '@testing-library/jest-dom/extend-expect';
import {render, cleanup} from '@testing-library/react';
import TestElements from '../components/TestElements';
afterEach(cleanup);
it('should equal to 0', () => {
  const {getByTestId} = render(<TestElements />);
  expect(getByTestId('counter')).toHaveTextContent(0);
});
it('should be enabled', () => {
  const {getByTestId} = render(<TestElements />);
  expect(getByTestId('button-up')).not.toHaveAttribute('disabled');
});
it('should be disabled', () => {
  const {getByTestId} = render(<TestElements />);
  expect(getByTestId('button-down')).toBeDisabled();
});
```

### Event Testing

```js
// TestEvents.js
import React from 'react';
const TestEvents = () => {
  const [counter, setCounter] = React.useState(0);
  return (
    <>
      <h1 data-testid="counter">{counter}</h1>
      <button data-testid="button-up" onClick={() => setCounter(counter + 1)}>
        {' '}
        Up
      </button>
      <button data-testid="button-down" onClick={() => setCounter(counter - 1)}>
        Down
      </button>
    </>
  );
};
export default TestEvents;
```

```js
// TestEvents.test.js
import React from 'react';
import '@testing-library/jest-dom/extend-expect';
import {render, cleanup, fireEvent} from '@testing-library/react';
import TestEvents from '../components/TestEvents';
afterEach(cleanup);
it('increments counter', () => {
  const {getByTestId} = render(<TestEvents />);
  fireEvent.click(getByTestId('button-up'));
  expect(getByTestId('counter')).toHaveTextContent('1');
});
it('decrements counter', () => {
  const {getByTestId} = render(<TestEvents />);
  fireEvent.click(getByTestId('button-down'));
  expect(getByTestId('counter')).toHaveTextContent('-1');
});
```

### Async Testing

```js
// TestAsync.js
import React from 'react';
const TestAsync = () => {
  const [counter, setCounter] = React.useState(0);
  const delayCount = () =>
    setTimeout(() => {
      setCounter(counter + 1);
    }, 500);
  return (
    <>
      <h1 data-testid="counter">{counter}</h1>
      <button data-testid="button-up" onClick={delayCount}>
        {' '}
        Up
      </button>
      <button data-testid="button-down" onClick={() => setCounter(counter - 1)}>
        Down
      </button>
    </>
  );
};
export default TestAsync;
```

```js
// TestAsync.test.js
import React from 'react';
import '@testing-library/jest-dom/extend-expect';
import {
  render,
  cleanup,
  fireEvent,
  waitForElement,
} from '@testing-library/react';
import TestAsync from '../components/TestAsync';
afterEach(cleanup);
it('increments counter after 0.5s', async () => {
  const {getByTestId, getByText} = render(<TestAsync />);
  fireEvent.click(getByTestId('button-up'));
  const counter = await waitForElement(() => getByText('1'));
  expect(counter).toHaveTextContent('1');
});
```

### React Redux Testing

```js
// TestRedux.js
import React from 'react';
import {connect} from 'react-redux';
const TestRedux = ({counter, dispatch}) => {
  const increment = () => dispatch({type: 'INCREMENT'});
  const decrement = () => dispatch({type: 'DECREMENT'});
  return (
    <>
      <h1 data-testid="counter">{counter}</h1>
      <button data-testid="button-up" onClick={increment}>
        Up
      </button>
      <button data-testid="button-down" onClick={decrement}>
        Down
      </button>
    </>
  );
};
export default connect((state) => ({counter: state.count}))(TestRedux);
```

```js
// store/reducer.js
export const initialState = {
  count: 0,
};
export function reducer(state = initialState, action) {
  switch (action.type) {
    case 'INCREMENT':
      return {
        count: state.count + 1,
      };
    case 'DECREMENT':
      return {
        count: state.count - 1,
      };
    default:
      return state;
  }
}
```

```js
// TestRedux.test.js
import React from 'react';
import '@testing-library/jest-dom/extend-expect';
import {createStore} from 'redux';
import {Provider} from 'react-redux';
import {render, cleanup, fireEvent} from '@testing-library/react';
import {initialState, reducer} from '../store/reducer';
import TestRedux from '../components/TestRedux';
const renderWithRedux = (
  component,
  {initialState, store = createStore(reducer, initialState)} = {}
) => {
  return {
    ...render(<Provider store={store}>{component}</Provider>),
    store,
  };
};
afterEach(cleanup);
it('checks initial state is equal to 0', () => {
  const {getByTestId} = renderWithRedux(<TestRedux />);
  expect(getByTestId('counter')).toHaveTextContent('0');
});
it('increments the counter through redux', () => {
  const {getByTestId} = renderWithRedux(<TestRedux />, {
    initialState: {count: 5},
  });
  fireEvent.click(getByTestId('button-up'));
  expect(getByTestId('counter')).toHaveTextContent('6');
});
it('decrements the counter through redux', () => {
  const {getByTestId} = renderWithRedux(<TestRedux />, {
    initialState: {count: 100},
  });
  fireEvent.click(getByTestId('button-down'));
  expect(getByTestId('counter')).toHaveTextContent('99');
});
```

### React Context Testing

```js
// Counter.js
import React from 'react';
export const CounterContext = React.createContext();
const CounterProvider = () => {
  const [counter, setCounter] = React.useState(0);
  const increment = () => setCounter(counter + 1);
  const decrement = () => setCounter(counter - 1);
  return (
    <CounterContext.Provider value={{counter, increment, decrement}}>
      <Counter />
    </CounterContext.Provider>
  );
};
export const Counter = () => {
  const {counter, increment, decrement} = React.useContext(CounterContext);
  return (
    <>
      <h1 data-testid="counter">{counter}</h1>
      <button data-testid="button-up" onClick={increment}>
        {' '}
        Up
      </button>
      <button data-testid="button-down" onClick={decrement}>
        Down
      </button>
    </>
  );
};
export default CounterProvider;
```

```js
// TextContext.test.js
import React from 'react';
import '@testing-library/jest-dom/extend-expect';
import {render, cleanup, fireEvent} from '@testing-library/react';
import CounterProvider, {
  CounterContext,
  Counter,
} from '../components/TestContext';
const renderWithContext = (component) => {
  return {
    ...render(
      <CounterProvider value={CounterContext}>{component}</CounterProvider>
    ),
  };
};
afterEach(cleanup);
it('checks if initial state is equal to 0', () => {
  const {getByTestId} = renderWithContext(<Counter />);
  expect(getByTestId('counter')).toHaveTextContent('0');
});
it('increments the counter', () => {
  const {getByTestId} = renderWithContext(<Counter />);
  fireEvent.click(getByTestId('button-up'));
  expect(getByTestId('counter')).toHaveTextContent('1');
});
it('decrements the counter', () => {
  const {getByTestId} = renderWithContext(<Counter />);
  fireEvent.click(getByTestId('button-down'));
  expect(getByTestId('counter')).toHaveTextContent('-1');
});
```

### React Router Testing

```js
// TestRouter.js
import React from 'react';
import {Link, Route, Switch, useParams} from 'react-router-dom';
const About = () => <h1>About page</h1>;
const Home = () => <h1>Home page</h1>;
const Contact = () => {
  const {name} = useParams();
  return <h1 data-testid="contact-name">{name}</h1>;
};
const TestRouter = () => {
  const name = 'John Doe';
  return (
    <>
      <nav data-testid="navbar">
        <Link data-testid="home-link" to="/">
          Home
        </Link>
        <Link data-testid="about-link" to="/about">
          About
        </Link>
        <Link data-testid="contact-link" to={`/contact/${name}`}>
          Contact
        </Link>
      </nav>
      <Switch>
        <Route exact path="/" component={Home} />
        <Route path="/about" component={About} />
        <Route path="/about:name" component={Contact} />
      </Switch>
    </>
  );
};
export default TestRouter;
```

```js
// TestRouter.test.js
import React from 'react';
import '@testing-library/jest-dom/extend-expect';
import {Router} from 'react-router-dom';
import {render, fireEvent} from '@testing-library/react';
import {createMemoryHistory} from 'history';
import TestRouter from '../components/TestRouter';
const renderWithRouter = (component) => {
  const history = createMemoryHistory();
  return {
    ...render(<Router history={history}>{component}</Router>),
  };
};
it('should render the home page', () => {
  const {container, getByTestId} = renderWithRouter(<TestRouter />);
  const navbar = getByTestId('navbar');
  const link = getByTestId('home-link');
  expect(container.innerHTML).toMatch('Home page');
  expect(navbar).toContainElement(link);
});
it('should navigate to the about page', () => {
  const {container, getByTestId} = renderWithRouter(<TestRouter />);
  fireEvent.click(getByTestId('about-link'));
  expect(container.innerHTML).toMatch('About page');
});
it('should navigate to the contact page with the params', () => {
  const {container, getByTestId} = renderWithRouter(<TestRouter />);
  fireEvent.click(getByTestId('contact-link'));
  expect(container.innerHTML).toMatch('John Doe');
});
```

### Http Request Testing

```js
// TestAxios.js
import React from 'react';
import axios from 'axios';
const TestAxios = ({url}) => {
  const [data, setData] = React.useState();
  const fetchData = async () => {
    const response = await axios.get(url);
    setData(response.data.greeting);
  };
  return (
    <>
      <button onClick={fetchData} data-testid="fetch-data">
        Load Data
      </button>
      {data ? (
        <div data-testid="show-data">{data}</div>
      ) : (
        <h1 data-testid="loading">Loading...</h1>
      )}
    </>
  );
};
export default TestAxios;
```

```js
// TextAxios.test.js
import React from 'react';
import '@testing-library/jest-dom/extend-expect';
import {render, waitForElement, fireEvent} from '@testing-library/react';
import axiosMock from 'axios';
import TestAxios from '../components/TestAxios';
jest.mock('axios');
it('should display a loading text', () => {
  const {getByTestId} = render(<TestAxios />);
  expect(getByTestId('loading')).toHaveTextContent('Loading...');
});
it('should load and display the data', async () => {
  const url = '/greeting';
  const {getByTestId} = render(<TestAxios url={url} />);
  axiosMock.get.mockResolvedValueOnce({
    data: {greeting: 'hello there'},
  });
  fireEvent.click(getByTestId('fetch-data'));
  const greetingData = await waitForElement(() => getByTestId('show-data'));
  expect(axiosMock.get).toHaveBeenCalledTimes(1);
  expect(axiosMock.get).toHaveBeenCalledWith(url);
  expect(greetingData).toHaveTextContent('hello there');
});
```
