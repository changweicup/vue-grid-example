<template>
  <div class="row">
    <div class="col-3 block-menu">
      <el-scrollbar wrap-style="height: 100vh;">

        <div class="list-group-item-menu" @drag="drag(element)" v-for="element in list1" :key="element.id"
          @dragend="dragend(element)" draggable="true" unselectable="on">
          {{ element.component }}
        </div>
      </el-scrollbar>

    </div>
    <div id="content" class="content" style="height: 100vh">
      <div>
        {{layout}}
      </div>
      <grid-layout ref="gridlayout" :layout.sync="layout" :col-num="24" :row-height="20" :is-draggable="true"
        :is-resizable="false" :vertical-compact="true" :use-css-transforms="true">
        <grid-item :key="item.i" v-for="item in layout" :x="item.x" :y="item.y" :w="item.w" :h="item.h" :i="item.i">
          <el-button type="primary" v-if="item.component === 'button'">Primary</el-button>
          <el-table :data="tableData" v-if="item.component === 'table'">
            <el-table-column prop="date" label="Date" width="180" />
            <el-table-column prop="name" label="Name" width="180" />
            <el-table-column prop="address" label="Address" />
          </el-table>
        </grid-item>
      </grid-layout>
    </div>
  </div>
</template>

<script>
import { GridLayout, GridItem } from "vue-grid-layout"
let mouseXY = { "x": null, "y": null };
let DragPos = { "x": null, "y": null, "w": null, "h": null, "i": null };
export default {
  components: {
    GridLayout,
    GridItem
  },
  data () {
    return {
      list1: [
        { "x": 0, "y": 0, "w": 24, "h": 9, "i": "0", component: 'table' },
        { "x": 0, "y": 0, "w": 2, "h": 4, "i": "1", component: 'button' },
        { "x": 0, "y": 0, "w": 2, "h": 5, "i": "2", component: 'input' },
        { "x": 0, "y": 0, "w": 2, "h": 3, "i": "3", component: 'chart' },
      ],
      tableData: [
        {
          date: '2016-05-03',
          name: 'Tom',
          address: 'No. 189, Grove St, Los Angeles',
        },
        {
          date: '2016-05-02',
          name: 'Tom',
          address: 'No. 189, Grove St, Los Angeles',
        },
        {
          date: '2016-05-04',
          name: 'Tom',
          address: 'No. 189, Grove St, Los Angeles',
        },
        {
          date: '2016-05-01',
          name: 'Tom',
          address: 'No. 189, Grove St, Los Angeles',
        },
      ],
      layout: [],
    }
  },
  mounted () {
    document.addEventListener("dragover", function (e) {
      mouseXY.x = e.clientX;
      mouseXY.y = e.clientY;
    }, false);
  },
  beforeDestroy () {
  },
  methods: {
    drag: function (e) {
      console.log(e);
      let parentRect = document.getElementById('content').getBoundingClientRect();
      let mouseInGrid = false;
      if (((mouseXY.x > parentRect.left) && (mouseXY.x < parentRect.right)) && ((mouseXY.y > parentRect.top) && (mouseXY.y < parentRect.bottom))) {
        mouseInGrid = true;
      }
      if (mouseInGrid === true && (this.layout.findIndex(item => item.i === 'drop')) === -1) {
        this.layout.push({
          x: (this.layout.length * 2) % 24,
          y: this.layout.length + 24, // puts it at the bottom
          w: e.w,
          h: e.h,
          i: 'drop',
        });
      }
      let index = this.layout.findIndex(item => item.i === 'drop');
      if (index !== -1) {
        try {
          this.$refs.gridlayout.$children[this.layout.length].$refs.item.style.display = "none";
        } catch {
          console.log(1231);
        }
        let el = this.$refs.gridlayout.$children[index];
        el.dragging = { "top": mouseXY.y - parentRect.top, "left": mouseXY.x - parentRect.left };
        let new_pos = el.calcXY(mouseXY.y - parentRect.top, mouseXY.x - parentRect.left);
        if (mouseInGrid === true) {
          this.$refs.gridlayout.dragEvent('dragstart', 'drop', new_pos.x, new_pos.y, e.h, e.w);
          DragPos.i = String(index);
          DragPos.x = this.layout[index].x;
          DragPos.y = this.layout[index].y;
        }
        if (mouseInGrid === false) {
          this.$refs.gridlayout.dragEvent('dragend', 'drop', new_pos.x, new_pos.y, e.h, e.w);
          this.layout = this.layout.filter(obj => obj.i !== 'drop');
        }
      }
    },
    dragend: function (e) {
      let parentRect = document.getElementById('content').getBoundingClientRect();
      let mouseInGrid = false;
      if (((mouseXY.x > parentRect.left) && (mouseXY.x < parentRect.right)) && ((mouseXY.y > parentRect.top) && (mouseXY.y < parentRect.bottom))) {
        mouseInGrid = true;
      }
      if (mouseInGrid === true) {
        this.$refs.gridlayout.dragEvent('dragend', 'drop', DragPos.x, DragPos.y, e.h, e.w);
        this.layout = this.layout.filter(obj => obj.i !== 'drop');
        // UNCOMMENT below if you want to add a grid-item
        this.layout.push({
            x: DragPos.x,
            y: DragPos.y,
            w: e.w,
            h: e.h,
            i: DragPos.i,
            component: e.component
        });
        this.$refs.gridLayout.dragEvent('dragend', DragPos.i, DragPos.x,DragPos.y, e.h, e.w);
        try {
            this.$refs.gridLayout.$children[this.layout.length].$refs.item.style.display="block";
        } catch {
          console.log(22);
        }
      }
    },
  }
}
</script>

<style scoped>
.row {
  display: flex;
  box-sizing: border-box;
  height: 100%;
}
.col-3 {
  box-sizing: border-box;
  background-color: #ffffff;
  padding: 20px;
  width: 240px;
}
.content {
  box-sizing: border-box;
  width: calc(100% - 240px);
  height: 100vh;
  margin: 15px;
  padding: 15px;
  background-color: #ffffff;
}
.dragArea {
  height: 100%;
  width: 100%;
}
.block-menu {
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  align-items: center;
}
.list-group-item-menu {
  height: 150px;
  width: 180px;
  line-height: 150px;
  text-align: center;
  background: #d4b7fd;
  border-radius: 10px;
  cursor: pointer;
  margin-bottom: 20px;
}
.list-group-item-menu:hover {
  box-shadow: 0px 12px 32px 4px rgba(0, 0, 0, 0.04),
    0px 8px 20px rgba(0, 0, 0, 0.08);
}
.droppable-element {
  width: 150px;
  text-align: center;
  background: #fdd;
  border: 1px solid black;
  margin: 10px 0;
  padding: 10px;
}
.vue-grid-layout {
  background: #eee;
  height: 100% !important;
}
.vue-grid-item:not(.vue-grid-placeholder) {
  background: rgba(0, 170, 232, 0.1);
  border: 1px dashed #00AAE8;
}
/deep/.vue-grid-placeholder {
  background: rgba(0, 170, 232, 0.1) !important;
  opacity: 1;
  border: 1px dashed #00AAE8;
}
.vue-grid-item .resizing {
  opacity: 0.9;
}
.vue-grid-item .static {
  background: #cce;
}
.vue-grid-item .text {
  font-size: 24px;
  text-align: center;
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  margin: auto;
  height: 100%;
  width: 100%;
}
.vue-grid-item .no-drag {
  height: 100%;
  width: 100%;
}
.vue-grid-item .minMax {
  font-size: 12px;
}
.vue-grid-item .add {
  cursor: pointer;
}
.vue-draggable-handle {
  position: absolute;
  width: 20px;
  height: 20px;
  top: 0;
  left: 0;
  background: url("data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='10' height='10'><circle cx='5' cy='5' r='5' fill='#999999'/></svg>")
    no-repeat;
  background-position: bottom right;
  padding: 0 8px 8px 0;
  background-repeat: no-repeat;
  background-origin: content-box;
  box-sizing: border-box;
  cursor: pointer;
}
.layoutJSON {
  background: #ddd;
  border: 1px solid black;
  margin-top: 10px;
  padding: 10px;
}
.layoutJSON {
  background: #ddd;
  border: 1px solid black;
  margin-top: 10px;
  padding: 10px;
}
.columns {
  -moz-columns: 120px;
  -webkit-columns: 120px;
  columns: 120px;
}
</style>