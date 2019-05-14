<template>
  <div id="mountNode"></div>
</template>

<script>
import G6 from "@antv/g6";
export default {
  data() {
    return {
      data: {
        nodes: [
          {
            x: 100,
            y: 100,
            shape: "circle",
            label: "circle"
          },
          {
            x: 200,
            y: 100,
            shape: "rect",
            label: "rect"
          },
          {
            x: 300,
            y: 100,
            size: [60, 30],
            shape: "ellipse",
            label: "ellipse"
          },
          {
            x: 400,
            y: 100,
            shape: "image",
            img:
              "https://gw.alipayobjects.com/zos/rmsportal/XuVpGqBFxXplzvLjJBZB.svg",
            label: "image"
          }
        ]
      }
    };
  },
  mounted() {
    const graph = new G6.Graph({
      container: "mountNode",
      width: 500,
      height: 500
    });
    graph.data(this.data);
    graph.render();
  }
};
</script>

