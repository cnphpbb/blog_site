+++
title = "今天学到了 Jest 单元测试"
draft = false
date = "2022-08-10T07:56:00+08:00"
menu = "main"
Categories = ["vue", "Jest", "Guide"]
Description = "Jest是一款优雅、简洁的 JavaScript 测试框架。支持 Babel、TypeScript、Node、React、Angular、Vue 等诸多框架！"
Tags = ["jest", "guide", "solutions", "JavaScript", "vue", "开源"]
+++

**Jest 是一款优雅、简洁的 JavaScript 测试框架。**

**Jest 支持 Babel、TypeScript、Node、React、Angular、Vue 等诸多框架！**

有幸参与了一个微信小程序项目前端开发，此项目使用 uniapp vue2开发。 接触了一次jest单元测试，写下笔记有益于以后继续学习和回顾。

为了保证项目中代码方法的正确性，必然就需要去做单元测试。选择了 Jest 测试框架。

### jest单元测试NPM包

在项目目录下执行如下指令

```shell
yarn add --dev jest \
  babel-jest \
  @babel/core \
  @babel/preset-env \
  @vue/vue2-jest \
  @vue/test-utils \
  jsdom \
  jest-environment-jsdom
```

需要注意下 Vue2 版本中npm包的版本

```shell
"@babel/preset-env": "^7.18.10",
"@vue/test-utils": "^1.2.0"
"@vue/test-utils": "^1.3.0",
"@vue/vue2-jest": "^28.0.1",
"babel-jest": "^28.1.3",
"jest": "^28.1.3",
"jest-environment-jsdom": "^28.1.3",
"jsdom": "^20.0.0",
```

以上内容直接添加到 `package.json` 文件 `devDependencies` 节点中， 执行 `yarn install` 即可。 :construction: 

### jest单元测试配置

- jest.config.js

```javascript
module.exports = {
    testEnvironment: "jsdom", //用于测试的测试环境
    testMatch: [ //检测测试文件
        '**/test/units/**/?(*.)+(spec|test).[jt]s?(x)',
    ],
    moduleNameMapper: { //映射路径到模块名称 完成`@/`的路径映射
        "^@/(.*)$": "<rootDir>/$1"
    },
    transform: {    //转换器路径的映射
        "^.+\.js$": "babel-jest",
        "^.+\.vue$": "@vue/vue2-jest"
    },
    moduleFileExtensions: [ //使用的文件扩展名
        "js",
        "vue"
    ],
    collectCoverage: true,  //收集测试时的覆盖率信息
    coverageDirectory: '<rootDir>/test/coverage',   //输出其覆盖范围文件的目录
    testPathIgnorePatterns: [   //忽略的目录
        "<rootDir>/node_modules/",
    ],
    transformIgnorePatterns: [  //转换器忽略的目录
        "<rootDir>/node_modules/",
    ]
};
```

在 `jest.config.js` 中 `testMatch`节点配置了测试文件的目录 `test/units`。 在研发时要在这里写入单元测试的文件。 :fire: 

- .babelrc

```json5
{
    presets: [
        [
            "@babel/preset-env",
            {
                "targets": {
                    "node": "current"
                }
            }
        ]
    ]
}
```

- package.json

在 `package.json` 中 `scripts` 节点，添加如下配置。

```json5
{
  "scripts": {
    ......
    "test:mp-weixin": "cross-env UNI_PLATFORM=mp-weixin jest -i",
    ......
  },
}
```

在进行单元测试时，在终端中就可以看到执行结果和覆盖率报告。

```shell
yarn test:mp-weixin
```

### 单元测试常用API

- [describe](https://www.jestjs.cn/docs/api#describename-fn)
```javascript
const myBeverage = {
  delicious: true,
  sour: false,
};

describe('my beverage', () => {
  test('is delicious', () => {
    expect(myBeverage.delicious).toBeTruthy();
  });

  test('is not sour', () => {
    expect(myBeverage.sour).toBeFalsy();
  });
});
```

- [test](https://www.jestjs.cn/docs/api#testname-fn-timeout)
```javascript
test('has lemon in it', () => {
  return fetchBeverageList().then(list => {
    expect(list).toContain('lemon');
  });
});
```

参考网址：
- [Jest 学习04 - DOM 测试、快照测试、测试覆盖率](https://blog.csdn.net/u012961419/article/details/123638655)
- [在uniapp框架下实现用jest进行h5环境的自动测试](https://juejin.cn/post/6946580423954661384)
- [Jest中文文档](https://www.jestjs.cn/)