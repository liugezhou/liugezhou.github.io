---
layout: post
title: Week33-组件平台开发
author: liugezhou
date: 2021-08-26
category: 
    Web架构之脚手架
tags:
    Web架构之脚手架
---
<a name="HSEoC"></a>
### 第一章 本周导学

---

<a name="q5Q62"></a>
#### 1-1 本章介绍
> - 组件平台
> - 组件预览
> - 组件README

<a name="MG4nm"></a>
### 
<a name="ztC8A"></a>
### 第二章 组件平台架构设计和技术选型

---

<a name="algMo"></a>
#### 2-1 组件平台架构设计
[点击查看【processon】](https://www.processon.com/embed/61298d22637689579630027a)
<a name="gTsNZ"></a>
#### 2-2 组件平台技术选型和框架搭建
  umi-component-test改项目代码提交至：[https://github.com/liugezhou/umi-component-test](https://github.com/liugezhou/umi-component-test)

1. **初始化UmiJS**
> - mkdir umi-component-dev
> - cd umi-component-dev
> - yarn create @umijs/umi-app
> - yarn install
> - yarn start

<a name="UykDt"></a>
#####  2. 新建页面
> npx umi g page detail

3. **配置路由**
> **.umirc.ts新建页面路由**
> routes: [
>  { path:'/', component:'@/pages/index' },
>  { path:'/nice', component:'@/pages/detail' },
>  ],
> yarn start启动项目后，访问 http://192.168.1.3:8000/nice即可看到最新的页面

<a name="jJsRL"></a>
#####  4. 安装 Ant Design
> yarn add antd

5. **使用**
```html
import { Button } from "antd";

<Button type="primary">Button</Button>
```
<a name="mbeWU"></a>
### 
<a name="JWwJv"></a>
### 第三章 组件平台基础功能开发

---

<a name="CmlG4"></a>
#### 
3-1 umi 项目全局入口文件+国际化开发

> - 运行时配置:约定src/app.tsx为运行时配置,该配置文件下可以做一些全局性的操作
> - 国际化：文档-插件-@umijs/plugin-locale
>    - **mkdir：**src/locales/zh-CN.ts | en-US.ts，配置WELCOME_TO_UMI_WORLD字段内容
>    - .umirc.ts中配置国际化[代码如下所示]
>    - src/app.tsx中引入国际化[代码如下所示]

```javascript
// src/umirc.ts
locale: {
    default: 'zh-CN',
    antd: false,
    title: false,
    baseNavigator: true,
    baseSeparator: '-',
  },
    
 // src/app.tsx
  import 'antd/dist/antd.css';
  import { getLocale,setLocale } from 'umi';
  import qs from 'qs';
  const { search } = window.location;
  const { locale = 'zh-CN' } = qs.parse(search, { ignoreQueryPrefix: true });
  setLocale(locale,false)
  
  
  // src/pages/index.tsx
  import { useIntl } from 'umi';
export default function IndexPage() {
  const init = useIntl()
  const msg = init.formatMessage({
      id:'WELCOME_TO_UMI_WORLD',
      defaultMessage:'你好，牛逼的前端架构师！'
  },{
    name:'liugezhou'
  })
  console.log(msg)
}
```
> 最后：yarn start启动文件，访问查看控制台：http://localhost:8000/?locale=en-US



<a name="hRNXK"></a>
#### 3-2 组件平台功能展示 + 页头页脚开发

> umijs支持layout引入，于是我们在开发页头页脚的时候，页面页头与页脚是在各个页面都存在的，于是我们可以将页面不同的地方以layout的形式注入

```javascript
// src/layouts/index.js

import styles from './index.less'

function BasicLayout(props){
	return (
  <div className={styles.normal}>
    <div className={styles.title}>{slogan}</div>
		{props.children}
  	<div className={styles.footer}>{copyright}</div>
  </div>
  )
}
export default BasicLayout

//src/layouts/index.less
@import '~antd/dist/antd';

.normal{
    text-align:center
}

.title{
    font-size:15px;
    font-weight: normal;
    background: @primary-color;
    padding: 10px 0;
    color: white;
    margin:0;
}

.footer {
    font-size: 12px;
    font-weight: normal;
    background: @primary-color;
    padding: 10px 0;
    color: white;
    margin: 0;
}
  
// .umijsrc.js
import { defineConfig } from 'umi';

export default defineConfig({
  nodeModulesTransform: {
    type: 'none',
  },
  routes: [
    {
      path: '/',
      component: '@/layouts/index',
      routes: [
        {
          path: '/',
          component: '@/pages/index'
        },
        {
          path: '/detail',
          component: '@/pages/detail'
        }
      ]
    }
  ],
  fastRefresh: {},
  locale: {
    default: 'zh-CN',
    antd: false,
    title: false,
    baseNavigator: true,
    baseSeparator: '-',
  }
});

```

<a name="By9Ts"></a>
#### 3-3 组件平台动态配置 API 开发
> 我们的页面与页脚内容需要从接口获取，因此，本节内容为在 cloudscope-cli-server服务中去编写接口代码。
> 本周相关代码提交至：[cloudscope-cli/server/lesson33](https://github.com/liugezhou/cloudscope-cli-server/tree/lesson32)


**3-3-1 添加路由 **
```javascript
// app/route.js
'use strict';

module.exports = app => {
  const { router, controller } = app;
 
  router.resources('componentSite', '/api/v1/componentSite', controller.v1.componentSite);

};

```
**3-3-2 API编写**
```javascript
// app/controller/v1/ComponentSite.js
'use strict'
const Controller = require('egg').Controller;
const mongo = require('../../utils/mongo')

class ComponentSiteController extends Controller{
  asynx index(){
  	const { ctx } = this;
		const data = await mongo.query('componentSite);
		  if(data && data.length>0){
    		ctx.body = data[0]
     	}else{
      	ctx.body = []
      }
  }                                
}
```
> - 需要在mongoDB中新建集合 componentSite,添加页面页脚数据
> - 启动项目，浏览器输入地址，测试访问结果


<a name="MFxMY"></a>
#### 3-4 前端动态配置 API 接入
> 代码无分支提交至：[umi-component-test](https://github.com/liugezhou/umi-component-test)

> 上一节我们开发完毕api接口后，在前端请求该接口
> - 首先需要install axios
> - 接着需要封装request请求。

```javascript
// src/layouts/util/reques.js

import axios from 'axios';
const BASE_URl = 'http://liugezhou.com:7001'
const service = axios.create({
	baseURl:BASE_URL,
  timeout:5000
})

service.interceptor.request.use({
	config => {
  	return config;
	}
  error => {
  	return new Promise.reject(error)                              
	}
  
  service.interceptor.response.use({
    response => {
    return new response.data;
  }
	error => {
		return new Promise.reject(error)                              
	}
})

export default request;

//src/layouts/utils/service.js

import request from './request'

export function getSiteInfo(){
	return request({
  	url:'/api/v1/componentSite'
  })
}

export default {}
```
> - 最后，在首页中去调用该接口，且赋值为页头与页脚

```javascript
// src/layouts/index.js

import styles from './index.less';
import {getSiteInfo } from '../utils/service';
import { useState, useEffect } from 'react'';

function BasicLayout(props){
		const [init,setInit] = useState(false);
  	const [slogan,setSlogan] = useState('');
  	const [copyright,setCopyright ] = useState('');
  	
  	useEffect( () =>{
    	 if (!init) {
         setInit(true);
         getSiteInfo().then(data => {
           setSlogan(data.slogan);
           setCopyright(data.copyright);
         }).catch(err => {
           console.log(err);
           setSlogan('');
           setCopyright('');
         });
       }
    },[init])
  
  return (
      	 <div className={styles.normal}>
            <div className={styles.title}>{slogan}</div>
            {props.children}
            <div className={styles.footer}>{copyright}</div>
        </div>
      )
}
 export default BasiLayout;
```

<a name="cr6u6"></a>
### 第四章 组件平台组件列表页面开发

---

<a name="R1lR7"></a>
#### 4-1 组件列表 API 开发
> 本周相关代码提交至：[cloudscope-cli/server/lesson33](https://github.com/liugezhou/cloudscope-cli-server/tree/lesson32)

```javascript
// app/controller/v1/Components.js

async index(){
  const { ctx, app } = this;
  const { name } = ctx.query;
  const andWhere = name ? `AND c.name LIKE '%${name}%'` : '';
  const sql = `SELECT c.id, c.name, c.classname, c.description, c.npm_name, c.npm_version, c.git_type, c.git_remote, c.git_owner, c.git_login, c.create_dt, c.update_dt, v.version, v.build_path, v.example_path, v.example_list
FROM component AS c
LEFT JOIN version AS v ON c.id = v.component_id
WHERE c.status = 1 AND v.status = 1 ${andWhere}
ORDER BY c.create_dt, v.version DESC`;
  const result = await app.mysql.query(sql);
  const components = [];
  result.forEach(component => {
    let hasComponent = components.find(item => item.id === component.id);
    if (!hasComponent) {
      hasComponent = {
        ...component,
      };
      delete hasComponent.version;
      delete hasComponent.build_path;
      delete hasComponent.example_path;
      delete hasComponent.example_list;
      hasComponent.versions = [];
      components.push(hasComponent);
      hasComponent.versions.push({
        version: component.version,
        build_path: component.build_path,
        example_path: component.example_path,
        example_list: component.example_list,
      });
    } else {
      hasComponent.versions.push({
        version: component.version,
        build_path: component.build_path,
        example_path: component.example_path,
        example_list: component.example_list,
      });
    }
  });
  ctx.body = components;
}
```
<a name="nM4NP"></a>
#### 4-2 测试组件数据创建
> - 上一节我们获取的组件列表数据为一条，本节首先再去创建几条测试数据。
> - 在此之前，先去更改之前的组件模版 cloudscope-cli-components，packahe.json的配置：publishConfig为access：true 和 build:demo
>    - 外层package.json 增加一个版本号
>    - template内package.json 配置 publishConfig
>    - template内package.json 配置‘build:demo’:"npm install && npm run build && cd examples && npm install && npm run build"
>    - npm publish
>    - mongodb数据库中将版本号升级
> 
> - mkdir cloudscope-cli_component-test3
> - cd cloudscope-cli_component-test3
> - cloudscope-cli init -tp /Users/liumingzhou/Documents/imoocCourse/Web前端架构师/cloudscope-cli/commands/init/  【这里需要注意的是，由于我本读安装且默认设置了node的版本为14，而之前开发的本地脚手架为12.16，因此支持以上代码需要更换node版本】
>    - @cloudscope-cli/components-test2
>    - 1.0.0
>    - 通用的Vue3组件库模版
>    - components test2
> - code .
> - npm run build:demo
> - cloudscope-cli publish -tp /Users/liumingzhou/Documents/imoocCourse/Web前端架构师/cloudscope-cli/commands/publish/  [不加prod属性，不用关心OSS与NPM发布]
> 
> 检查：
> - git remote -v  【查看远程仓库信息】
> - 查看mysql数据库插入信息
> 
> - 如果添加了 --prod属性
> - 查看npm发布组件信息
> - 查看云OSS上传信息

<a name="CFZP7"></a>
#### 
<a name="tTLK9"></a>
#### 4-3 组件列表页面开发
<a name="QLk1x"></a>
#### 4-4 组件卡片面板开发
<a name="Bht6s"></a>
#### 4-5 搜索框组件开发+模糊搜索API开发
> 这三节内容为组件首页列表的umi项目代码开发，包括布局、请求、点击事件等功能，代码分类为：国际化配置、工具类、业务代码，其中核心内容为业务代码，主要是使用UI库ant-design-react和umi以及react的一些用法。
> 
> - **国际化配置**

```javascript
// src/locales/en-US.js
export default {
    WELCOME_TO_UMI_WORLD: "{name}, welcome to umi's world",
    INDEX_SEARCH: 'Search',
    INDEX_PLACEHOLDER: 'Component Name',
  };
// src/locales/zh-CN.js
export default {
  WELCOME_TO_UMI_WORLD: '{name}，欢迎光临umi的世界',
  INDEX_SEARCH: '搜索',
  INDEX_PLACEHOLDER: '组件名称',
};
```
> - **工具类**

```javascript
// src/utils/index.js

export function formatName(name) {
  let _name = name;
  if (_name && _name.startsWith('@') && _name.indexOf('/') > 0) {
    // @cloudscope-cli/component-test ->
    // @cloudscope-cli_component-test
    const nameArray = _name.split('/');
    _name = nameArray.join('_').replace('@', '');
  }
  return _name;
}

export default {};
// src/utils/request.js
import axios from 'axios'

const BASE_URL = 'http://liugezhou.com:7001'

const service = axios.create({
    baseURL: BASE_URL,
    timeout: 5000
})

service.interceptors.request.use(
    config => {
        return config
    },
    error => {
        return Promise.reject(error)
    }
)

service.interceptors.response.use(
    response => {
        return response.data
    },
    error => {
        return Promise.reject(error)
    }
);

export default service;

//src/utils/service.js
import request from './request';

export function getSiteInfo() {
  return request({
    url: '/api/v1/componentSite',
  });
}

export function getComponentList(params) {
  return request({
    url: '/api/v1/components',
    params,
  });
}

export default {};

//src/utils/git.js
import { formatName } from '../utils';

export function getGitUrl(item) {
  let name = item.classname;
  if (name.startsWith('@') && name.indexOf('/') > 0) {
    const nameArray = name.split('/');
    name = nameArray.join('_').replace('@', '');
  }
  return `https://${item.git_type}.com/${item.git_login}/${name}`;
}

/**
 * 获取组件预览链接
 * @param name      组件名称
 * @param version   组件版本
 * @param path      组件预览文件路径
 * @param file      组件预览文件名称
 */
export function getPreviewUrl({ name, version, path, file }) {
  // 上传至OSS再来配置
  return `https:// /${formatName(name)}@${version}/${path}/${file}`;
}

export default {};
```
> - **业务代码 ✨✨✨✨✨**

```javascript
// src/pages/index.jsx
import styles from './index.less';
import { Divider, Row, Col, Card, Input } from 'antd';
import { EditOutlined, EllipsisOutlined, EyeOutlined } from '@ant-design/icons';
import { useIntl,history } from 'umi';
import { getGitUrl,getPreviewUrl } from "../utils/git";
import { getComponentList } from '../utils/service';
import { useState, useEffect } from "react";

const { Meta } = Card;
const { Search } = Input;

export default function IndexPage() {
  const [init, setInit] = useState(false);
  const [list, setList] = useState([]);
  const [name, setName] = useState(null);
  const intl = useIntl();

  useEffect(() => {
    if (!init) {
      setInit(true);
      getComponentList({ name }).then(data => {
        console.log(data);
        setList(data);
      }).catch(err => {
        console.error(err);
        setList([]);
      });
    }
  }, [init, name]);

  function getAvatar(item) {
    if (item.git_type === 'gitee') {
      return {
        img: 'https://gitee.com/static/images/logo-black.svg',
        style: { height: '20px', cursor: 'pointer' },
      };
    } else {
      return {
        img: 'https://www.youbaobao.xyz/arch/img/github.jpeg',
        style: { height: '40px', cursor: 'pointer' },
      };
    }
  }
  function getLastPreviewUrl(item) {
    const lastVersion = item.versions[0];
    const examplePath = lastVersion.example_path;
    const exampleFile = JSON.parse(lastVersion.example_list)[0];
    return getPreviewUrl({
      name: item.classname,
      version: lastVersion.version,
      path: examplePath,
      file: exampleFile,
    });
  }

  return (
    <div className={styles.container}>
      <div className={styles.search}>
      <Search
          style={{ width: '50%' }}
          placeholder={intl.formatMessage({ id: 'INDEX_PLACEHOLDER' })}
          allowClear
          enterButton={intl.formatMessage({ id: 'INDEX_SEARCH' })}
          size='large'
          onSearch={value => {
            setName(value);
            setInit(false);
          }}
        />
      </div>
      <Divider orientation='right'>共{list.length}个组件</Divider>
      <Row gutter={16} justify='space-around'>
        {
          list.map(item => (
            <Col span={6} key={item.id}>
              <Card
                actions={[
                  <EyeOutlined key='setting' onClick={() => {
                    window.open(getLastPreviewUrl(item));
                  }}
                  />,
                  <EditOutlined key='edit' onClick={() => {
                    history.push({
                      pathname: '/detail',
                      query: {
                        id: item.id,
                      },
                    });
                  }}
                  />,
                  <EllipsisOutlined key='ellipsis' />,
                ]}
              >
                <Meta
                  avatar={<img
                    alt={item.name}
                    src={getAvatar(item).img}
                    style={getAvatar(item).style}
                    onClick={() => window.open(getGitUrl(item))}
                  />}
                  title={item.name}
                  description={item.description}
                />
              </Card>
            </Col>
          ))
        }
      </Row>
    </div>
  );
}
// src/pages/index.less
.containter {
  padding: 20px;
}

.search {
  padding:30px;
}
```
<a name="xxQ5W"></a>
### 
<a name="MRgHV"></a>
### 第五章 组件平台组件详情页面开发

---

<a name="hwguh"></a>
#### 5-1 组件详情获取API开发
> 首先在umi-component-dev项目下写details页面

```javascript
// src/pages/details.jsx
import { useState, useEffect } from 'react';
import styles from './detail.css';
import { getComponentItem } from "../utils/service";

export default function Page(props) {
  const [init, setInit] = useState(false)
  const [data, setData] = useState(null)

  useEffect(() => {
    const query = props.location.query
    if(!init && query.id){
      setInit(true);
      getComponentItem(query.id).then( data=> {
        console.log(data)
      })
    }
  })
  return (
    <div>
    </div>
  );
}
// src/utils/service.js
export function getComponentItem(id) {
  return request({
    url: `/api/v1/components/${id}`,
  });
}
```
> 接着，重点就是去开发接口获取组件的具体信息了

```javascript
// app/controller/v1/Components.js

 // api/v1/components/:id
async show() {
  const { ctx, app } = this;
  const id = ctx.params.id
  const results = await app.mysql.select('component', {
    where: { id }
  })
  if (results && results.length > 0) {
    const component = results[0]
    component.versions = await app.mysql.select('version', {
      where: { component_id: id },
      orders: [['version', 'desc']]
    })
    // gitee GET https://gitee.com/api/v5/repos/{owner}/{repo}/contents(/{path})
    // git GET https://api.github.com/repos/{owner}/{repo}/{path})
    let readmeUrl;
    let _name = component.classname;
    if (_name && _name.startsWith('@') && _name.indexOf('/') > 0) {
      const nameArray = _name.split('/');
      _name = nameArray.join('_').replace('@', '');
    }
    if (component.git_type === 'gitee') {
      readmeUrl = `https://gitee.com/api/v5/repos/${component.git_login}/${_name}/contents/README.md`
    } else {
      readmeUrl = `https://api.github.com/repos/${component.git_login}/${_name}/README.md`
    }
    const readme = await axios.get(readmeUrl);
    let content = readme.data && readme.data.content;
    if (content) {
      content = decode(content)
      if (content) {
        component.readme = content
      }
    }
    ctx.body = component
  } else {
    ctx.body = {}
  }
}
```
<a name="pqi63"></a>
#### 5-2 组件基本信息样式开发
<a name="x1hXf"></a>
#### 5-3 组件代码+预览样式开发
<a name="aiaLK"></a>
#### 5-4 组件安装样式和复制命令功能开发
<a name="nyPFY"></a>
#### 5-5 组件多预览文件上传工作
<a name="ECOlL"></a>
#### 5-6 组件多预览文件上传开发
