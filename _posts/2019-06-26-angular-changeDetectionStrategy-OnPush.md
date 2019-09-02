---
title: Angular Change Detection：变化检测策略
tags: Angular
layout: post
---


在【[Angular Change Detection:变化检测机制](https://limeii.github.io/2019/06/angular-changedetection/)】这篇文章里介绍了angular的变化检测机制，也提到了异步事件都会触发整个angular应用的变化检测。


angular默认的变化检测机制是```ChangeDetectionStrategy.Default```：异步事件callback结束后，NgZone会触发整个组件树至上而下做变化检测，如下所示：

![angular-change-detection](https://limeii.github.io/assets/images/posts/angular/angular-change-detection-strategy01.png){:height="100%" width="100%"}

但是在实际应用里，并不是每个异步操作需要变化检测，某些组件也可以完全不用做变化检测，应用越大页面越复杂，过多的变化检测会影响整个应用的性能。angular除了默认的变化检测机制，也提供了```ChangeDetectionStrategy.OnPush```，用OnPush可以跳过某个component或者某个父组件以及它下面所有子组件的变化检测，如下所示：

![angular-change-detection](https://limeii.github.io/assets/images/posts/angular/angular-change-detection-strategy02.png){:height="100%" width="100%"}

我们来看下OnPush具体是怎么用的：

定义一个CDParentComponent如下：

```ts
@Component({
    template: `<h1>I am { { data.name } } and I live in { { data.address } } </h1>

               <cd-child [data]="data"></cd-child>
   
               <button (click)="changeInfo()">Change Info</button>`
})
export class CDParentComponent {
    data: any = {
        name: 'meii',
        address: 'ShangHai',
        contact: {
            email: 'XXX@gmail.com',
            phone: '1234567890'
        }
    };
    changeInfo() {
        this.data.contact.email = 'update@gmail.com';
        this.data.contact.phone = '00000000';
        this.data.name = 'limeii';
    }

}
```

定义一个CDChildComponent如下：

```ts
@Component({
    selector: "cd-child",
    template: `<h3>here is email in the child: { { data.contact.email } } </h3>`,
    changeDetection: ChangeDetectionStrategy.OnPush
})

export class CDChildComponent implements OnChanges {
    @Input() data: any;

    ngOnChanges() {
        console.log('data has been changed: ' + this.data.name + ' ' + this.data.address);
    }
}
```

在CDChildComponent加了一行代码：```changeDetection: ChangeDetectionStrategy.OnPush```，我们点击Change Info按钮，不会触发CDChildComponent中的变化检测，页面email也不会有变化。

<blockquote>
<p>
在CDChildComponent加了OnPush表示，在发生异步事件以后触发变化检测，angular会跳过这个组件，不会触发这个组件的变化检测。如果OnPush是加在某个父组件上，那么这个父组件和它下面所有的子组件都不会触发变化检测。
</p>
</blockquote>

但是在实际应用里，我们并不希望把整个组件的变化检测都禁掉，而是希望部分操作还是可以触发它的变化检测，比如从后端API返回新的数据，虽然加了OnPush，这些数据还是能够更新在页面上。


**<font color="#BF1827">angular也涵盖了上述需求，在组件里加了OnPush策略，以下四种情况还是可以触发该组件的变化检测：</font>**


1. 组件的@Input引用发生变化。

2. 组件的DOM事件，包括它子组件的DOM事件，比如click、submit、mouse down。

3. Observable订阅事件，同时设置Async pipe。

4. ChangeDetectorRef.detectChanges()、ChangeDetectorRef.markForCheck()、ApplicationRef.tick()，手动调用这三种方式触发变化检测。


## 1. 组件的@Input引用发生变化

必须是@Input的引用发生改变才会触发变化检测，并且仅限于@Input的变化检测，在OnPush策略下，会触发组件的变化检测。在这里先解释一下JS中的数据类型，在JS中有七种数据类型，其中包括六中原始类型（primitive values）和Object。


六种原始类型分别为：Boolean、Null、Undefined、Number、String、Symbol (ECMAScript 6 新定义)。


除了Object以外的所有类型（即原始类型）都是不可变的（Immutable），是通过值传递的，每次对它们的改动都会在内存里生成一个新的值。而Object是通过引用传递的，每次对Object改动，引用不会改变。


在上面的示例代码CDParentComponent中的```changeInfo```方法如下：

```ts
    changeInfo() {
        this.data.contact.email = 'update@gmail.com';
        this.data.contact.phone = '00000000';
        this.data.name = 'limeii';
    }

```
data是一个对象，在```changeInfo```方法里通过如上方式改变email的值。同时在CDChildComponent设置了OnPush，虽然```@Input data```的属性eamil发生变化但是data对象的引用并没有改变，并不会触发CDChildComponent中的变化检测，页面的eamil也不会发生变化。


如果把CDParentComponent中的```changeInfo```方法改成下面这样：

```ts
    changeInfo() {
        this.data = {
            name: 'meii', address: 'ShangHai',
            contact: {
                email: 'update@gmail.com',
                phone: '1234567890'
            }
        };
    }
```

这时候点击Change Info按钮，触发了变化检测，页面的email被更新了：

![angular-change-detection](https://limeii.github.io/assets/images/posts/angular/angular-change-detection-strategy05.gif){:height="100%" width="100%"}

<blockquote>
<p>
这种方式在改变data对象email值同时也改变了对象的引用。这时组件的@Input引用发生变化，虽然加了OnPush但@Input的变化检测还是会被触发。
</p>
</blockquote>

## 2. 组件DOM事件触发

组件的DOM事件，包括它子组件的DOM事件，比如click、submit、mouse down等事件，在OnPush策略下，会触发组件的变化检测。

在CDChildComponent加一个counter，并把它显示在页面里，在ngOnInit里把设置了setInterval，每过一秒就让```counter+1```，代码如下：

```ts
@Component({
    selector: "cd-child",
    template: `<h3>here is email in the child: { {data.contact.email } }</h3>
                <h3>here is counter in the child: { { counter } }</h3>
                `,
    changeDetection: ChangeDetectionStrategy.OnPush
})

export class CDChildComponent implements OnInit, OnChanges {
    @Input() data: any;
    counter: number = 1;

    ngOnInit() {
        setInterval(() => this.counter++, 1000);
    }

    ngOnChanges() {
        console.log('data has been changed: ' + this.data.name + ' ' + this.data.address);
    }

}
```
<blockquote>
<p>
如果是默认的变化检测策略，setInterval会触发组件变化检测，页面的counter会每过一秒就自动更新一次。现在CDChildComponent设置了OnPush，setInterval不会触发变化检测，页面上的counter不会有任何变化。
</p>
</blockquote>

在CDChildComponent页面加一个按钮，在这个按钮的点击事件里，设置每点击一次按钮让```counter+1```，具体代码如下：
```ts
@Component({
    selector: "cd-child",
    template: `<h3>here is email in the child: { { data.contact.email } } </h3>
               <h3>here is counter in the child: { { counter } } </h3>
               <div style="margin-bottom:10px;">
                    <button (click)="changeCounter()">change child counter</button>
                </div>`,
    changeDetection: ChangeDetectionStrategy.OnPush
})
export class CDChildComponent implements OnChanges {
    @Input() data: any;
    counter: number = 1;

    changeCounter() {
        this.counter++;
    }

    ngOnChanges() {
        console.log('data has been changed: ' + this.data.name + ' ' + this.data.address);
    }

}
```
<blockquote>
<p>
按钮点击事件是属于DOM事件，虽然在CDChildComponent设置了OnPush，组件的DOM事件（或者它的子组件DOM事件）还是会触发这个组件的变化检测，页面的counter会更新
</p>
</blockquote>
效果如下：

![angular-change-detection](https://limeii.github.io/assets/images/posts/angular/angular-change-detection-strategy08.gif){:height="100%" width="100%"}

<blockquote> <p> 
<font color="#BF1827">注意：这两个示例代码都是在@Input data引用没有发生变化的前提下运行的！</font>
</p></blockquote>

## 3. Observable事件订阅，同时设置Async pipe

在CDChildComponent有```Observable```事件订阅，并在模板里设置```Async pipe```，在OnPush策略下，会触发变化检测，代码如下：
```ts
@Component({
    selector: "cd-child",
    template: `<h3>here is email in the child: { { data.contact.email } } </h3>
                <h3>here is counter in the child: { { count$ | async } } </h3>
               <div style="margin-bottom:10px;">
                    <button (click)="changeCounter()">change child counter</button>
                </div>`,
    changeDetection: ChangeDetectionStrategy.OnPush
})

export class CDChildComponent implements OnInit, OnChanges {
    @Input() data: any;
    counter: number = 1;
    count$: Observable<number>;

    ngOnInit() {
        this.count$ = interval(1000)
            .pipe(
                map((count: number) => ++count)
            );
    }
    changeCounter() {
        this.counter++;
    }
    ngOnChanges() {
        console.log('data has been changed: ' + this.data.name + ' ' + this.data.address);
    }
}
```

这时候页面的会每隔一秒更新一次：

![angular-change-detection](https://limeii.github.io/assets/images/posts/angular/angular-change-detection-strategy10.gif){:height="100%" width="100%"}


## 4. 手动触发

ChangeDetectorRef.detectChanges()、ChangeDetectorRef.markForCheck()、ApplicationRef.tick()，在OnPush策略下，手动调用这三种方式会触发变化检测。

### 4.1 ChangeDetectorRef.detectChanges()

代码如下：
```ts
@Component({
    selector: "cd-child",
    template: `<h3>here is email in the child: { { data.contact.email } } </h3>
                <h3>here is the counter triggered manually in the child: { { counter } } </h3> 
               <div style="margin-bottom:10px;">
                    <button (click)="changeCounter()">change child counter</button>
                </div>`,
    changeDetection: ChangeDetectionStrategy.OnPush
})

export class CDChildComponent implements OnInit, OnChanges {
    @Input() data: any;
    counter: number = 1;
    count$: Observable<number>;

    constructor(private cd: ChangeDetectorRef) { }

    ngOnInit() {
        setInterval(() => {
            this.counter = this.counter + 5;
            this.cd.detectChanges();
        }, 1000);
    }
    ngOnChanges() {
        console.log('data has been changed: ' + this.data.name + ' ' + this.data.address);
    }

}
```

每隔一秒，counter自动加五，在OnPush策略下，组件会触发策略检测，页面每隔一秒会自动更新：

![angular-change-detection](https://limeii.github.io/assets/images/posts/angular/angular-change-detection-strategy12.gif){:height="100%" width="100%"}


### 4.2：ChangeDetectorRef.markForCheck()

代码如下：
```ts
@Component({
    selector: "cd-child",
    template: `<h3>here is email in the child: { { data.contact.email } } </h3>
                <h3>here is the counter triggered manually in the child: { { counter } } </h3> 
               <div style="margin-bottom:10px;">
                    <button (click)="changeCounter()">change child counter</button>
                </div>`,
    changeDetection: ChangeDetectionStrategy.OnPush
})

export class CDChildComponent implements OnInit, OnChanges {
    @Input() data: any;
    counter: number = 1;
    count$: Observable<number>;

    constructor(private cd: ChangeDetectorRef) { }

    ngOnInit() {
        setInterval(() => {
            this.counter = this.counter + 10;
            this.cd.markForCheck();
        }, 1000);
    }
    ngOnChanges() {
        console.log('data has been changed: ' + this.data.name + ' ' + this.data.address);
    }

}
```

<blockquote>
<p>
效果跟detectChanges是一样的，只不过detectChanges会立马触发当前组件和它子组件变化检测。markForCheck并不会立马触发变化检测，而是标记需要被变化检测，在当前或下一轮的变化检测中被触发。
</p>
</blockquote>


### 4.3：ApplicationRef.tick()

代码如下：

```ts
@Component({
    selector: "cd-child",
    template: `<h3>here is email in the child: { { data.contact.email } } </h3>
                <h3>here is the counter triggered manually in the child: { { counter } } </h3> 
               <div style="margin-bottom:10px;">
                    <button (click)="changeCounter()">change child counter</button>
                </div>`,
    changeDetection: ChangeDetectionStrategy.OnPush
})

export class CDChildComponent implements OnInit, OnChanges {
    @Input() data: any;
    counter: number = 1;
    count$: Observable<number>;

    constructor(private applicationRef: ApplicationRef)) { }

    ngOnInit() {
        setInterval(() => {
            this.counter = this.counter + 20;
             this.applicationRef.tick();
        }, 1000);
    }
    ngOnChanges() {
        console.log('data has been changed: ' + this.data.name + ' ' + this.data.address);
    }

}
```

<blockquote>
<p>
ApplicationRef.tick()触发整个应用的组件树从上到下执行变化检测。
</p>
</blockquote>

## 总结

在实际应用开发过程中，应该尽量遵循以下的原则：

- 尽量使用OnPush策略从叶节点组件开始来优化整个应用的性能

- 尽量多结合使用OnPush和async pipe

- 尽量多使用state management library，比如RxJS


本文中用到到的示例代码在这里：[angular-change-detection](https://github.com/LiMeii/angular-change-detection)