---
title: "让你的代码更可读之 Table-driven"
date: "2015-06-24T13:50:46+02:00"
tags: ["theme"]
categories: ["starting"]
banner: "img/banners/banner-1.jpg"
comments: true
---

平时我们开发, 经常会遇到需要使用if, case的场景, 有时候阅读起来比较困难, 也不易于维护. 增加了修改的复杂性.

像比如我见过的有这样的.

```javascript
const getCdnAssetOrigin = ({ isLocal, isDev, isPro, isDemo, isQa }) => {
  let version = process.env.appVersion;
  let cdnOrigin = '';

  if (isLocal) {
    cdnOrigin = '';
  } else if (isPro) {
    cdnOrigin = '//other.wkcoding.com/cm_www/pro/';
  } else if (isDemo) {
    cdnOrigin = '//other.wkcoding.com/cm_www/demo/';
  } else if (isDev) {
    cdnOrigin = '//other.wkcoding.com/cm_www/dev/';
  }

  if (version) {
    cdnOrigin = `${cdnOrigin}${version}/`;
  }

  return cdnOrigin;
};
```

还有这样的.

```php
$xun_fei_text_2_audio = XunFeiText2AudioModule::instance();
switch ($type) {
    case XunFeiText2AudioModule::VOICE_NAME_LIST_TYPE['all']:
        $voice_name_list = $xun_fei_text_2_audio->getAllVoiceNames();
        break;
    case XunFeiText2AudioModule::VOICE_NAME_LIST_TYPE['free']:
        $voice_name_list = $xun_fei_text_2_audio->getFreeVoiceNames();
        break;
    case XunFeiText2AudioModule::VOICE_NAME_LIST_TYPE['paid']:
        $voice_name_list = $xun_fei_text_2_audio->getPaidVoiceNames();
        break;
    default:
        return $this->error('发音人不存在');
}
```

还有其他一些很复杂的情况, 上面只是列举最近我们团队在这两天code review发现的. 所以在这里我就不得不推荐使用一下 [《Code Complete》](https://en.wikipedia.org/wiki/Code_Complete) 第185章所提到的 table-driven.

### 关于 Table-driven
Table-driven是在Code Complete书中关于逻辑控制优化里面被提及的. 大概意思就是指. 我们可以将逻辑控制中的condition 和 对应处理的logic. 转换成表（key -> handler）的形式.

比如:
```javascript
let cdnOrigin = '';

if (isLocal) {
  cdnOrigin = '';
} else if (isPro) {
  cdnOrigin = '//other.wkcoding.com/cm_www/pro/';
} else if (isDemo) {
  cdnOrigin = '//other.wkcoding.com/cm_www/demo/';
} else if (isDev) {
  cdnOrigin = '//other.wkcoding.com/cm_www/dev/';
}
```

使用Table-driven后.

```javascript

let cdnTable = {
    [isLocal]: '',
    [isPro]: '//other.wkcoding.com/cm_www/pro/',
    [isDemo]: '//other.wkcoding.com/cm_www/demo/',
    [isDev]: '//other.wkcoding.com/cm_www/dev/'
}

let cdnOrigin = cdnTable[true];
```

那么这么做的好处是什么呢. 其实显而易见的就是. 对修改更加友好了。

```diff

let cdnTable = {
    [isLocal]: '',
    [isPro]: '//other.wkcoding.com/cm_www/pro/',
    [isDemo]: '//other.wkcoding.com/cm_www/demo/',
-    [isDev]: '//other.wkcoding.com/cm_www/dev/',
+    [isX]: '//other.wkcoding.com/cm_www/dev/'
}

if (isLocal) {
  cdnOrigin = '';
} else if (isPro) {
  cdnOrigin = '//other.wkcoding.com/cm_www/pro/';
-} else if (isDemo) {
-  cdnOrigin = '//other.wkcoding.com/cm_www/demo/';
-} 
+ } else if (isDev) {
  cdnOrigin = '//other.wkcoding.com/cm_www/dev/';
}
```

在上面一个如此简单的例子中. Table-driven 删减对应逻辑, 或在中间增加对应逻辑的方式就很简. 增加一行. 就算是闭着眼睛也不会出错。

当然上面的例子还稍微有点简单. 我们大部分场景可见的还是这种场景
```javascript
if (condition1) {
  // logic 1 line
  // logic 2 line
  // logic 3 line
} else if (condition2) {
  // logic 1 line
  // logic 2 line
  // logic 3 line
} else if (condition3) {
  // logic 1 line
  // logic 2 line
  // logic 3 line
}

/// Table-driven:
let handleTable = {
    [condition1]: () => {

    },
    [condition2]: () => {

    },
    [condition3]: () => {

    }
}

let handle = handleTable[condition];

if (handle) {
    return handle();
}

// default logic
```

### 最后
对于Table-driven实际场景有很多地方可以运用. 相信大家在日常code review, 或者看别人源码的过程中都会看到只是之前不认识. 这里我就介绍到这里. 大家可以在实际场景中运用来更加深入了解

### Reference
- https://www.d.umn.edu/~gshute/softeng/table-driven.html
- https://www.codeproject.com/Articles/42732/Table-driven-Approach
- https://zh.wikipedia.org/wiki/%E6%95%B0%E6%8D%AE%E9%A9%B1%E5%8A%A8%E7%BC%96%E7%A8%8B
- https://homepage.cs.uri.edu/~thenry/resources/unix_art/ch09s01.html