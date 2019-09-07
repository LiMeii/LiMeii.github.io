---
title: Angular ngOnChanges事件没有被触发
tags: 问题
layout: post
---

## 问题描述

1. Angular version： 4.2.4
2. AppComponent和ChildComponent，这两个Component是父子关系，在ChildComponent里定义了@Input() data，由AppComponent把data这个对象传给ChildComponent，当data是一个对象的时候，更改这个对象某个属性的值的时候，不会触发ChildComponent里的ngOnChanges()事件，代码如下：

```ts
@Component({
  selector: 'app-root',
  template: `<child-component [data]="data"></child-component>
             <button (click)="onClick()">Submit</button>`
})
export class AppComponent {
  data: any =  { name: 'meii', address: 'ShangHai' };

  onClick(){
      this.data.address='China'
  }
}

```

```ts
@Component({
    template: '<h5>{{data.name}} {{data.address}}</h5>',
    selector: 'child-component'
})

export class ChildComponent implements OnChanges,DoCheck {
    @Input() data: any = {}


    ngOnChanges() {
        // won't trigger when user click submit button in AppComponent
        console.log('data has been changed: ' + this.data.name + ' ' + this.data.address);
    }
}
```
## 问题分析

把data改成一个string类型, 这次是正常的，用户点击submit按钮，把data从ShangHai改为China，ChildComponent里的ngOnChanges事件会被触发， 代码如下：

```ts
@Component({
  selector: 'app-root',
  template: `<child-component [data]="data"></child-component>
             <button (click)="onClick()">Submit</button>`
})
export class AppComponent {
  data: string =  'ShangHai';

  onClick(){
      this.data = 'China'
  }
}

```

```ts
@Component({
    template: '<h5>{{data.name}} {{data.address}}</h5>',
    selector: 'child-component'
})

export class ChildComponent implements OnChanges,DoCheck {
    @Input() data: any = {}


    ngOnChanges() {
        //  trigger when user click submit button in AppComponent
        console.log('data has been changed: ' + this.data.name + ' ' + this.data.address);
    }
}
```

Google了一下， 在github上找到了一个angular员工给出的一个答复如下：
![angular-ngonchanges-notfired](https://limeii.github.io/assets/images/posts/issues/angular-ngonchanges-notfired1.png){:height="100%" width="100%"}

总结来说，就是angular的变化检测的策略是：对于对象来说，只是比较当前对象的引用有没有变化，而不会去比较对象里面的属性值有没有变化。不会做deep comparison。

### 解决方案
用ngDoCheck()方法，任何变化都会触发这个方法。具体代码如下：

```ts
@Component({
    template: '<h1>here is the child template</h1>',
    selector: 'child-component'
})

export class ChildComponent implements OnChanges,DoCheck {
    @Input() data: any = {}

    ngDoCheck(){
        //always will be triggered
        console.log('do check has been fired, data has been changed: ' + this.data.name + ' ' + this.data.address);
    }
}
```