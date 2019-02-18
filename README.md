## [博客地址](http://mingshined.github.io)

## 技术栈概览
1. 前台：Umi(路由) + Antd(视图) + TypeScript(增加项目可维护性以及规范性)
2. 后台：Umi(路由) + Antd(视图) + TypeScript(增加项目可维护性以及规范性) + Rematch(数据管理)
3. 服务：Egg.js(基于koa的下一代企业级应用框架) + MongoDB

## 搭建思路
启动本地Mongo以及Egg服务，随后启动博客后台，编辑文章，将文章数据保存在本地Mongo数据库。将本地Mongo数据导出为JSON，前台页面用静态JSON渲染，通过webpack将前台工程打包成静态页，所有静态资源托管于github page。通过github page提供的映射域名即可访问静态页index.html。
### 优点
1. 不需要购买域名以及服务器
2. 可以通过issue参与文章评论互动
3. 整个搭建流程有利于提高对项目的理解与认知

### 缺点
1. github服务器响应速度慢，导致页面响应速度慢
2. 域名晦涩难记
3. 无法实现登录注册留言等交互功能，缺乏互动性

### 总结
该静态博客搭建适合有一定基础，需要项目练手的初级前端开发工程师。博客搭建省时省力，堪称居家必备良品。一方面有利用提升自身综合实力，另一方面不用为域名和服务器的到期而堪忧。当文章质量得到保证时，有利于提高github知名度。

## 搭建流程
### 一、 前台
> UI设计

由于博主本人对设计领域着实没有建树，所以该博客的UI设计借鉴了[一位小姐姐的博客UI](https://cherryblog.site)
> 路由设计

![](https://user-gold-cdn.xitu.io/2019/2/14/168ec0dd14155332?w=1307&h=250&f=jpeg&s=34680)

> 搭建工程目录

```
├── config
│   ├── config.js
│   └── proxyConfig.js
├── package.json
├── src
│   ├── assets
│   ├── common
│   ├── components
│   ├── global.less
│   ├── layouts
│   ├── pages
│   ├── plugins
│   │   └── rematchPlugin
│   │       ├── createPlugin.d.ts
│   │       ├── createPlugin.js
│   │       ├── index.js
│   │       ├── runtime.ts
│   │       ├── template
│   │       │   ├── RematchContainer.tsx
│   │       │   └── Store.ts
│   │       └── ulits.js
│   ├── services
│   ├── store.ts
│   ├── types
│   └── utils
├── tsconfig.json
├── tslint.json
├── typings.d.ts
```

> 响应式设计

博客响应式设计主要采用了Antd的栅格化系统以及enquire-js来判断浏览设备窗口大小

**封装工具类：**
```
/**
 * @name 判断浏览设备
 */
export function distingIsMobile<T>(): boolean {
  let isMobile = false;
  enquireScreen(value => {
    isMobile = value;
  });
  return isMobile;
}
```

> 技术难点

* **markdown语法的渲染**。博客采用react-markdown来渲染markdown语法。但是依然存在未解决的问题：

 code代码块语法高亮无法实现

  *博主目前依旧无法解决上诉两个问题。如在读的大神们对于上述问题有解决办法，请通过issue或者博主的联系方式联系博主，感激不尽！*

* **动画的设计**。博客采用Antd Animotion来实现动画效果。该博客的动画花费了博主大量的时间去设计和实现。
* **简历模块采用react-fullpage来实现单页滚动效果**。其中，第三板块的技术栈详情弹窗采用了antd的modal，弹窗实例化后，改变了body的overflow值，导致单页滚动失效。博主通过按钮的点击事件手动设置body的overflow样式得以解决该问题。
* **md引用语法不生效**。通过手动设置blockquote元素的css样式。
  ```
    blockquote {
    padding: 0 1rem;
    margin-left: 0;
    color: #819198;
    border-left: .3rem solid #dce6f0;
  }
  ```

### 二、 后台
> 模块设计

![](https://user-gold-cdn.xitu.io/2019/2/14/168ec0dd16de1126?w=1801&h=1080&f=jpeg&s=93027)

> 最终效果

![](https://user-gold-cdn.xitu.io/2019/2/14/168ec0dd175e07b3?w=1972&h=1080&f=jpeg&s=82418)

> 代理配置

```
const devPath = 'http://127.0.0.1:7001'; // 代理地址指向本地egg服务运行地址
export default {
  '/api/': {
    target: devPath,
    changeOrigin: false
  }
};
```


> 技术难点

* MarkDown编辑器选型

  博主最终采用SimpleMDE。

 **效果图：**


![](https://user-gold-cdn.xitu.io/2019/2/14/168ec0dd1719fff2?w=1747&h=1080&f=jpeg&s=87612)

  **实例化：**
```
  componentDidMount() {
    this.renderContEditorNode();  // 实例化内容编辑器
    this.renderSumEditorNode(); // 实例化摘要编辑器
  }
  renderContEditorNode() {
    this.contEditNode = new SimpleMDE({
      element: document.getElementById('contEditor').childElementCount,
      ...simpleMdConfig
    });
  }
  renderSumEditorNode() {
    this.sumEditNode = new SimpleMDE({
      element: document.getElementById('sumEditor').childElementCount,
      ...simpleMdConfig
    });
  }
```
  **编辑器simpleMdConfig配置：**
  ```
 /**
 * @name SimpleMDE配置
 */
export const simpleMdConfig = {
  autofocus: true,
  autosave: true,
  shortcuts: {
    drawTable: 'Cmd-Alt-T'
  },
  showIcons: ['code', 'table'],
  tabSize: 4,
  placeholder: '在这里编辑',
  toolbar: [
    'bold',
    'italic',
    'strikethrough',
    'heading',
    'code',
    'quote',
    'unordered-list',
    'ordered-list',
    'clean-block',
    'link',
    'image',
    'table',
    'horizontal-rule',
    'preview',
    'side-by-side',
    'fullscreen',
    'guide'
  ],
  previewRender(plainText) {
    return marked(plainText, {
      renderer: new marked.Renderer(),
      gfm: true,
      pedantic: false,
      sanitize: false,
      tables: true,
      breaks: true,
      smartLists: true,
      smartypants: true,
      highlight(code) {
        return highlight.highlightAuto(code).value; // 采用highlight.js实现代码块语法高亮
      }
    });
  }
};
  ```
 **遇见的问题：**
 ```
   {
      key: 'summary',
      label: '摘要 - 内容',
      node: (
        <textarea
          style={{ maxHeight: '300px', overflow: 'auto' }}
          id="sumEditor"
        />
      )
    },
    {
      key: 'content',
      node: (
        <textarea
          id="contEditor"
          style={{ maxHeight: '300px', overflow: 'auto' }}
        />
      )
    },
```

  由于我采用了自己封装的表单组件，组件内部存在一定的异步渲染，因此在页面渲染结束时，无法找到实例化的textarea元素，因此报错。

  **解决办法：**

  在render函数内写入两个隐藏的textarea元素，保障组件实例化后能够找到对应的dom元素，从而成功实例化编辑器。

  ```
 <textarea id="contEditor" style={{ display: 'none', maxHeight: '300px', overflow: 'auto' }} />
 <textarea id="sumEditor" style={{ display: 'none', maxHeight: '300px', overflow: 'auto' }} />
 ```


 * 图片资源的存放
   博客选用[ipic](https://toolinbox.net/iPic/)第三方软件来存放图片资源库。
   ![](https://user-gold-cdn.xitu.io/2019/2/14/168ec0dd17069220?w=690&h=388&f=gif&s=852608)
   通过简单的拖拽就可以实现图片的上传，上传成功后返回图片地址。

### 三、服务
> 通过官方脚手架叫快速生成

```
$ npm i egg-init -g
$ egg-init egg-example --type=simple
$ cd egg-example
$ npm i
```

> 修改配置

```
'use strict';

module.exports = appInfo => {
  const config = (exports = {});

  // use for cookie sign key, should change to your own and keep security
  config.keys = appInfo.name + '_1544961945990_6105';

  // add your config here
  config.middleware = [];

  /**
   * @name mongo配置,通过egg-mongoose连接mongo
   */
  config.mongoose = {
    url: 'mongodb://127.0.0.1:27017/MingShined',
    options: {
      useMongoClient: true,
      autoReconnect: true,
      reconnectTries: Number.MAX_VALUE,
      bufferMaxEntries: 0
    }
  };

  /**
   * @name 关闭csrf
   */
  config.security = {
    csrf: {
      enable: false
    }
  };

  return config;
};

```

启动项目
```
$ npm run dev
$ open localhost:7001
```

> 数据库设计

![](https://user-gold-cdn.xitu.io/2019/2/14/168ec0dd17b8cac8?w=2202&h=1014&f=jpeg&s=155489)

> restfulApi风格的路由设计

```
'use strict';
module.exports = app => {
  const { router, controller } = app;
  /**
   * @name 新增文章
   */
  router.post('/api/article', controller.article.article.createArticle);
  /**
   * @name 导出文章
   */
  router.get(
    '/api/article/download',
    controller.article.article.downloadArticle
  );
  /**
   * @name 获取文章列表
   */
  router.get('/api/article', controller.article.article.queryArticleList);
  /**
   * @name 编辑文章
   */
  router.put('/api/article', controller.article.article.updateArticle);

  /**
   * @name 获取文章详情
   */
  router.get('/api/article/:id', controller.article.article.findArticle);
  /**
   * @name 获取文章详情
   */
  router.delete('/api/article/:id', controller.article.article.deleteArticle);
  /**
   * @name 获取关于我
   */
  router.get('/api/about', controller.article.article.getAbout);
  /**
   * @name 保存关于我
   */
  router.post('/api/about', controller.article.article.createAbout);
  /**
   * @name 下载关于我
   */
  router.get('/api/about/download', controller.article.article.downloadAbout);
};

```

> 技术难点

* 导出本地mongo数据为JSON
```
  /**
   * @name 下载文章
   */
  async downloadArticle() {
    const json = await this.service.article.queryArticleList();
    this.ctx.attachment('article.json');
    this.ctx.set('Content-Type', 'application/json');
    this.ctx.body = JSON.stringify(json);
  }
```

## 总结
总的来说，纯手撸一个静态博客还是挺有成就感。整个过程遇到种种问题，在这个过程中，不断总结进步，提升较大。搭建成功后，也为自己开拓了一个属于自己的平台，通用的md语法也可以同步到知乎，掘金等社区。对博主以及博主的文章有什么意见建议，欢迎联系博主。

* QQ:996578843