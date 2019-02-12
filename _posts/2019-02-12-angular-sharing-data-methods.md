---
title: angular sharing data
layout: post
---

# Angular5：如何在多个组件之间通信

<div class="title-meta">
    <span><img class="title-category-img" src="../../../assets/images/categories/angular.svg" alt="Angular"></span>
    <span><a class="github-link" href="/2018/09/19/angular.html">Angular</a></span>
    <span class="title-bullet">•</span>
    <span>Feb 12, 2019</span>
    <span class="title-bullet">•</span>
    <span>by <a class="github-link" href="http://github.com/limeii" title="http://github.com/limeii">Mei</a></span>
</div>

在Angular组件之间共享数据，有以下四种方式：


**1. 父组件至子组件: 通过@Input共享数据**


**2. 子组件至父组件: 通过@Output EventEmitter共享数据**


**3. 子组件至父组件: 通过@ViewChild共享数据**


**4. 不相关组件： 通过service共享数据**


在介绍这几种方式之前，先来看下父子组件和不相关组件是什么，在下图中可以看出，左边是描述父子组件关系，左右两个是描述不相关组件关系。
![css pseudo-classes-img]( https://limeii.github.io/assets/images/posts/angular/angular-sharingdata.png){:height="50%" width="50%"}


## 第一种方式，父组件至子组件: 通过@Input共享数据
这个例子是在子组件中直接列出出父组件中所有的颜色，具体代码如下：

```ts
//ParentComponent
import { Component, OnInit } from '@angular/core';

@Component({
    template:`<h4>Here is the color lists:</h4>
    <app-child [colorLists]="colorLists"></app-child>`
})

export class ParentComponent implements OnInit {
    private colorLists: any;
    ngOnInit() {
        this.colorLists = ['red', 'black', 'green', 'bule', 'yellow'];

    }
}
```
```ts
//ParentComponent
import { Component, OnInit, Input } from "@angular/core";

@Component({
    selector: 'app-child',
    template: `<ul>
                 <li *ngFor="let item of colorLists">{{item}}</li>
              </ul>`
})

export class ParentComponent implements OnInit {
    @Input() colorLists: any;
    ngOnInit() {
    }
}
```


## 第二种方式，子组件至父组件: 通过@Output EventEmitter共享数据
这个例子是子组件提供两个按钮进行投票，而父组件中需要实时显示投票结果，具体代码如下：

```ts
//VoteParentComponent
import { Component } from "@angular/core";

@Component({
    template: `<div style="margin-left:100px;font-size:1.28em;">
               <div> <strong>Here is the vote results:</strong></div>
               <div style="margin-top:20px;"> agreed:{{agreed}}   disagreed: {{disAgreed}}<div> 
               </div>   
               <app-vote-child (vote)="onVote($event)"></app-vote-child>
    `
})

export class VoteParentComponent {
    private agreed: number = 0;
    private disAgreed: number = 0;

    onVote(agree: boolean) {
        agree ? this.agreed++ : this.disAgreed++;
    }
}
```
```ts
//VoteChildComponent
import { Component, Output, EventEmitter } from "@angular/core";

@Component({
    selector: 'app-vote-child',
    template: `
                <div style="margin-top:30px;margin-bottom: 15px;"><strong>please take your vote:</strong> </div>
                <button (click)="takeVote(true)">Agree</button> <button (click)="takeVote(false)">Disagree</button>
                `
})

export class VoteChildComponent {
    @Output() vote = new EventEmitter<Boolean>();

    takeVote(agree: Boolean) {
        this.vote.emit(agree);
    }

}
```

## 第三种方式，子组件至父组件: 通过@ViewChild共享数据

在第二种方式中，通过@Output EventEmitter共享数据，父组件只能访问子组件的某几个属性值，但是没办法去调用子组件中的方法。但是通过@ViewChild可是实现父组件同时可以访问子组件中的属性和方法。


这个例子是一个简单的计时器，所有的计时逻辑都在子组件中，父组件负责显示时间。这个方式需要注意以下两点：


**子组件中方法或者变量为public的时候才能被父组件方法**
**在父组件中，只有等它本是AfterViewInit之后timerChildComponent才存在**

```ts
//TimerParentComponent
import { Component, ViewChild, AfterViewInit } from "@angular/core";

import { TimerChildComponent } from './timer-child.component';
import { timer } from "rxjs/observable/timer";

@Component({
    template: `
    <div style="margin-left:100px;font-size:1.28em">
    <div style="margin-top:20px;margin-bottom: 15px;"> <strong>This is a timer: </strong></div>
    <button (click)="start()"> start </button> <button (click)="reset()"> reset </button> 
    <div style="background:black;width:80px;height:80px;margin-top:20px;position:relative">
        <div style="font-size:2.5em;color:green;position:absolute;margin-left: 25px;margin-top: 10px;">{{seconds()}}</div>
    </div>
    <app-timer-child></app-timer-child>
    </div>
 `
})
export class TimerParentComponent implements AfterViewInit {

    @ViewChild(TimerChildComponent)
    private timerChildComponent: TimerChildComponent;
    seconds() {
        return 0;
    }
    ngAfterViewInit() {
        setTimeout(() => this.seconds = () => this.timerChildComponent.seconds, 0);
    }
    start() {
        this.timerChildComponent.onStart();
    }
    reset() {
        this.timerChildComponent.onReset();
    }
}
```

```ts
//TimerChildComponent
import { Component } from "@angular/core";

@Component({
    selector: 'app-timer-child',
    template: `
            <div style="margin-top:15px">reset count: {{resetCount}}</div>
    `

})
export class TimerChildComponent {

    public seconds: number = 10;
    private resetCount: number = 0;
    onStart() {
        window.setInterval(() => {
            if (this.seconds) {
                this.seconds = this.seconds - 1;
            }
        }, 1000)
    }
    onReset() {
        this.seconds = 10;
        this.resetCount++
    }
}
```