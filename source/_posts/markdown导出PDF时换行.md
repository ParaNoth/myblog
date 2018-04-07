---
title: markdown导出PDF时换行
date: 2018-04-07 15:20:43
tags: ['markdown', '文档']
category: 技术文档
---



用typora导出PDF时经常会发生一些奇怪的情况，比如把表格从两页中间截断，比如把代码块截断，所以需要手动分页。以下是分页方法：

在要分页的地方插入如下代码：

``` html
<div STYLE="page-break-after: always;"></div>
```

即可。