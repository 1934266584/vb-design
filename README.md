<div align="center">
<h1>vb-design</h1>
<p>基于pnpm创建的Monorepo多包项目</p>
</div>

## 初始化
```
pnpm install
```

## 开发

```
cd examples/demo

pnpm run dev
```

## 打包
```
pnpm run build
```

## 发布
```
pnpm run release:only
```


# Monorepo概念
### 随着各种技术的发展和超级应用的出现，人们开始考虑怎么才能将所有的小应用都集成在一个大项目中，特别是在这些项目互相影响时，在实现过程中，工程师们最关注的两点是：项目功能分离 和 避免重复代码。
### 如果将每个功能作为独立的项目打包，随着业务的发展，项目会越来越多，根本没法管理，项目与项目之间的协作也会越来越困难，所以Monorepo的概念并产生了。
在Monorepo中我们可以在一个项目下进行功能拆分，他们互相独立不影响，但是又可以通过引用来达到互相协助。

# Monorepo的优缺点

## 优点：

### 简化依赖的管理。
### 跨组合作更加方便。
### 代码复用简单。

## 缺点：

### 项目构建时间过长。
### 版本信息杂糅不清晰。

# 项目工程的搭建
## 技术选型
### 基于pnpm的Monorepo工程，项目打包工具vite、gulp，使用sass处理样式。
### Vue组件写法会支持Jsx和template的方式。项目支持Typescript。
### lint规范的接入，prettier的格式化统一，husky的卡点校验。
### 组件单元测试使用vitest+happy-dom。

基于以上的技术开始搭建我们的项目。
### 项目的大概结构
```
|—— config        //放置一些脚本
|—— examples      //存放演示包
    |—— demo
    |—— taro-demo 
    ...
|—— packages      //存放npm库的包
    |—— hooks
    |—— icon
    |—— ui-h5
    |—— ui-taro
    ...
.eslintignore
.eslintrc.js
.gitignore
.lintstagedrc.cjs
.npmrc
.prettierrc.cjs
package.json
pnpm-workspace.yaml
README.md
tsconfig.root.json
...
```

## 项目配置
### Monerepo工程的起步

pnpm搭建Monorepo是非常简单的，只需要我们配置pnpm-workspace.yaml文件即可。具体的配置可参考pnpm-workspace.yaml | pnpm

### lint、prettier、eslint的接入
lint、prettier、eslint的配置大部人应该都很熟练了，在这我就不一一贴代码说明了。还不清楚的小伙伴可参考我的代码或者找篇相关的教程自己跟着试试。

## 统一开发环境

## 开发环境的统一，主要是统一Node版本和pnpm，我们可以通过在package.json中配置一些字段来统一开发环境。
### 1、限制Node版本和pnpm
通过配置volta和engines限制Node和pnpm的版本
```
//package.json

"volta": {
    "node": "16.13.0"
},
"engines": {
    "node": "16.13.0",
    "pnpm": ">=6"
}
```

### 2、限制项目只能通过pnpm初始化依赖
```
//package.json

"scripts": {
    "preinstall": "npx only-allow pnpm",
}
```
### packages子包搭建
对于子包的搭建，不会详细地一一讲解，需要深入了解的可以自行到源代码里看。
### ui-h5搭建
目录结构设计
```
//ui-h5

components                  //组件目录
    |—— Button
        |—— demo            //demo演示存放目录
            |—— base.vue
            ...
        |—— index.md        //组件使用文档
        |—— index.scss      //组件样式
        |—— index.ts        //vue组件
        |—— index.taro.ts   //taro组件
    |—— Icon
    ...
style          //公共样式存放
    |—— index.scss
    ...
ui.h5.ts       //vue组件对外暴露
ui.taro.ts     //taro组件对外暴露
```

为了开发方便，把vue端、taro端的组件都放在该包下，以及examples需要的演示文档和demo也放在该包下。至于各端写好的组件、demo、演示文档是怎么使用，后续会说明。

### 构建产物
```
// dist
components                    //单个组件
    |—— Button
        |—— index.scss
        |—— index.css
        |—— index.js
        |—— index.taro.js
    |—— Icon
    ...
style 
    |—— index.scss
    ...
types
    |—— 
    ...
style.css
vb-ui.es.js
vu-ui.umd.js
vb-ui.taro.es.js
vb-ui.taro.umd.js
```

### ui-taro搭建
该包其实没有做更多的事情，只是初始化之后把package.json做了相关配置。最重要的地方是在根目录下的package.json配置了脚本，在ui-h5包构建之后，通过脚本copy了该包需要的东西。

脚本文件都在根目录config里

### icon搭建
在ui-h5中，Icon组件的设计支持了iconfont和svg的方式，这也是参考了element-ui的Icon组件设计。所以该包主要是处理svg图标，把图标转化成vue组件统一向外暴露的过程。

另外，我并不会用设计软件，没有svg可用，所以借用了字节arco-design的图标Arco Design Icons – Figma

### hooks搭建

该包主要是一些公共方法的包，目前也没有更多的想法，所以也只是先放着。

### examples浏览器端演示包搭建
因为搭建的是移动端组件库，所以演示包需要有两个入口，H5端和PC端。整个搭建的过程大致是：

通过vite-plugin-md解析md文件。

通过vite-plugin-pages和vite-plugin-vue-layouts管理路由，页面的存放路径是在ui-h5包下。

vite-plugin-pages和vite-plugin-vue-layouts的配置
```
//vite.config.js

Pages({
  dirs: [
    {
      dir: resolve(__dirname, '../../packages/ui-h5/components'),
      baseRoute: 'component',
    },
  ],
  exclude: ['**/components/*.vue'],
  extensions: ['vue', 'md'],
}),
Layout({
  layoutsDirs: 'src/layouts',
  defaultLayout: 'preview',
})
```
### 项目的构建与npm包发布
这么多的子包，打包构建以及推送到npm是不是需要到每个子包下执行完打包和执推送的命令？针对这个问题pnpm官方是有解决方案的：

首先所有的子包都定义个build命令来执行当前包的所有打包构建事情，最后项目根目录package.json的配置如下：
```
//package.json

"scripts": {
    "build": "pnpm --filter './packages/**' run build && pnpm run build:taro",  //执行packages下所有子包的build方法
    "release": "pnpm run build && pnpm run release:only",
    "release:only": "changeset publish --tag=beta --access=publish",            //发布所有子包
    "build:taro": "node ./config/build-taro.js",
    "build:demo": "pnpm --filter './examples/**' run build"
}

```
上方build这个地方的配置pnpm官方有很详细的说明。再就是关于npm publish的，主要是通过@changesets/cli这个cli工具去解决。

changeset publish 只是一个很纯净的发包命令，手动提升/修改版本后再 changeset publish他会将所有包都publish一次。

写到这，顺便再给大家说下这个项目的代码发布流吧。

版本号是手动修改的。

通过打tag的方式会触发workflows去执行打包构建，然后publish和部署演示的demo。

很多开源的项目通过changeset-bot + changesets/action + @changesets/cli能玩出各种各样的工作流。

### 关于单元测试
单测在开源项目里是不可缺少的存在，虽然我不一定会去写单测😀，但是该有的东西还是得搭起来。

Vitest是基于vite的原生快速单元测试，完全兼容Jest的Api，还能共用vite的配置。所以针对当前项目使用Vitest是最快接入单元测试的方式，但是很遗憾，针对小程序端还没有更好的接入单元测试方案。
