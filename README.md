[Demo](https://virtual-list-scroll-v3.vercel.app/)

### Install

```shell
npm i virtual-list-scroll
```

### Import

```shell
import Vue from "vue"
import VirtualListScroll from "virtual-list-scroll-v3"
// 在components里面注册组件即可
```

### Usage

### Page Mode

```vue
<VirtualListScroll :data="blocks">
    <template v-slot:default="item">
        <div :style="{ height: '100%', 'background-color': item.color }">
          {{ item.id }}
        </div>
      </template>
</VirtualListScroll>
```

### Container Mode

```vue
<VirtualListScroll :height="300" :pageMode="false" :data="blocks">
    <template v-slot:default="item">
        <div :style="{ height: '100%', 'background-color': item.color }">
          {{ item.id }}
        </div>
      </template>
</VirtualListScroll>
```

### Flxed Block Height

```vue
<VirtualListScroll
  :height="300"
  :pageMode="false"
  :flxedBlockHeight="50"
  :data="blocks"
>
<template v-slot:default="item">
        <div :style="{ height: '100%', 'background-color': item.color }">
          {{ item.id }}
        </div>
      </template>
</VirtualListScroll>
```

### Unique virtual block

```vue
<VirtualListScroll :height="300" :pageMode="false" :data="blocks">
    <template v-bind:default="item">
    <template v-if="item.id===0"></template>
    <template v-if="item.id===1"></template>
    </template>
</VirtualListScroll>
```

### Props

| Name             | Type              | Default | Description                                        |
| ---------------- | ----------------- | ------- | -------------------------------------------------- |
| data             | Array<DataObject> | -       | 必填项,列表中数组数据                              |
| height           | Number            | -       | 虚拟滚动区域的高度                                 |
| flxedBlockHeight | Number            | -       | 每一个列表 Item 的高度                             |
| pageMode         | Boolean           | true    | 假定滚动条是在 window 上还是在指定的虚拟滚动盒子上 |

### DataObjet

| Name   | Type          | Default | Description                                          |
| ------ | ------------- | ------- | ---------------------------------------------------- |
| id     | String/Number | -       | 必填项，虚拟列表渲染的唯一 key                       |
| height | Number        | -       | 只有当`flxedBlockHeight`没有设定的时候才会使用这个值 |

### License

MIT
