---
title: Angular：动态创建Form表单
tags: Angular
layout: post
---


在web应用里通常会有这样一种场景：比如支付宝信用卡还款，假设支付宝收费标准如下：

- 普通用户，2000元以内免费，2000-50000收费10元，50000元以上收费15元。
- 砖石会员，5000元以内免费，5000-50000收费5元，50000元以上收费10元。

现在需要做一个页面，用来专门用来收集这样的收费标准，以后可能需要增加新的收费标准或者修改现有的收费标准。
这个页面可以设计成这样：
![dynamic form](https://limeii.github.io/assets/images/posts/angular/angular-dynamic-form.gif){:height="100%" width="100%"}

在angular用dynamic form可以很容易实现这种动态加载表单的效果，并且可以轻松实现对每一个field进行校验。接下来介绍如何在angular里实现上面的表单。

#### 开发环境如下：
![dynamic form](https://limeii.github.io/assets/images/posts/angular/dynamicform-env.png){:height="100%" width="100%"}

#### 项目结构如下：
![dynamic form](https://limeii.github.io/assets/images/posts/angular/angular-dynamic-form-structure.png){:height="100%" width="100%"}

**第一步，需要在app.module.ts引入FormsModule和ReactiveFormsModule**

```ts
//app.module.ts
import { FormsModule, ReactiveFormsModule } from '@angular/forms'
```

**第二步，创建DynamicFeeComponent，这个是每次动态添加的form表单**

 详细代码： 
 - [angular-dynamic-form-dynamic-fee.component.html](https://github.com/LiMeii/angular-dynamic-form/blob/master/src/app/dynamic-fee/dynamic-fee.component.html)
 - [angular-dynamic-form-dynamic-fee.component.ts](https://github.com/LiMeii/angular-dynamic-form/blob/master/src/app/dynamic-fee/dynamic-fee.component.ts)


**第三步，在app.component.ts中用FormArray动态添加DynamicFeeComponent**

 详细代码：
 - [angular-dynamic-form-app.component.html](https://github.com/LiMeii/angular-dynamic-form/blob/master/src/app/app.component.html)
 - [angular-dynamic-form-app.component.ts](https://github.com/LiMeii/angular-dynamic-form/blob/master/src/app/app.component.ts)

需要注意的是，formarray中每一项都是一个独立的formgroup，本质上来说在app.componnet中就是有两层嵌套的form，只不过这里的被嵌套的是一个formarray.
formarray是多个formgroup数组集合。在formarray formgroup的命名需要用数字表示：

```html
    <div formArrayName="feeArray">
      <div *ngFor="let arrayItem of feeArray.controls;let i=index">
        <div formGroupName=" { { i } } ">
          <dynamic-fee (deleteFeeItem)="removeFeeItem()" [group]="feeForm.controls.feeArray.controls[i]"></dynamic-fee>
        </div>
      </div>
    </div>
```

在DynamicFeeComponent里需要用到每一个formControl的时候，通过```[group]="feeForm.controls.feeArray.controls[i]" ```把每个formgroup传给DynamicFeeComponent。
否则的话一直会报类似这样的错:cannot access formcontrol


**第四步，在DynamicFeeComponent里，为每一个字段添加require的校验，formarray中只要有一个字段校验不对，整个form.valid就为false**

在每添加一个feeItem的时候，给每个字段设置require校验：

```ts
//app.component.ts
 addFeeItem() {
    this.feeArray.push(
      this.fb.group({
        userlevel: ['', Validators.required],
        tierMin: ['', Validators.required],
        tierMax: ['', Validators.required],
        amount: ['', Validators.required]
      })
    )
  }
```


校验方法如下：
```ts
//dynamic-fee.component.ts
    onValidate() {
        if (this.isSubmitted) {
            const form = this.group;
            for (const field in this.feeItemErrors) {
                if (form.get(field).errors) {
                    const error = Object.keys(form.get(field).errors);
                    this.feeItemErrors[field] = this.feeItemErrorsMessage[field][error[0]];

                } else {
                    this.feeItemErrors[field] = null;
                }
            }
        }
    }
```

需要注意的是，在钩子函数ngAfterContentChecked里需要重新调用onValidate方法，保证在点击submit button以后，每次更改页面里的值后能够实时校验。

```ts
  //dynamic-fee.component.ts
   ngAfterContentChecked() {
        if (this.isSubmitted) {
            this.onValidate();
        }
    }
```