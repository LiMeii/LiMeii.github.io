---
title: Angular生命周期和钩子函数
tags: Angular
layout: post
---


Angular中每个component/directive都有它自己的生命周期。包括创建组件，渲染组件，创建渲染子组件，检测绑定属性变化，回收和从DOM中移除。


生命周期有着几种：OnChanges，OnInit，DoCheck，AfterContentInit，AfterContentChecked，AfterViewInit，AfterViewChecked，OnDestroy。


钩子函数就是在对应的生命周期前面加上前缀ng，比如OnInit，对应的钩子函数是ngOnInit()。

### 生命周期顺序

##### ngOnChanges()

##### ngOnInit()

##### ngDoCheck()

##### ngAfterContentInit()

##### ngAfterContentChecked()

##### ngAfterViewInit()

##### ngAfterViewChecked()

##### ngOnDestroy()
