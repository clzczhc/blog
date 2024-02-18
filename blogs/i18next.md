---
title: i18next、react-i18next实现国际化
categories: 前端
date: 2023-10-11 23:55:04
tags:
    - i18next
    - 国际化
---

# i18next、react-i18next实现国际化

## 简单使用

i18n.js

```js
import i18n from "i18next";

import { initReactI18next } from "react-i18next";



i18n.use(initReactI18next).init({

  resources: {

    en: {

      translation: {

        React: "Welcome to React and react-i18next",

        Vue: "Welcome to Vue",

      },

    },

  },

  lng: "en",

  fallbackLng: "en", // 备选语言（当lng对应语言资源不存在时）



  interpolation: {

    escapeValue: false, // 是否进行转义，避免xss攻击。但是React本身可以防xss。所以可以关

  },

});



export default i18n;
```

在入口文件引入

```js
import "./i18n";
```

使用

```js
import React from "react";

import { useTranslation } from "react-i18next";



export default function Home() {

  const { t } = useTranslation();



  console.log(t("Vue"));



  return <h2>{t("Vue")}</h2>;

}
```

![](https://www.clzczh.top/CLZ_img/images/202310112303850.png)

## 使用`i18next-http-backend`插件

`yarn add i18next-http-backend --dev`

在 public 下面创建 locales 目录，在这个目录下创建和语言缩写对应的文件夹，其中放置 translation.json 文件。这个命名是约定好的，backend 插件会去按照这个路径请求资源文件。

![](https://www.clzczh.top/CLZ_img/images/202310112310680.png)

i18n.js

```js
import i18n from "i18next";

import { initReactI18next } from "react-i18next";



import Backend from "i18next-http-backend";



i18n

  .use(Backend)

  .use(initReactI18next)

  .init({

    lng: "en",

    fallbackLng: "en", // 备选语言（当lng对应语言资源不存在时）



    interpolation: {

      escapeValue: false,

    },

  });



export default i18n;
```

`public/locales/en/translation.json`

```json
{

  "React": "Welcome to React",

  "Vue": "Welcome to Vue"

}
```

##语言切换

```js
import { Translation, useTranslation } from "react-i18next";



import "./App.css";



function App() {

  const { t, i18n } = useTranslation();



  const changeLanguage = (lng) => {

    i18n.changeLanguage(lng);

  };



  return (

    <>

      <h1>{t("Vue")}</h1>

      <button onClick={() => changeLanguage("en")}>en</button>

      <button onClick={() => changeLanguage("zh")}>zh</button>

    </>

  );

}



export default App;
```

![](https://www.clzczh.top/CLZ_img/images/202310112314128.gif)

> 从上面的例子中能看出使用`i18next-http-backend`的好处：翻译资源能做到按需加载。而不使用的时候，暂时还没找到能实现按需加载的办法。即使将翻译资源分离开，最后还是得`import`进来，最后就一块打包了。

## 模块

> i18next的`namespace`可以实现将翻译资源分模块。这样子就能实现一个较好的按需加载，尤其是大项目里翻译资源非常多的时候。

i18n.js

```js
import i18n from "i18next";

import { initReactI18next } from "react-i18next";



import Backend from "i18next-http-backend";



i18n

  .use(Backend)

  .use(initReactI18next)

  .init({

    lng: "en",

    fallbackLng: "en", // 备选语言（当lng对应语言资源不存在时）



    interpolation: {

      escapeValue: false, // react already safes from xss => https://www.i18next.com/translation-function/interpolation#unescape

    },



    defaultNS: "common", // 默认是translation。修改后能让我们的模块块更有语义

    backend: {

      loadPath: "/locales/{{lng}}/{{ns}}.json",

    },

  });



export default i18n;
```

App.js

```js
import { useTranslation } from "react-i18next";

import { useState } from "react";



import "./App.css";



function App() {

  // 因为设置了defaultNS为`common`，所以可传可不传

  // 如果没设置，那想要加载`common`模块的资源时，就可以传参`common`

  const { t, i18n } = useTranslation(); /* useTranslation("common") */



  const changeLanguage = (lng) => {

    i18n.changeLanguage(lng);

  };



  const [show, setShow] = useState(false);



  const addPart = () => {

    i18n.loadNamespaces("addPart", (err) => {

      if (!err) {

        setShow(true);

      }

    });

  };



  return (

    <>

      <h1>{t("React")}</h1>

      <button onClick={() => changeLanguage("en")}>en</button>

      <button onClick={() => changeLanguage("zh")}>zh</button>

      <button onClick={() => changeLanguage("fr")}>fr</button>

      <button onClick={() => addPart()}>addPart</button>



      {/* 也能使用t("addPart:name")的形式 */}

      {show && <h1>{t("name", { ns: "addPart" })}</h1>}

    </>

  );

}



export default App;
```

![](https://www.clzczh.top/CLZ_img/images/202310112329915.gif)

> `loadNamespaces` 时，不只是加载当前语言的，而且还会加载`fallbackLng`的，应该是是保险之类的原因。上面的例子`fallbackLng`是`en`，当前语言为`zh`时，添加新模块就加载了`zh`和`en`的对应模块。

![](https://www.clzczh.top/CLZ_img/images/202310112332624.gif)

> 当当前语言就是`fallbackLng`时，则不会多加载。

## 完善

> 上面其实还是有一个问题，当加载某种语言的翻译资源时，默认都会加载`translation.json`，而该文件实际并没有内容

![](https://www.clzczh.top/CLZ_img/images/202310112336135.png)

> 猜测是因为没有设置`ns`，而默认的`ns`可能是`translation`。

i18n.js

```js
import i18n from "i18next";

import { initReactI18next } from "react-i18next";

import Backend from "i18next-http-backend";



i18n

  .use(Backend)

  .use(initReactI18next)

  .init({

    fallbackLng: "zh", //



    ns: ['common'],   // 没有这个，则会加载translation.json（默认）

    defaultNS: 'common',



    interpolation: {

      escapeValue: false,

    },

    backend: {

      loadPath: "/locales/{{lng}}/{{ns}}.json",

    }

  });



export default i18n;
```

![](https://www.clzczh.top/CLZ_img/images/202310112338965.gif)
