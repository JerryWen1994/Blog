# React hooks 长列表加载优化

![3e70a9d027964f667c3548ac3925e468](https://user-images.githubusercontent.com/20969011/61099814-37d0ef00-a496-11e9-94d4-09165a235e04.jpg)

前端小兵，不吝赐教

## 背景

在项目开发中会经常遇到需要加载滚动列表，而且这列表还不是少量数据的列表，数据量可能达到上百甚至是上千。
对于较长的列表，例如数据量为一千，如果没有分页，而是想同时渲染这一千条数据，生成对应的 dom 节点，这么多数量的 dom 元素全部加载的话，可能会使页面失去响应，导致整个应用崩溃。

所以我们就需要通过虚拟列表的方式来优化长列表的加载。

现有的 React 长列表优化方案有几种，如：

- [react-virtualized](https://github.com/bvaughn/react-virtualized)
- [react-tiny-virtual-list](https://github.com/clauderic/react-tiny-virtual-list)

所以我打算用另一种方案来实现长列表的加载。

## 虚拟列表优化原理

首先，我们先讲下虚拟列表的优化原理。

它的原理是 `将数据保存至数组中，只渲染可视窗口的列表元素。当可视区域滚动时，需要根据滚动的位移量，计算出可视区域中的元素下标，通过下标来渲染数组元素。`

具体实现步骤如下：

1. 首先我们可以通过 react-custom-scrollbars 来获取可视窗口当前所在的高度偏移量 scrollTop ，并且我们需要手动传入每行的高度 rowHeight ，计算出已加载的元素下标 baseIndex 。

2. 接着我们需要计算出可视区域所能够渲染的行数 rowNumberCanSee ，这个也需要 react-custom-scrollbars 来获取可视窗口的高度与传入的 rowHeight 来计算得出。

3. 然后通过上面计算的 baseIndex 与 rowNumberCanSee 来计算可视区域的起始下标与结束下标。

![20190712103737](https://user-images.githubusercontent.com/20969011/61098783-be83cd00-a492-11e9-9962-93f5b56c1a64.png)

## react-custom-scrollbars

长列表优化方案的原理我们知道了，接下来我们介绍下 [react-custom-scrollbars](https://malte-wessel.com/react-custom-scrollbars/)，它的 [API 文档](https://github.com/malte-wessel/react-custom-scrollbars/blob/master/docs/API.md)。

react-custom-scrollbars 可以通过 getValues() 方法，给出我们当前滚动条的状态参数：

```typescript
positionValues: {
  top: number; // 滚动条离顶部位移比，最大为 1 时代表滚动条已置底
  left: number; // 滚动条离左部位移比，最大为 1 时代表滚动条已置右
  clientWidth: number; // 可视窗口的宽度
  clientHeight: number; // 可视窗口的高度
  scrollWidth: number; // 可滚动的最大宽度
  scrollHeight: number; // 可滚动的最大高度
  scrollLeft: number; // 当前距离左部已滚动的宽度
  scrollTop: number; // 当前距离顶部已滚动的高度
}
```

它也暴露几个方法给我们使用，在这个优化里，笔者就使用了 `onScrollFrame` 与 `onScrollStop`。

在滚动条滚动时，都会触发 `onScrollFrame` ，并且会将滚动条的信息传给我们。

在滚动条停止滚动时，会触发 `onScrollStop` ，这个方法我们就可以用来监听是否需要加载更多数据，这个在动态加载的时候用到，相当于分页加载。

实现效果
![GIF](https://user-images.githubusercontent.com/20969011/61096844-8e84fb80-a48b-11e9-85bb-e428dff7e32d.gif)

## 具体实现

我们需要接收以下 props 值：

```tsx
interface LongListProps<T> {
  dataSource: T[];
  rowHeight: number;
  hasMore?: boolean;
  loading?: boolean;
  empty?: boolean;
  error?: boolean;
  className?: string;
  reachEndThreshold?: number;
  overscanNumber?: number;
  rowKey?: string;
  renderRow: (data: {
    row: T;
    index: number;
    style: React.CSSProperties;
  }) => React.ReactNode;
  renderLoader?: () => React.ReactNode;
  renderEmpty?: () => React.ReactNode;
  renderError?: () => React.ReactNode;
  onReachEnd?: () => void;
}
```

我们将类型定义为泛型，可以方便我们传入数据格式的多样化。
以上的值不固定，以项目需求来使用。但是 `dataSource`, `rowHeight` 与 `renderRow` 为必传项，需要知道数据，固定行的高度，以及渲染我们的 dom 元素的方法 `renderRow` 的返回值就是我们的虚拟 dom 节点。

接着就是计算我们的可视区域范围所展示的起始节点与结束节点的下标。

```tsx
const computedVisibleRange = useCallback((pos: positionValues) => {
  // pos 滚动条当前状况参数
  const { scrollTop, clientHeight } = pos;
  const { rowHeight, overscanNumber = 5, dataSource } = propsRef.current;
  // 如果没有数据就直接跳过计算
  if (dataSource === null || !dataSource.length) {
    setRange(null);
    return;
  }

  const baseIndex = Math.ceil(scrollTop / rowHeight);
  const rowNumbercanSee = Math.floor(clientHeight / rowHeight);
  const startIndex = clamp(baseIndex - overscanNumber, 0, dataSource.length - 1 );
  const endIndex = clamp( baseIndex + rowNumbercanSee + overscanNumber, baseIndex, dataSource.length - 1 );
  requestAnimationFrame(() => setRange([startIndex, endIndex]));
}, []);
```

其中 overscanNumber 是我们定义的超过可视化区域加载的行数。如果我们赋值 5 ，这时可以看到已加载的 dom 节点为可视区域加上 overscanNumber 的下标的元素。
dataSource 则是我们传入的列表数据。

根据业务需求，我们可以传入 `hasMore` 与 `onReachEnd`， 来动态加载更多的数据，在滚动条停止滚动的时候触发以下方法。

```tsx
const handleScrollStop = useCallback(() => {
  const { loading, onReachEnd, hasMore, reachEndThreshold } = propsRef.current
  // 判断是否有 onReachEnd，并且判断是否正在加载与能够加载更多数据
  if (onReachEnd && !loading && hasMore) {
    const threshold = reachEndThreshold || 200
    const value = scrollbar.current!.getValues()
    // 计算当前位置距离底部的高度
    const scrollBottom = value.scrollHeight - value.scrollTop - value.clientHeight
    if (scrollBottom < threshold) {
      // 小于设置的加载高度，则进行加载更多数据
      onReachEnd()
    }
  }
}, [])
```

其中 `loading` 需要我们手动去维护该变量， `hasMore` 则是请求分页返回的数据中进行判断是否有下一页。
我们需要计算 `scrollBottom` 距离底部的高度，并比较是否小于我们设定的 `threshold` 来进行判断是否需要加载更多数据，这样就避免每次滚动条停止滚动时都执行加载更多。

然后我们将计算出的范围下标进行渲染加载，通过 `renderRow` 方法进行渲染加载。
当然，这个方法是在 `render` 中执行。

```tsx
range && iterRange(range[0], range[1], i => {
  const row = props.dataSource[i]
  return (
    row != null &&
    props.renderRow({
      row,
      index: i,
      style: {
        position: 'absolute',
        width: '100%',
        top: i * props.rowHeight,
      },
    })
  )
})
```

其中 `iterRange` 方法是将 range 范围的元素遍历并返回一个新的数组，跟 `map` 方法类似。

```tsx
function iterRange<T>(start: number, end: number, cb: (index: number) => T) {
  const list: T[] = []
  for (let i = start; i <= end; i++) {
    list.push(cb(i))
  }
  return list
}
```

完成了长列表的加载渲染后，我们可以考虑下扩展。

- 首次加载时，如果数据请求慢，我们可以通过骨架屏的形式，减少我们白屏的显示，也让用户不会感觉太突兀

- 数据加载失败时，错误信息的提示，以及可重新请求接口的操作

- 数据为空时，空数据的提示，避免以为是渲染失败

以上都可以通过我们传进去的 props 值进行判断加载节点

```tsx
{props.loading && props.renderLoader && props.renderLoader()}
{props.error && props.renderError && props.renderError()}
{!props.hasMore && props.dataSource.length === 0 && props.renderEmpty && props.renderEmpty()}
```

骨架屏的话，我们可以用 `react-content-loader` 来进行编写，具体实现就不在这里细讲了，看官网文档就可以实现了，非常简单。

## 总结

今天分享了通过虚列表来实现长列表的加载渲染优化，其实都是比较限制性的，需要限制行高，这也是一个待解决的问题之一，还有就是在网上看到别人说到的惯性滚动，这些都需要我们更加深入去研究并实现的。
今天分享的东西有点皮毛，也是想记录下自己的代码，希望各位看官也能够提出各自宝贵的意见，谢谢！！

## 整体代码：

```tsx
import { useRefProps } from '@gdjiami/hooks'
import { observer } from 'mobx-react-lite'
import React, { useRef, useState, useCallback, useEffect } from 'react'
import Scrollbars, { ScrollbarProps, positionValues } from 'react-custom-scrollbars'


export interface LongListProps<T> {
  dataSource: T[]
  rowHeight: number
  hasMore?: boolean
  loading?: boolean
  empty?: boolean
  error?: boolean
  className?: string
  reachEndThreshold?: number
  overscanNumber?: number
  rowKey?: string
  renderRow: (data: { row: T; index: number; style: React.CSSProperties }) => React.ReactNode
  renderLoader?: () => React.ReactNode
  renderEmpty?: () => React.ReactNode
  renderError?: () => React.ReactNode
  onReachEnd?: () => void
}

function clamp(num: number, min: number, max: number) {
  return num < min ? min : num > max ? max : num
}

function iterRange<T>(start: number, end: number, cb: (index: number) => T) {
  const list: T[] = []
  for (let i = start; i <= end; i++) {
    list.push(cb(i))
  }
  return list
}

/**
 * 长列表，支持虚列表
 */
export const LongList = observer(props => {
  const scrollbar = useRef<Scrollbars>(null)
  const propsRef = useRefProps(props)
  const [range, setRange] = useState<[number, number] | null>()

  const handleScrollStop = useCallback(() => {
    const { loading, onReachEnd, hasMore, reachEndThreshold } = propsRef.current
    if (onReachEnd && !loading && hasMore) {
      const threshold = reachEndThreshold || 200
      const value = scrollbar.current!.getValues()
      // 计算当前位置距离底部的高度
      const scrollBottom = value.scrollHeight - value.scrollTop - value.clientHeight
      if (scrollBottom < threshold) {
        // 小于设置的加载高度，则进行加载更多数据
        onReachEnd()
      }
    }
  }, [])

  const computedVisibleRange = useCallback((pos: positionValues) => {
    const { scrollTop, clientHeight } = pos
    const { rowHeight, overscanNumber = 5, dataSource } = propsRef.current
    if (dataSource === null || !dataSource.length) {
      setRange(null)
      return
    }

    const baseIndex = Math.ceil(scrollTop / rowHeight)
    const rowNumbercanSee = Math.floor(clientHeight / rowHeight)
    const startIndex = clamp(baseIndex - overscanNumber, 0, dataSource.length - 1)
    const endIndex = clamp(baseIndex + rowNumbercanSee + overscanNumber, baseIndex, dataSource.length - 1)
    requestAnimationFrame(() => setRange([startIndex, endIndex]))
  }, [])

  useEffect(() => {
    if (props.dataSource === null || !props.dataSource.length) {
      setRange(null)
    } else {
      computedVisibleRange(scrollbar.current!.getValues())
    }
  }, [props.dataSource, props.overscanNumber, props.rowHeight])

  return (
    <Scrollbars
      ref={scrollbar}
      autoHide
      className={props.className}
      onScrollStop={handleScrollStop}
      onScrollFrame={computedVisibleRange}
    >
      {range &&
        iterRange(range[0], range[1], i => {
          const row = props.dataSource[i]
          return (
            row != null &&
            props.renderRow({
              row,
              index: i,
              style: {
                position: 'absolute',
                width: '100%',
                top: i * props.rowHeight,
              },
            })
          )
        })}
      {props.loading && props.renderLoader && props.renderLoader()}
      {props.error && props.renderError && props.renderError()}
      {!props.hasMore && props.dataSource.length === 0 && props.renderEmpty && props.renderEmpty()}
    </Scrollbars>
  )
}) as <T>(props: LongListProps<T>) => React.ReactElement
```
