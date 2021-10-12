## Redux

```
npm i react-redux
```



- 자바스크립트 어플리케이션을 위한 라이브러리

- React, Angular, Vue, Vanilla에 사용

- 어플리케이션의 상태를 저장, 관리

- React-redux를 통해 리액트에 사용

  

#### Redux를 왜 사용할까?

- 리액트는 컴포넌트가 상당히 많이 구조화 되어 있음 

  ->  계층화되고 구조화 되면 재사용성이 좋은 장점이 있지만 상태를 관리할 때 어려운 부분이 있음

  -> why?

  - 리액트의 각 컴포넌트는 각각의 state를 가지고 있기 때문

- 어딘가에 저장을 해놓고 필요한 컴포넌트에서 직접적으로 바로 데이터를 가져올 수 있는 방법이 필요해서 사용하는것이 redux



#### Redux core concept 4가지

- component: 화면에 보여줄 view
- store: 어떠한 정보를 저장하는 공간
  - component에서 어떠한 값이 변동이 일어나게 되면 그 값을 store에 업데이트 해야함
  - 그 때 action 과 reducer를 통해 관리를 하게 됨

![reduxcore](https://user-images.githubusercontent.com/83201109/136780079-0b9145f7-d475-472b-826d-c63752e44a50.png)

1. component에서 action으로 dispatch를 통해 action으로 호출
2. action에 정의되어 있는 내용이 reducer에 의해서 핸들링이 되고 그 핸들링에 따라서
3. 상태값이 store에 업데이트 됨
4. 업데이트된 store를 subscribe를 통해  실시간으로 받아와서 component에서 사용할 수 있게 할 수 있음



##### redux 폴더

```
redux
	|--- drawerManagement
	|	   |--- action.js
	|	   |--- reducer.js
	|	   |--- types.js
	|--- viewManagement
	|	   |--- action.js
	|	   |--- reducer.js
	|	   |--- types.js
	|--- rootReducer.js
	|--- store.js
```



###### drawerManagement/types.js

```
export const ChangeState = 'CheckState';
```

- type을 action, reducer에서 사용해야하기 때문에 따로 관리하는 것이 좋음 

  

###### drawerManagement/action.js

```
import { ChangeState } from './types';

export const drawerManage = () => {
    return {
        type: ChangeState
    }
}
```



###### drawerManagement/reducer.js

```
import { ChangeState } from './types';

const initialState = {
    open: true
};

const drawerManageReducer = (state=initialState, action) => {	
    switch(action.type){
        case ChangeState:
            return{
                ...state,
                open: !state.open
            }
        default: return state
    }
};

export default drawerManageReducer;
```

- state=initialState: state에 어떠한 값도 들어오지 않으면 initialState를 기본값으로 받아서 사용



###### rootReducer.js

```
import { combineReducers } from 'redux';
import drawerManageReducer from './drawerManagement/reducer';
import viewManageReducer from './viewManagement/reducer';

const rootReducer = combineReducers({
    drawerManageReducer,
    viewManageReducer
});

export default rootReducer;
```



###### store.js

```
import { createStore } from 'redux';
import rootReducer from './rootReducer'

const store = createStore(rootReducer);

export default store;
```



###### Header.js

```
import React from 'react';
import { connect } from 'react-redux';
import { drawerManage } from '../redux/drawerManagement/actions';

const Header = ({ open, drawerManage }) => {
	return (
	.
	.
      <IconButton
        edge="start"
        color="inherit"
        aria-label="open drawer"
        onClick={() => { drawerManage() }}
        className={clsx(classes.menuButton, open && classes.menuButtonHidden)}>
    .
    .
	)
}

const mapStateToProps = (state) => {
  return {
    open: state.drawerManageReducer.open	// open이 Header.js props의 open으로 전달이 됨
  }
}

const mapDispatchToProps = {	// Function | Object로 넘김 
  drawerManage	// drawerManage: drawerManage(es6에서는 property == value이면 생략가능)
}

export default connect(mapStateToProps, mapDispatchToProps)(Header);
```





### Redux-Persist

```
npm install redux-persist
```



#### Redux-Persist를 왜 사용할까?

- redux의 store는 페이지를 새로고침 할 경우 state가 날아가기 때문에
- 이것에 대한 대응 방안으로 localStorage 또는 session에 저장하고자 하는 reducer state를 저장하여, 새로고침 하여도 저장공간에 있는 데이터를 redux에 불러오는 형식으로 이루어짐
- 위에서 말한 이 작동을 위해 `redux-persist`를 사용



#### reducer에 persist store 정의

- localStorage에 저장: `import storage from 'redux-persist/lib/storage`
- session Storage에 저장:`import storageSession from 'redux-persist/lib/storage/session`



###### store.js

```
import { createStore } from 'redux';
import rootReducer from './rootReducer'
import { persistStore, persistReducer } from 'redux-persist';
import storageSession from 'redux-persist/lib/storage/session';

const persistConfig = {
    key: 'root',
    storage: storageSession
};

const enhancedReducer = persistReducer(persistConfig, rootReducer);

export default function configureStore(){
    const store = createStore(enhancedReducer);
    const persist = persistStore(store);
    return {store, persist};
}
```

- persisConfig = {} : 새로운 persist 선언
  - key : reducer의 어느 지점에서부터 데이터를 저장할 것 인지
- persistReducer(persisConfig, rootReducer) : persisConfig가 추가된 reducer 반환
- const store = createStore(enhancedReducer) : persistConfig가 추가된 enhancedReducer로 store를 생성
- persistStore : 새로 고침, 종료해도 지속될 store 생성



###### App.js

```
import React, { Fragment } from 'react';
import UcareRoute from './routes/UcareRoute';
import { Provider } from 'react-redux';
import { PersistGate } from 'redux-persist/integration/react';
import configureStore from './redux/store';

const { store, persist } = configureStore();

export default function App() {
    return (
        <RecoilRoot>
            <Provider store={store}>
                <PersistGate loading={null} persistor={persist}>
                    <Fragment>
                        <UcareRoute />
                    </Fragment>
                </PersistGate>
            </Provider>
        </RecoilRoot>
    );
}
```

- PersistGage : react로 개발한다면 App 컴포는터를 PersistGate로 매핑
- persist store가 redux에 저장될 때까지 앱 UI 렌더링이 지연됨

