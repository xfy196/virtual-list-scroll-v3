<template>
  <div>
    <div class="header">Virtual List Scroll Vue3</div>
    <div class="userOptions">
      <span class="option">
        <span> Data Amount </span>
        <select v-model="dataAmt">
          <option value="100">100</option>
          <option value="1000">1000</option>
          <option value="10000">10000</option>
          <option value="200000">200000</option>
          <option value="500000">500000</option>
        </select>
      </span>
      <span class="option">
        <span> Page Mode </span>
        <input type="checkbox" v-model="isPageMode" />
      </span>
      <span class="option">
        <span> Fixed Block Height </span>
        <input type="checkbox" v-model="isFixedHeight" />
      </span>
    </div>
    <VirtualListScroll
      :fixedBlockHeight="isFixedHeight ? 50 : undefined"
      v-if="true"
      :pageMode="isPageMode"
      :height="500"
      :data="data"
      ref="vb"
    >
      <template v-slot:default="item">
        <div :style="{ height: '100%', 'background-color': item.color }">
          {{ item.id }}
        </div>
      </template>
    </VirtualListScroll>
  </div>
</template>

<script>
import { reactive, toRefs, watch } from "vue";
import VirtualListScroll from "./VirtualListScroll/index.vue";
export default {
  name: "App",
  components: {
    VirtualListScroll,
  },
  setup(props, ctx) {
    const state = reactive({
      data: [],
      dataAmt: "1000",
      isPageMode: false,
      isFixedHeight: true,
    });
    state.data = dataConstructor(state.dataAmt, state.isFixedHeight);
    watch(
      () => state.dataAmt,
      (dataAmt) => {
        state.data = this.dataConstructor(dataAmt, state.isFixedHeight);
        // this.$forceUpdate();
      }
    );
    watch(
      () => state.isPageMode,
      (isPageMode) => {}
    );
    watch(
      () => state.isFixedHeight,
      (isFixedHeight) => {
        state.data = this.dataConstructor(state.dataAmt, isFixedHeight);
        // this.$forceUpdate();
      }
    );
    // methods

    function dataConstructor(amount, fixedHeight) {
      let constructedArr = [];
      for (let i = 0; i < Number(amount); i++) {
        let constructedObj = {};
        constructedObj["height"] = fixedHeight ? 50 : randomInteger(30, 90);
        constructedObj["id"] = i;
        constructedObj["color"] = "#" + randomColor();
        constructedArr.push(constructedObj);
      }
      return constructedArr;
    }

    function randomColor() {
      return Math.floor(Math.random() * 16777215).toString(16);
    }

    function randomInteger(min, max) {
      return Math.floor(Math.random() * (max - min + 1)) + min;
    }
    return {
      ...toRefs(state),
      dataConstructor,
      randomColor,
      randomInteger,
    };
  },
};
</script>

<style scoped>
span,
div {
  font-family: "Helvetica Neue", sans-serif;
}

.header {
  font-family: "Helvetica Neue", sans-serif;
  font-size: 25px;
  text-align: center;
}

.userOptions {
  margin: 10px 0;
}

.option {
  margin: 0 7px;
}

.footer {
  font-size: 10px;
  text-align: center;
}

.btn {
  cursor: pointer;
}
</style>