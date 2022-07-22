---
layout:     post
title:      如何解决devextreme  angular错误TypeError Cannot assign to read only property name of function data
subtitle:   fix devextreme TypeError: Cannot assign to read only property 'name' of function data
date:       2022-7-3
author:     王帅
catalog: true
tags:
    - devextreme
    - angular
typora-root-url: ..
---

### 引言

最近使用`devextreme`动态绑定`grid`数据的时候遇到一个异常

```js
core.mjs:6494 ERROR TypeError: Cannot assign to read only property 'name' of function 'data => {
                    var isCacheUpdated = storeData && storeData !== this._cachedStoreData;
        ...<omitted>... }'
    at inheritor.setName (ui.grid_core.columns_controller.js:1920:1)
    at createColumn (ui.grid_core.columns_controller.js:133:1)
    at Function.<anonymous> (ui.grid_core.columns_controller.js:169:1)
    at each (iterator.js:25:1)
    at createColumnsFromOptions (ui.grid_core.columns_controller.js:167:25)
    at inheritor.updateSortingGrouping (ui.grid_core.columns_controller.js:1758:1)
    at inheritor.updateColumns (ui.grid_core.columns_controller.js:1682:1)
    at inheritor.applyDataSource (ui.grid_core.columns_controller.js:950:1)
    at inheritor._handleDataChanged (ui.grid_core.data_controller.js:320:29)
    at inheritor._handleDataChanged (ui.grid_core.selection.js:511:1)
```



但本地测试的时候确实正常,只有发布后才有这个错误

![image-20220703105032197](/img/devextreme_assign_to_readonly_property.png)

进一步研究之后确定是数据加载过程的问题,本地测试时数据用的静态的;加载瞬间完成,从服务器取数据后绑定列延后了导致报错

只要给本地测试数据添加延迟就能复现问题

`return of(dataTemp).pipe(delay(3000));`

### 解决方案

解决方法就是动态列的创建放在数据取完之后

```ts
  constructor(private service: Service) {
          this.service.getdata().subscribe(data => {              
            	    this.columns.push(<Column>{ isBand: true, caption: xxx,columns:listmidcol });//先创建列
                    this.dataGrid.instance.option('dataSource',data);//再绑定数据
          }
  }
```

### 参考资料
无
