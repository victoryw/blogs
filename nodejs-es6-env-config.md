# 为NodeJs配置Babel
## TL;DR
原有的开发环境为：

- Node.js 10.8.0
- Mocha & Chai
- istanbul

希望通过添加Babel来更好的支持ES6的特性
做了以下的事情：

- 如何配置Babel 支持编译

    ``` bash
    yarn add -D babel-preset-env babel-cli

    .babelrc
    {
    "presets": [
        ["env", {
        "targets": {
            "node": "current"
        }
        }]
    ],
    "plugins": ["transform-runtime"]
    }
    ```

- 如何配置mocha

    ``` bash
    yarn add -D babel-register
    pacakge.json
    "script":{
        “test”:"mocha --opts ./spec/mocha.opts --recursive './spec/**/*.spec.js'"
    }

    mocha.opts
    # mocha.opts
    --require babel-register
    --reporter dot
    --ui bdd
    ```

- 如何配置istanbul

    [es2015](https://istanbul.js.org/docs/tutorials/es2015/)  
    `yarn add -D babel-plugin-istanbul cross-env mocha chai nyc`

    ``` bash
    package.json
    "nyc": {
    "require": [
      "babel-register"
    ],
    "reporter": [
      "lcov",
      "text"
    ],
    "sourceMap": false,
    "instrument": false,
    "exclude": "./spec/**/*.js"
    }
    ```