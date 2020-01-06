<template>
  <div class="home">
    <div class="leftmenu">
      <CheckboxGroup v-model="checkedItems" @on-change="checkBoxChange">
        <Checkbox
          v-for="item in items"
          :key="item.id"
          :label="item.label"
          border
        ></Checkbox>
      </CheckboxGroup>
      <Button type="primary" @click="saveLayout">保存布局</Button>
    </div>
    <div class="main">
      <grid-layout
        :layout.sync="layout"
        :col-num="12"
        :row-height="30"
        :is-draggable="true"
        :is-resizable="true"
        :is-mirrored="false"
        :vertical-compact="true"
        :margin="[10, 10]"
        :use-css-transforms="true"
      >
        <grid-item
          v-for="item in layout"
          :x="item.x"
          :y="item.y"
          :w="item.w"
          :h="item.h"
          :i="item.i"
          :key="item.i"
        >
          <Card :bordered="false" class="iframeCard">
            <p slot="title">{{ item.label }}</p>
            <iframe
              :src="item.url"
              frameborder="0"
              width="100%"
              height="100%"
            ></iframe>
          </Card>
        </grid-item>
      </grid-layout>
    </div>
  </div>
</template>

<script>
import VueGridLayout from "vue-grid-layout";
import * as _ from "lodash";
export default {
  name: "home",
  components: {
    GridLayout: VueGridLayout.GridLayout,
    GridItem: VueGridLayout.GridItem
  },
  data() {
    return {
      // 基础模块数据
      items: [
        {
          // 模块id
          id: 1,
          // 模块名称
          label: "百度",
          // 模块url
          url: "https://www.baidu.com/"
        },
        {
          id: 2,
          label: "新浪",
          url: "https://www.sina.com.cn/"
        },
        {
          id: 3,
          label: "搜狐",
          url: "http://www.sohu.com/"
        }
      ],
      // 当前选中的模块
      checkedItems: [],
      // 前一次选中的模块
      checkedItemsChangedBefore: [],
      // 布局
      layout: []
    };
  },
  created() {
    this.loadLayoutData();
  },
  methods: {
    // 初始化加载布局数据
    loadLayoutData() {
      // 后台接口加载的布局数据
      this.layout = [
        {
          x: 0,
          y: 0,
          w: 6,
          h: 7,
          i: 1,
          label: "百度",
          url: "https://www.baidu.com/"
        },
        {
          x: 0,
          y: 0,
          w: 6,
          h: 7,
          i: 2,
          label: "新浪",
          url: "https://www.sina.com.cn/"
        }
      ];
      // 反显选中项
      this.layoutToSelect();
    },
    // 复选框变化时触发
    checkBoxChange(selected) {
      // 找到新增或取消的项
      if (selected.length > this.checkedItemsChangedBefore.length) {
        // 新增
        let label = _.difference(selected, this.checkedItemsChangedBefore)[0];
        let item = _.find(this.items, { label });
        this.addLayout(item);
      } else {
        // 移除
        let label = _.difference(this.checkedItemsChangedBefore, selected)[0];
        let item = _.find(this.items, { label });
        this.delLayout(item);
      }
      this.checkedItemsChangedBefore = selected;
    },
    // 添加模块，用于选中项新增时动态添加布局。
    addLayout(item) {
      this.layout.push({
        x: 0,
        y: 0,
        w: 6,
        h: 7,
        i: item.id,
        label: item.label,
        url: item.url
      });
    },
    // 移除模块，用于选中项取消时删除布局。
    delLayout(item) {
      _.remove(this.layout, function(v) {
        return v.i === item.id;
      });
    },
    // 布局转化为选中项，用于初始化数据时选中项反显。
    layoutToSelect() {
      this.checkedItems = _.map(this.layout, "label");
      this.checkedItemsChangedBefore = this.checkedItems;
    },
    // 保存布局
    saveLayout() {
      // 调用后台接口保存布局
      const layout = JSON.stringify(this.layout);
      alert("保存的布局为：" + layout);
      let layoutForPortal = this.layout.map(v => {
        v.leftOffset = (v.x / 12) * 100 + "%";
        v.topOffset = v.y * 30 + "px";
        v.width = (v.w / 12) * 100 + "%";
        v.height = v.h * 30 + "px";
        return v;
      });
      alert("输出给门户的布局信息为:" + JSON.stringify(layoutForPortal));
    }
  }
};
</script>

<style scoped>
.home {
  width: 100vw;
  height: 100vh;
  overflow-y: auto;
  position: relative;
}
.leftmenu {
  width: 100px;
  height: 100%;
  position: absolute;
  top: 0;
  left: 0;
}
.main {
  min-height: 300px;
  margin: 10px 10px 10px 100px;
  border: 1px dashed #666;
  padding: 10px;
  background: #eee;
}
.iframeCard {
  height: 100%;
  overflow: hidden;
}
</style>
<style>
.iframeCard .ivu-card-head {
  position: absolute;
  top: 0;
  width: 100%;
}
.iframeCard .ivu-card-body {
  height: 100%;
  padding-top: 50px;
}
</style>
