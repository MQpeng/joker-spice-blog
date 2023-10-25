---
title: vue是如何收集依赖的
date: 2023-10-12 10:12:07
categories: 前端深入浅出
tags: [前端, vue, vue3]
---

> 本文从源码层面进行分析，结合了实际`vue`项目经验，梳理`vue`的依赖收集，分析的`vue`版本为`3.3.3`
> 本文适合有一定实战经验或阅读经验的`vue`开发者

## 响应式基础

在了解依赖收集前，我们先了解下`vue`是如何书写响应式的。
```html
<!-- 模板代码 -->
<template>
    <div>count：{{ count }}</div>
    <div>data：{{ data.count }}</div>
</template>
```
```ts
// 脚本代码
// ==============
// 1. `ref`对象是可变的
// 2. 任何`.value`的读操作都会被追踪
// 3. 任何`.value`的写操作都会触发关联的副作用
const count = ref(0)
count.value++
console.log(count.value) // 1

// 1. 深层的：响应式会去覆盖所有的可嵌套属性
// 2. 如果属性本身是响应式的，响应式覆盖将被忽略
// 3. 原生的集合类型(Map、Array、Set)等，响应式覆盖将被忽略
// 4. 传入的对象将被包裹`ES Proxy`
const data = reactive({
    count: 0
})
data.count++
console.log(data.count) // 1
```

### `ref`响应式源码分析

```ts
export function ref(value?: unknown) {
  return createRef(value, false)
}

export function shallowRef(value?: unknown) {
  return createRef(value, true)
}

function createRef(rawValue: unknown, shallow: boolean) {
  if (isRef(rawValue)) {
    return rawValue
  }
  return new RefImpl(rawValue, shallow)
}

class RefImpl<T> {
  private _value: T
  private _rawValue: T

  // Set<ReactiveEffect>(effects?) 依赖集合，保存在响应式对象中
  public dep?: Dep = undefined
  public readonly __v_isRef = true

  constructor(value: T, public readonly __v_isShallow: boolean) {
    // toRaw: 返回原始对象，非响应式对象直接返回，响应式对象将返回原始对象(递归操作)
    this._rawValue = __v_isShallow ? value : toRaw(value)
    // toReactive: return isObject(value) ? reactive(value) : value
    this._value = __v_isShallow ? value : toReactive(value)
  }

  get value() {
    trackRefValue(this) // 最终调用 trackEffects(ref.dep || (ref.dep = createDep()))
    return this._value
  }

  set value(newVal) {
    const useDirectValue = this.__v_isShallow || isShallow(newVal) || isReadonly(newVal)
    newVal = useDirectValue ? newVal : toRaw(newVal)
    if (hasChanged(newVal, this._rawValue)) {
      this._rawValue = newVal
      this._value = useDirectValue ? newVal : toReactive(newVal)
      triggerRefValue(this, newVal)
    }
  }
}
```
在`ref`读操作的时候会调用`trackRefValue`，最终调用`trackEffects`进行依赖收集
另一方面，依赖收集只会存在在`watch`、`computed`、`template bind`等进行(`watchEffect`)


