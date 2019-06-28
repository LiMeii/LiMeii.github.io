---
title: Angular下拉列表在IE和Safari显示问题
tags: 问题
layout: post
---

## 问题描述

1. Angular version： 4.2.4
2. 在 angular form里面包含Dropdown，并且对每个field都有require校验
3. 在chrome中dropdown显示如下：
![angular-selectoption-abnormal]( https://limeii.github.io/assets/images/posts/issues/issues-angular-selectoption-abnormal-iesafari1.png){:height="100%" width="100%"}
4. 在chrome中，点击提交按钮以后，因为每个dropdown都没有选择值，那么对应的FormControl都没有值，校验为False，required 错误信息显示，效果如下：
![angular-selectoption-abnormal]( https://limeii.github.io/assets/images/posts/issues/issues-angular-selectoption-abnormal-iesafari2.png){:height="100%" width="100%"}
5. 在chrome中，第一行的每个field都填上有效值，效果如下： 
![angular-selectoption-abnormal]( https://limeii.github.io/assets/images/posts/issues/issues-angular-selectoption-abnormal-iesafari3.png){:height="100%" width="100%"}
6. 在IE11中，默认显示的时候都把dropdown第一个值都显示已经选中了，效果如下：
![angular-selectoption-abnormal]( https://limeii.github.io/assets/images/posts/issues/issues-angular-selectoption-abnormal-iesafari4.png){:height="100%" width="100%"}
7. 在IE11/Safari中，页面显示dropdown都有值，实际对应的FormControl都没有值，点击提交按钮，还是有required错误信息：
![angular-selectoption-abnormal]( https://limeii.github.io/assets/images/posts/issues/issues-angular-selectoption-abnormal-iesafari5.png){:height="100%" width="100%"}
8. 在IE11/Safari中，切换dropdown里面的值，对应的FormControl会有值，错误信息消失，但是某个dropdown只有一个选项的时候，最开始默认会选上这个值，现在没有办法切换，导致它对应的FormControl永远没有值，这个field的校验永远为False

## 问题分析
在Angular Github的Issues下面有人提过类型的bug。


bug地址：[https://github.com/angular/angular/issues/14505](https://github.com/angular/angular/issues/14505)

其中有人给出了如下的解释：

![angular-selectoption-abnormal]( https://limeii.github.io/assets/images/posts/issues/issues-angular-selectoption-abnormal-iesafari6.png){:height="100%" width="100%"}


也就是说当用ngFor和[value]/[ngValue]动态显示dropdown中option：
```ts
@Component({
  selector: 'ng-model-select-form',
  template: `
    <select [(ngModel)]="selectedCity">
      <option *ngFor="let c of cities" [ngValue]="c"> {{c.name}} </option>
    </select>
  `
})
class NgModelSelectForm {
  selectedCity: {name: string} | null = null;
  cities: {name: string}[] = [];
}
```

<blockquote>
<p>
Chrome的行为是，没有默认值的时候，会把selectedIndex设置为-1，所以没有正常显示没有选中值，这也和FormControl的状态一致。
但是IE11和Safari的行为是，是把selectedIndex设置为0，页面显示是第一个被选中，但是FormControl的状态还是没有值，用户把dropwon选中的值从第一个值切换成其他值，会触发FormControl状态的改变。但是如果dropdown里只有一个值的时候，FormControl状态永远不会被触发改变。导致用户点击提交按钮，一直检测到这个FormControl没有值，校验永远过不了。
</p>
</blockquote>

### 解决方案

当没有选中任何值的时候，手动加一个空值，作为dropdown的第一个option，这样IE11/Safari会默认选择第一个空值，切换dropdown值的时候，会触发对应FormControl状态的改变。

```ts
<form [formGroup]="form">
	<select formControlName="select">
		<option *ngIf="!form.value.select" [ngValue]="null" selected></option>
		<option *ngFor="let option of options" [ngValue]="option">{{option}}</option>
	</select>
</form>
```