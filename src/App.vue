<template>
  <div id="app">
    <div id="mountNode"></div>
    <!-- modal1：{{modal1}} -->
    <Modal v-model="modal1" title="Common Modal dialog box title" @on-ok="addInfo">
      <p slot="header"></p>
      <Button @click="info('next')">下一岔口</Button>
      <Button @click="info('output')">下一出货口</Button>
      <p slot="footer"></p>
    </Modal>

    <Modal v-model="modal2" title="Common Modal dialog box title" @on-ok="update">
      <Input type="text" v-model="model.label"/>
      <Input type="text" v-model="model.camera"/>
      <Select @on-change="updateDirection" v-model="model.direction">
        <Option v-for="item in direction" :key="item.value" :value="item.value" :label="item.name"></Option>
      </Select>
    </Modal>

    <Button @click="getInfo">get info</Button>
  </div>
</template>

<script>
import G6 from "@antv/g6";

export default {
  data() {
    return {
      direction: [
        {
          value: 1,
          name: "上"
        },
        {
          value: 2,
          name: "右"
        },
        {
          value: 3,
          name: "左"
        },
        {
          value: 4,
          name: "右上"
        },
        {
          value: 5,
          name: "右下"
        },
        {
          value: 6,
          name: "左下"
        },
        {
          value: 7,
          name: "左上"
        }
      ],
      modal1: false,
      modal2: false,
      currentInfo: "next",
      data: {
        nodes: [
          {
            x: 100,
            y: 100,
            shape: "rect",
            label: "卸货口",
            id: "1"
          },
          {
            x: 100,
            y: 200,
            shape: "circle",
            label: "选择岔口",
            id: "plus",
            camera: "摄像头"
          }
        ],
        edges: [
          {
            shape: "line",
            source: "1",
            target: "plus"
          }
        ]
      },
      graph: null,
      model: {
        camera: "",
        name: "",
        direction: 1
      },
      edge: {
        x: 0,
        y: 0
      }
    };
  },
  watch: {
    modal1(val) {
      if (val) {
        this.currentInfo = "next";
      }
    }
  },
  methods: {
    updateDirection() {
      const { x, y } = this.model;
      const n = 100;
      let direction = [x, y];
      // console.log(direction);
      console.log(this.model.direction);
      switch (this.model.direction) {
        case 1:
          direction = [x, y + n];
          break;
        case 2:
          direction = [x + n, y];
          break;
        case 3:
          direction = [x - n, y];
          break;
        case 4:
          direction = [x + (1 / 2) * n, y + (1 / 2) * n];
          break;
        case 5:
          direction = [x + (1 / 2) * n, y - (1 / 2) * n];
          break;
        case 6:
          direction = [x - (1 / 2) * n, y - (1 / 2) * n];
          break;
        case 7:
          direction = [x - (1 / 2) * n, y + (1 / 2) * n];
          break;
        default:
          direction = [x, y];
      }
      console.log(direction);
      this.model.x = direction[0];
      this.model.y = direction[1];
    },
    getInfo() {
      console.log(this.graph && this.graph.save());
    },
    update() {
      this.modal2 = false;
      // console.log(this.model);
      const item = this.graph.findById(this.model.id);
      this.graph.update(item, {
        x: this.model.x,
        y: this.model.y
      });
    },
    addInfo() {
      this.info(this.currentInfo);
    },
    info(tag) {
      this.currentInfo = tag;
      this.modal1 = false;
      tag === "output"
        ? this.addOutput({ label: `出货口` })
        : this.addOutput({ label: `分岔口` });
    },
    addOutput({ label }) {
      // console.log(this.model);
      // const index = this.data.nodes.length;
      const index = this.graph && this.graph.save().nodes.length;
      const lastNode = this.data.nodes[index - 1];
      const nowNodeId = `item${index}`;

      const node = {
        x: this.model.x,
        y: this.model.y,
        shape: "circle",
        label: label + index,
        id: nowNodeId
      };

      const edge = {
        shape: "line",
        source: this.model.id,
        target: nowNodeId
      };

      this.graph.add("node", node);
      this.graph.add("edge", edge);

      this.model = node;
      this.edge = edge;

      this.modal2 = true;
    }
  },
  mounted() {
    const _this = this;

    this.graph = new G6.Graph({
      container: "mountNode",
      width: 500,
      height: 500
    });

    // 封装点击添加边的交互
    G6.registerBehavior("click-add-edge", {
      getEvents() {
        return {
          "node:click": "onClick"
          // mousemove: "onMousemove",
          // "edge:click": "onEdgeClick" // 点击空白处，取消边
        };
      },
      onEdgeClick(ev) {
        const currentEdge = ev.item;
        // 拖拽过程中，点击会点击到新增的边上
        if (this.addingEdge && this.edge == currentEdge) {
          _this.graph.removeItem(this.edge);
          this.edge = null;
          this.addingEdge = false;
        }
      },
      onClick(ev) {
        const node = ev.item;
        const point = { x: ev.x, y: ev.y };
        const model = node.getModel();
        _this.model = model;

        if (model.id === "1") {
          // alert("卸货口");
          // this.addingEdge = false;
          return false;
        } else if (model.label.includes("出货口")) {
          // 出货口
          return false;
        } else {
          // 弹框选择添加下一岔口或出货口
          _this.modal1 = true;

          this.addingEdge = false;
          return false;
        }

        // 如果在添加边的过程中，再次点击另一个节点，结束边的添加
        // if (this.addingEdge && this.edge) {
        //   _this.graph.updateItem(this.edge, {
        //     target: model.id
        //   });
        //   this.edge = null;
        //   this.addingEdge = false;
        // } else {
        //   // 点击节点，触发增加边
        //   this.edge = _this.graph.addItem("edge", {
        //     source: model.id,
        //     target: point
        //   });
        //   this.addingEdge = true;
        // }
      },
      onMousemove(ev) {
        const point = { x: ev.x, y: ev.y };
        if (this.addingEdge && this.edge) {
          // 增加边的过程中，移动时边跟着移动
          _this.graph.updateItem(this.edge, {
            target: point
          });
        }
      }
    });

    // 封装点击添加边的交互
    G6.registerBehavior("click-add-node", {
      getEvents() {
        return {
          "canvas:click": "onClick"
        };
      },
      onClick(ev) {
        const node = _this.graph.addItem("node", {
          x: ev.x,
          y: ev.y,
          id: G6.Util.uniqueId()
        });
        _this.graph.setItemState(node, "selected", true); // 添加后默认选中
      }
    });

    // console.log(graph);

    _this.graph.addBehaviors("click-add-edge", "default");
    // graph.addBehaviors('click-add-node', 'default');

    _this.graph.data(this.data);
    _this.graph.render();
  }
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.g6-tooltip {
  padding: 10px 6px;
  color: #444;
  background-color: rgba(255, 255, 255, 0.9);
  border: 1px solid #e2e2e2;
  border-radius: 4px;
}
</style>
