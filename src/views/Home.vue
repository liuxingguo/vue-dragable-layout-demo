<!-- @format -->

<template>
  <div class="home">
    <div class="leftmenu">
      <Divider>模板</Divider>
      <RadioGroup v-model="currentTemplate" @on-change="templateChange">
        <Radio label="1:2" style="margin:8px;" border></Radio>
        <Radio label="2:1" style="margin:8px;" border></Radio>
        <Radio label="1:1:1" style="margin:8px;" border></Radio>
        <Radio label="自由" style="margin:8px;" border></Radio>
      </RadioGroup>
      <Divider>组件</Divider>
      <CheckboxGroup v-model="checkedItems" @on-change="checkBoxChange">
        <Checkbox
          v-for="item in items"
          :key="item.id"
          :label="item.label"
          border
          style="margin:8px;"
        ></Checkbox>
      </CheckboxGroup>
      <Button type="primary" @click="saveLayout">保存布局</Button>
    </div>
    <div class="main">
      <grid-layout
        class="divider"
        :class="dividerStyle"
        :layout.sync="layout"
        :col-num="colNum"
        :row-height="rowHeight"
        :is-draggable="true"
        :is-resizable="true"
        :is-mirrored="false"
        :vertical-compact="true"
        :margin="[10, 10]"
        :use-css-transforms="true"
        :drag-transform="dragTransform"
      >
        <grid-item
          v-for="item in layout"
          :x="item.x"
          :y="item.y"
          :w="item.w"
          :h="item.h"
          :i="item.i"
          :key="item.i"
          :resizeAxis="item.resizeAxis"
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
      // 布局列数
      colNum: 12,
      // 布局行高
      rowHeight: 30,
      // 选择的布局模板
      currentTemplate: "1:2",
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
  computed: {
    dividerStyle() {
      if (this.currentTemplate === "1:2") {
        return "divider1-2";
      } else if (this.currentTemplate === "2:1") {
        return "divider2-1";
      } else if (this.currentTemplate === "1:1:1") {
        return "divider1-1-1";
      } else {
        return "";
      }
    }
  },
  methods: {
    // 初始化加载布局数据
    loadLayoutData() {
      // 后台接口加载的布局数据
      this.layout = [
        {
          x: 0,
          y: 0,
          w: 4,
          h: 6,
          i: 1,
          resizeAxis: "y",
          label: "百度",
          url: "https://www.baidu.com/"
        },
        {
          x: 4,
          y: 0,
          w: 8,
          h: 6,
          i: 2,
          resizeAxis: "y",
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
    // 模板变化时触发
    templateChange() {
      this.translateByTemplate();
    },
    // 添加模块，用于选中项新增时动态添加布局。
    addLayout(item) {
      const template = this.currentTemplate;
      switch (template) {
        case "1:2":
        case "1:1:1":
          this.layout.push({
            x: 0,
            y: 0,
            w: this.colNum / 3,
            h: 6,
            resizeAxis: "y",
            i: item.id,
            label: item.label,
            url: item.url
          });
          break;
        case "2:1":
          this.layout.push({
            x: 0,
            y: 0,
            w: this.colNum * (2 / 3),
            h: 6,
            resizeAxis: "y",
            i: item.id,
            label: item.label,
            url: item.url
          });
          break;
        default:
          this.layout.push({
            x: 0,
            y: 0,
            w: this.colNum / 3,
            h: 6,
            resizeAxis: "xy",
            i: item.id,
            label: item.label,
            url: item.url
          });
      }
    },
    // 移除模块，用于选中项取消时删除布局。
    delLayout(item) {
      _.remove(this.layout, function(v) {
        return v.i === item.id;
      });
    },
    // 布局转化为选中项，用于初始化数据时左侧选中项反显。
    layoutToSelect() {
      this.checkedItems = _.map(this.layout, "label");
      this.checkedItemsChangedBefore = this.checkedItems;
    },
    // 按模板转换当前布局，用于模板改变时调整现有布局
    translateByTemplate() {
      // 组件按行列排序
      let sortedLayout = _.sortBy(this.layout, ["y", "x"]);
      let template = this.currentTemplate;
      sortedLayout.forEach((item, index) => {
        switch (template) {
          case "1:2":
            item.x = (index % 2) * (this.colNum / 3);
            item.y = Math.floor(index / 2);
            item.w = (this.colNum / 3) * ((index % 2) + 1);
            item.resizeAxis = "y";
            break;
          case "2:1":
            item.x = (index % 2) * ((this.colNum / 3) * 2);
            item.y = Math.floor(index / 2);
            item.w = index % 2 === 0 ? (this.colNum / 3) * 2 : this.colNum / 3;
            item.resizeAxis = "y";
            break;
          case "1:1:1":
            item.x = (index % 3) * (this.colNum / 3);
            item.y = Math.floor(index / 3);
            item.w = this.colNum / 3;
            item.resizeAxis = "y";
            break;
          default:
            item.resizeAxis = "xy";
        }
      });
      this.layout = sortedLayout;
    },
    // 拖拽时根据布局调整组件位置和大小
    dragTransform(x, y, h, w) {
      // template
      switch (this.currentTemplate) {
        case "1:2":
          if (x < this.colNum / 3 / 2) {
            x = 0;
            w = this.colNum / 3;
          } else {
            x = this.colNum / 3;
            w = (this.colNum * 2) / 3;
          }
          break;
        case "2:1":
          if (x < this.colNum / 3) {
            x = 0;
            w = (this.colNum * 2) / 3;
          } else {
            x = (this.colNum * 2) / 3;
            w = this.colNum / 3;
          }
          break;
        case "1:1:1":
          if (x < this.colNum / 3 / 2) {
            x = 0;
          } else if (x < (this.colNum * 2) / 3) {
            x = this.colNum / 3;
          } else {
            x = (this.colNum * 2) / 3;
          }
          break;
        default:
          break;
      }
      return { x, y, h, w };
    },
    // 保存布局
    saveLayout() {
      alert("打开控制台查看！");
      // 调用后台接口保存布局 全部的文本保存起来，加载的时候再转json使用
      const layout = JSON.stringify(this.layout);
      window.console.log(layout);
      // 输出给门户使用的布局数据根据x、y、w、h计算，注意组件间的间距为10px要计算在内。
      // 比如行高为30px, h = 12, 则高度为：30*12+10*(12-1)
    }
  }
};
</script>

<style scoped>
.divider::before,
.divider::after {
  content: "";
  display: none;
  position: absolute;
  width: 0;
  top: 0;
  bottom: 0;
  border-left: 1px dashed #666;
}
.divider1-2::before {
  display: block;
  left: 33.333%;
}
.divider2-1::before {
  display: block;
  left: 66.667%;
}
.divider1-1-1::before {
  display: block;
  left: 33.333%;
}
.divider1-1-1::after {
  display: block;
  right: 33.333%;
}
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
  text-align: center;
}
.main {
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
