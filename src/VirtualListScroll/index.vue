<template>
  <div
    v-on="pageMode ? {} : { scroll: handleScroll }"
    :style="containerStyle"
    ref="vb"
  >
    <div :style="{ height: `${offsetTop}px` }"></div>
    <div
      :style="{
        height: `${flxedBlockHeight ? flxedBlockHeight : item.height}px`,
      }"
      v-for="item in renderList"
      :key="item.id"
    >
      <slot v-bind="item"></slot>
    </div>
    <div :style="{ height: `${offsetBot}px` }"></div>
  </div>
</template>

<script>
import {
  reactive,
  ref,
  toRefs,
  watch,
  nextTick,
  onMounted,
  onUnmounted,
  computed,
} from "vue";
import { getScrollParent } from './scroll-parent';
export default {
  props: {
    // 具体的列表的数组
    data: {
      type: Array,
      required: true,
    },
    // 虚拟块的高度
    height: {
      type: Number,
    },
    // 虚拟滚动去每一个item的高度
    flxedBlockHeight: {
      type: Number,
    },
    // 假定是window有滚动条还是虚拟滚动区有滚动条
    pageMode: {
      type: Boolean,
      default: false,
    },
  },
  setup(props, ctx) {
    const state = reactive({
      renderList: [],
      viewportEnd: props.height,
      viewportBegin: 0,
      offsetTop: 0,
      offsetBot: 0,
      transformedData: [],
    });
    const vb = ref(null);
    const vbOffSet = ref(0);
    let scrollableElement = ref(null);

    watch(
      () => props.data,
      (data, preData) => {
        computeTransformedData(data);
        // 如果存在旧值
        if (preData) {
          nextTick(() => {
            vb.value.scrollTop = 0;
            handleScroll();
          });
        }
      },
      {
        immediate: true,
      }
    );

    onMounted(() => {
      // Determine the scrollable parent element
      scrollableElement = getListenerTarget();
      // Determine the offsetTop
      const { top } = vb.value.getBoundingClientRect();
      vbOffSet.value = top;      
      // 判断页面的模式
      if (props.pageMode) {
        computeTransformedData(props.data);       
        scrollableElement.addEventListener('scroll', handleScroll);
      }
      updateVb(0);
    });

    onUnmounted(() => {
      // 组价销毁时清除之前的事件
      if (props.pageMode) {
        scrollableElement.removeEventListener("scroll", handleScroll);
      }
    });
    watch(
      () => props.pageMode,
      (pageMode, prevPageMode) => {
        if (pageMode) {
          scrollableElement = getListenerTarget();
          scrollableElement.addEventListener("scroll", handleScroll);
        } else {
          // 如果不存在新的之变化直接清楚这个scorll的滚动监听
          scrollableElement.removeEventListener("scroll", handleScroll);
        }
        // 值发生变化直接需要重新计算
        computeTransformedData(props.data);
        nextTick(() => {
          vb.value.scrollTop = 0;
          handleScroll();
        });
      }
    );
    watch(
      () => props.flxedBlockHeight,
      (flxedBlockHeight, prevFlxedBlockHeight) => {
        handleScroll();
      }
    );
    function computeTransformedData(oldArr) {
      if (
        (!props.flxedBlockHeight && props.pageMode && vb.value) ||
        !props.pageMode
      ) {
        let curHeight = props.pageMode ? vb.value.offsetTop : 0;
        // 每一次的虚拟列表的第一个的scrolTop
        let rt = [curHeight];
        oldArr.forEach((item) => {
          curHeight += item.height;
          // 每一次计算后端的scrollTop
          rt.push(curHeight);
        });
        state.transformedData = rt;
      }
    }

    /**
  	滚动检测方法
  	*/
    function handleScroll() {
      // 计算高度
      const scrollTop = props.pageMode
        ? (scrollableElement.scrollTop || scrollableElement.pageYOffset)
        : vb.value.scrollTop;
      // 为了必要滚动过快需要使用动画帧
      window.requestAnimationFrame(() => {
        // 更新虚拟滚动区内的内容
        updateVb(scrollTop);
      });
    }

    /**
     * get listener target
     */
     function getListenerTarget() {
      let target = getScrollParent(vb.value);
      // Fix global scroll target for Chrome and Safari
      if (window.document && (target === window.document.documentElement || target === window.document.body)) {
        target = window;
      }
      return target;
    }

    /**
     * 存在指定的盒子高度寻找开始的下表
     */
    function findBlockHeightLowerBound(viewportBegin, flxedBlockHeight) {
      const sAdjusted = props.pageMode
        ? viewportBegin - vb.value.scrollTop
        : viewportBegin;
      const computedStartIndex = ~~(sAdjusted / flxedBlockHeight);
      return computedStartIndex >= 0 ? computedStartIndex : 0;
    }

    /**
     * 不存在盒子指定的高度二分法寻找开始的下表
     */
     function binarySearchLowerBound(viewportBegin, transformedData) {
      let lo = 0;
      // 最后一个高度的前一个其实就代表最后一个的头部位置
      let hi = transformedData.length - 1;
      let mid;
      while (lo <= hi) {
        // 向下取整
        mid = ~~((lo + hi) / 2);
        // 判断这个中间位置的高度是否大于当前开始的高度
        if (transformedData[mid] > viewportBegin) {
          // 如果中间值为0说明就是开始位置
          if (mid === 0) {
            return 0;
          } else {
            hi = mid - 1;
          }
        } else if (transformedData[mid] < viewportBegin) {
          // 如果当前中间的高度 小于 当前开始的高度
          if (mid + 1 < transformedData.length) {
            if (transformedData[mid + 1] > viewportBegin) {
              return mid;
            } else {
              lo = mid + 1;
            }
          } else {
            return -1;
          }
        } else {
          return mid;
        }
      }
    }
    function findBlockHeightUpperBound(viewportEnd, flxedBlockHeight) {
      const eAdjusted = props.pageMode
        ? viewportEnd - vb.value.scrollTop
        : viewportEnd;
      // 末尾的开始下标
      const computedStartIndex = ~~(eAdjusted / flxedBlockHeight);
      // 判断一下大于0就是正常返回否则就是返回0
      return computedStartIndex <= props.data.length
        ? computedStartIndex
        : props.data.length;
    }
    /**
     * 找到后面的的开始下标，但是不能有固定的高度
     */
     function binarySearchUpperBound(viewportEnd, transformedData) {
      let lo = 0;
      let hi = transformedData.length - 1;
      let mid;
      // 二分法查找，找到末尾的下标
      while (lo <= hi) {
        mid = ~~((lo + hi) / 2);
        if (transformedData[mid] > viewportEnd) {
          if (mid > 0) {
            if (transformedData[mid - 1] < viewportEnd) {
              return mid;
            } else {
              hi = mid - 1;
            }
          } else {
            return -1;
          }
        } else if (transformedData[mid] < viewportEnd) {
          // 如果中间下标等于数组的长度减一
          if (mid === transformedData.length - 1) {
            return mid;
          } else {
            // 如果不等于数组长度那么我们需要加1
            lo = mid + 1;
          }
        } else {
          return mid;
        }
      }
    }
    function findBlocksInViewport(viewportBegin, viewportEnd, transformedData, data) {
      // 当视窗的开始高度小于视窗结束的高度
      if (viewportBegin < viewportEnd) {
        // 判断flxedBlockHeight是否存在
        // 开始下标
        const lo = props.flxedBlockHeight
          ? findBlockHeightLowerBound(viewportBegin, props.flxedBlockHeight)
          : binarySearchLowerBound(viewportBegin, transformedData);
        // 结束下标
        const hi = props.flxedBlockHeight
          ? findBlockHeightUpperBound(viewportEnd, props.flxedBlockHeight)
          : binarySearchUpperBound(viewportEnd, transformedData);
        // 计算虚拟偏移量
        let vbOffset = props.pageMode ? vb.value.offsetTop : 0;
        // 判断是否存在固定的item的高度
        if (props.flxedBlockHeight) {
          state.offsetTop = lo >= 0 ? lo * props.flxedBlockHeight : 0;
          state.offsetBot =
            hi >= 0
              ? (data.length - hi) * props.flxedBlockHeight : 0
        } else {
          state.offsetTop = lo >= 0 ? transformedData[lo] - vbOffset : 0;
          state.offsetBot =
            hi >= 0
              ? transformedData[transformedData.length - 1] -
                transformedData[hi]
              : 0;
        }
        return data.slice(lo, hi);
      } else {
        state.offsetTop = 0;
        state.offsetBot = 0;
        // 直接返回一个空的列表
        return [];
      }
    }
    // 更新虚拟滚动区域内部的高度
    function updateVb(scrollTop) {
      // 计算滚动可视区真实的高度
      const viewportHeight = props.pageMode ? window.innerHeight : props.height;
      state.viewportBegin = props.pageMode ? scrollTop - vbOffSet.value : scrollTop;
      state.viewportEnd = scrollTop + viewportHeight;
      state.renderList = findBlocksInViewport(
        state.viewportBegin,
        state.viewportEnd,
        state.transformedData,
        props.data
      );
    }

    const containerStyle = computed(() => {
      return {
        ...(!props.pageMode && { height: `${props.height}px` }),
        ...(!props.pageMode && { "overflow-y": "auto" })
      }
    })
    return {
      ...toRefs(state),
      vb,
      containerStyle,
      handleScroll
    };
  },
};
</script>