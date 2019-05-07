---
title: Hexo的NexT主题默认显示侧边栏
date: 2016-05-05 00:15:01
tags: "Hexo"
categories: "Hexo"
---

用Hexo的Next主题有一段时间了，但是发觉边框栏即便设置了 `sidebar: always`，也不能自动展开

在网上查了查，找到了[解决办法]("http://www.jianshu.com/p/29589e303f4d")

在 `\themes\next\source\js\motion_global.js` 做如下修改：

```
sidebarToggle: function (integrator) {
    sidebarToggleMotion.init();
    integrator.next();
    // 可以不加if直接显示
    // 加if的话，要在主题配置文件里面修改 sidebar: always
    if (CONFIG.sidebar === 'always') {
       // 模拟点击一下侧边栏按钮
       sidebarToggleMotion.clickHandler();
    }
}
```

哈哈，边框栏可以自动展开啦！开心