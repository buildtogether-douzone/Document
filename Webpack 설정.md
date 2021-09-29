### Webpack 설정

#### 개발환경

+ React



#### 설치

```bash
npm i -D webpack webpack-cli
```



#### 폴더 구성

```bash
ucare_frontend
	  | --- config
	  | 	   | --- webpack.config.js
	  | 	   | --- babel.config.js
	  | --- public
	  | 	   | --- favicon.ico
	  | 	   | --- index.html
	  | --- src
              .
              .
              .
              .
              .
```



#### 실행

+ package.json

```json
"scripts" : {
    "build" : "webpack"
}
```

+ 위와 같이 실행을 하면 rootDir 에 있는 webpack.config.js 파일이 실행이된다. 
+ 하지만 작성자는 config directory 를 만들어 관리를 하고 싶어서 package.json에 따로 설정을 해주었다

``` json
"scripts" : {
    "dev:frontend": "cross-env NODE_ENV=development webpack serve --config ucare_frontend/config/webpack.config.js --mode development --progress"
}
```

+ cross-env 는 환경 변수를 설정하기 위해서 사용하는데 현재 상태가 배포인지 개발인지 설정하기 위해서 NODE_ENV를 설정한다.  
+ webpack serve --config ucare_frontend/config/webpack.config.js --mode development --progress" 는 ucare_frontend/config 에있는  webpack.config.js 를 실행 시키기 위해서 사용 했다.



#### 설정 

+ webpack.config.js

```javascript
const path = require('path');

module.exports = {
    mode: 'development',
    entry: path.resolve('ucare_frontend/src/index.js'),
    output: {
    	path: path.resolve('ucare_backend/src/main/webapp/assets'),
    	filename: 'js/bundle.js'
        assetModuleFilename: 'images/[hash][ext]'
    }
}
```

+ mode는 개발용으로 실행 할 건지 아니면 배포용으로 실행을 할 건지 설정을 하는 부분이다.

+ entry는 소스코드의 시작점을 알려준다. (보통 index.js)
+ output 의 path는 파일이 저장될 절대 경로를 말하고 filename 은 ucare_backend/src/main/webapp/assets의 js 폴더에 bundle.js 라는 파일 이름으로  저장 하라는 의미이며 assetModuleFilename 는 file 들을 bundle링 하여 path의 image 폴더에  저장한다. 



### module 설정

+ module 설정에는 css, file, babel 설정 등이 들어간다.



#### 설치

```bash
npm i -D "@babel/core" "@babel/plugin-syntax-throw-expressions" "@babel/plugin-transform-runtime" "@babel/preset-env" "@babel/preset-react" "babel-loader" "css-loader" "file-loader" "node-sass" "sass-loader" "style-loader"
```



#### 설정

+ webpack.config.js

```javascript
const path = require('path');

module.exports = {
    mode: 'development',
    entry: path.resolve('ucare_frontend/src/index.js'),
    output: {
    	path: path.resolve('ucare_backend/src/main/webapp/assets'),
    	filename: 'js/bundle.js'
    },
    module: {
        rules: [{
            test: /\.(png|gif|jpe?g|svg|ico|tiff?|bmp)$/i,
            type: 'asset/resource'
        }, {
            test: /\.(sa|sc|c)ss$/i,
            use: [
                'style-loader',
                {loader: 'css-loader', options: {modules: true}},
                'sass-loader'
            ]
        }, {
            test: /\.js$/,
            exclude: /node_modules/,
            loader: 'babel-loader',
            options: {
                configFile: path.resolve('ucare_frontend/config/babel.config.json')
            }
        }]
    }
}
```

+ 첫 번째는 /\.(png|gif|jpe?g|svg|ico|tiff?|bmp)$/i 해당 확장자 명으로 된 파일을 로딩 하기 위한 설정이다.
+ 두 번째는 css, scss, sass 에 관한 설정이며 CSS Module 이라는 기술을 사용하여 CSS 클래스가 중첩되는 것을 완벽히 방지하기 위하여 options: {modules: true} 라는 설정을 하였다.
+ 마지막으로 es 6 문법을  es5로 변환 해주기위한 트렌스 파일러인 babel 을 사용 하였다. 이것을 통해 react 를 일반 브라우저에서 실행 시킬 수 있다.



### Devtool Option

+ devtool 은 소스맵을 생성하여 원본 소스랑 맵핑할 수게 있게 설정하는 부분 이다. 

``` javascript
const path = require('path');

module.exports = {
    mode: 'development',
    entry: path.resolve('ucare_frontend/src/index.js'),
    output: {
    	path: path.resolve('ucare_backend/src/main/webapp/assets'),
    	filename: 'js/bundle.js'
    },
    module: {
        rules: [{
            test: /\.(png|gif|jpe?g|svg|ico|tiff?|bmp)$/i,
            type: 'asset/resource'
        }, {
            test: /\.(sa|sc|c)ss$/i,
            use: [
                'style-loader',
                {loader: 'css-loader', options: {modules: true}},
                'sass-loader'
            ]
        }, {
            test: /\.js$/,
            exclude: /node_modules/,
            loader: 'babel-loader',
            options: {
                configFile: path.resolve('ucare_frontend/config/babel.config.json')
            }
        }]
    },
    devtool: "eval-source-map",
}
```

+ 여기서는 devtool 옵션에 eval-source-map 을 사용 하였다.

![캡처](https://user-images.githubusercontent.com/67888402/135216628-5ed9514d-cfcd-4349-a96b-dd8e06381576.PNG)



### DevServer

+ **webpack-dev-server**는 빠른 실시간 리로드 기능을 갖춘 개발 서버로
  + 디스크에 저장되지 않는 메모리 컴파일을 사용하기 때문에 컴파일 속도가 빨라지고
  + webpack.config.js에도 devServer 옵션을 통해 옵션을 지정하여 사용이 가능하다.

#### 설치

``` bash
npm i -D webpack-dev-server
```



#### 설정 

``` javascript
const path = require('path');

module.exports = {
    mode: 'development',
    entry: path.resolve('ucare_frontend/src/index.js'),
    output: {
    	path: path.resolve('ucare_backend/src/main/webapp/assets'),
    	filename: 'js/bundle.js'
    },
    module: {
        rules: [{
            test: /\.(png|gif|jpe?g|svg|ico|tiff?|bmp)$/i,
            type: 'asset/resource'
        }, {
            test: /\.(sa|sc|c)ss$/i,
            use: [
                'style-loader',
                {loader: 'css-loader', options: {modules: true}},
                'sass-loader'
            ]
        }, {
            test: /\.js$/,
            exclude: /node_modules/,
            loader: 'babel-loader',
            options: {
                configFile: path.resolve('ucare_frontend/config/babel.config.json')
            }
        }]
    },
    devtool: "eval-source-map",
    devServer: {
        disableHostCheck: true,
        contentBase: path.resolve('ucare_frontend/public'),
        watchContentBase: true,
        host: "0.0.0.0",
        port: 9999,
        proxy: {
            '/api': 'http://localhost:8080/',
            '/ucare_backend/assets/uploads-images': 'http://localhost:8080/'
        },
        inline: true,
        liveReload: true,
        hot: false,
        compress: true,
        historyApiFallback: true
    }
}
```

+ 각각 요소에 대해 알아보면
  + host: 사용될 호스트를 지정
  + contentBase: 콘텐츠를 제공할 경로지정 (정적파일을 제공하려는 경우에만 필요)
  + compress: 모든 항목에 대해 gzip압축 사용
  + hot: webpack의 HMR 기능 활성화
  + inline: inline 모드 활성화
  + port:  접속 포트 설정
  + open: dev server 구동 후 실행 될 브라우저 열기
  + disableHostCheck: local에서 원격으로 브라우저에서 접속하였을때 개발 보안상 접속 안되는 현상이 발생하는데 disableHostCheck: true 를 사용하면 같은 local 내에서 접속이 가능하다.
  + watchContentBase: 새로운 변경 사항이있을 때 웹 사이트가 업데이트가 되지 않는 현상이 있는데 이것을 해결 하기 위해서 watchContentBase: true 설정을 해준다.
  + proxy:  
    + '/api',  '/ucare_backend/assets/uploads-images'  ( 프록시 요청을 보낼 api의 시작 부분 )
    + 'http://localhost:8080/' (프록시 요청을 보낼 서버의 주소)
  + liveReload: 코드 변경이 일어나면 자동으로 reload 실행
  + historyApiFallback:  HTML5의 History API를 사용하는 경우에 설정해놓은 url 이외의 url 경로로 접근했을때 404 responses를 받게 되는데 이때도 index.html을 서빙할지 결정하는 옵션.

