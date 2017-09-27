# redux your web app

### redux简介

简单来说，redux 就是帮我们统一管理了 react 组件的 state 状态。

为什么要使用 redux 统一管理 state 呢？没有 redux 我们依旧可以开发 APP，但是当 APP 的复杂度到达一定程度的时候，摆在我们面前的就是 难以维护 的代码（其中包含组件大量的异步回调，数据处理等等），但是使用 redux 也会增加我们整个项目的复杂度，这就需要我们在两者之间进行权衡了，对于这一部分，redux 开发者给我们下面几个参考点：

>以下几种情况不需要使用 redux：

* 整体 UI 很简单，没有太多交互。

* 不需要与服务器进行大量交互，也没有使用 WebSocket。

* 视图层只从单一来源获取数据。

>以下几种情况可考虑使用 redux：

* 用户的交互复杂。

* 根据层级用户划分功能。

* 多个用户之间协作。

* 与服务器大量交互，或使用了 WebSocket。

* 视图层需要从多个来源获取数据。


总结以上内容：redux 适用于 多交互，多数据源，复杂程度高的工程中。

也就是说，当我们的组件出现 某个状态需要共享，需要改变另一个组件状态 等传值比较不容易的情况。就可以考虑 redux ，当然还有其他 redux 的替代产品供我们使用。。

### 重要内容
![](http://upload-images.jianshu.io/upload_images/2041009-06f5967c685f961b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>为了避免混乱，我们不应该直接修改 redux 的状态，需要有特定的办法来修改状态。
首先我们需要 **action** 来触发一个行为，告知 redux 我们需要修改状态，
然后应由专门生产新状态的 **reducer** 来产生新的状态

##### action
> action 是一个对象，包含这个行为的 **类型** 和必要的参数(可选)。action也可写成一个返回对象的函数(ActionCreator)

```js
function testAction(key1, key2, ...keyN) {
          return {
              type: "TEST_ACTION",
              key1: key1,
              //...
              keyN: keyN
          }
      }
```
##### reducer
> reducer 是一个纯函数，满足以下条件：
* 相同输入必须有相同输出
* 不能修改传入的参数
* 不能包含 random、Date 等非纯函数

***另外，reducer 每次返回的必须是一个全新的状态***
```js

const initialState = {
  article: {}
};

export function article(state = initialState, action) {
  switch (action.type) {
    case Actions.ADD_ARTICLE_TAG: {
      let article = Object.assign(state.article);
      if(article.tags){
        article.tags.push(action.tag);
      }else{
        article.tags = [action.tag];
      }
      return { ...state, article: article };
    }
  }
}
```

### 在 react 项目中使用 redux
##### 安装
```
 npm install --save redux
 npm install --save react-redux
 npm install --save-dev redux-devtools
```
##### 配置
###### 在 index.js 文件：
```
import {createStore, applyMiddleware} from 'redux';
import {Provider} from 'react-redux';
import thunk from 'redux-thunk';
import reducers from './src/reducers/index';

const createStoreWithMiddleware = applyMiddleware(thunk)(createStore);
const store = createStoreWithMiddleware(reducers);

const router = (
  <Provider store={store}>   // 在根组件注入store
    <Router history={hashHistory}>
      <Route path="/">
        <IndexRoute component={Login} />
        <Route path="/articles" component={Articles}/>
        <Route path="/articledetail" component={ArticleDetail}/>
      </Route>
    </Router>
  </Provider>

)

ReactDOM.render(router, document.getElementById('root'));

```
以上： 我们先引入redux和一些函数，再引入写好的 reducer ，`createStoreWithMiddleware` 函数创建了
一个顶级的管理所有状态的储存器，统一管理状态。然后将生成的 store 通过 `Provider` 组件注入，
这样为 `Provider` 的子组件提供了状态的获取途径。

###### 