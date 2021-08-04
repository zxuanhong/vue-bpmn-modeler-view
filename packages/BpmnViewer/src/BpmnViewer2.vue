<template>
  <div class="containers">
    <div ref="canvas" class="canvas"></div>
  </div>
</template>

<script>
import BpmnViewer from "bpmn-js/lib/NavigatedViewer";
import {
  append as svgAppend,
  attr as svgAttr,
  create as svgCreate,
} from "tiny-svg";

import { query as domQuery } from "min-dom";
export default {
  name: "BpmnViewer",
  props: {
    xmlData: {
      type: String,
      default: "",
    },
    taskData: {
      type: Array,
      default: () => [],
    },
  },
  data() {
    return {
      taskList: [],
    };
  },
  async mounted() {
    this.taskList = this.taskData;
    let bpmnViewer = new BpmnViewer({
      container: this.$refs["canvas"],
      bpmnRenderer: {},
    });
    try {
      await bpmnViewer.importXML(this.xmlData);
      this.initArrow("sequenceflow-arrow-complete");
      this.initArrow("sequenceflow-arrow-waiting");
      let canvas = bpmnViewer.get("canvas");
      if (this.taskList && this.taskList.length > 0) {
        bpmnViewer
          .getDefinitions()
          .rootElements[0].flowElements.forEach((element) => {
            this.setBpmnViewStyle(element, canvas);
          });
      }
    } catch (err) {
      console.log("error rendering", err);
    }
  },
  methods: {
    initArrow(id) {
      const marker = svgCreate("marker");
      svgAttr(marker, {
        id,
        viewBox: "0 0 20 20",
        refX: "11",
        refY: "10",
        markerWidth: "10",
        markerHeight: "10",
        orient: "auto",
      });
      const path = svgCreate("path");
      svgAttr(path, {
        d: "M 1 5 L 11 10 L 1 15 Z",
        style:
          " stroke-width: 1px; stroke-linecap: round; stroke-dasharray: 10000, 1; ",
      });
      const defs = domQuery("defs");
      svgAppend(marker, path);
      svgAppend(defs, marker);
    },
    setBpmnViewStyle(element, canvas) {
      // 设置当前节点样式
      this.setViewScopeCompleteInfo(element, canvas);
      this.setViewCancleInfo(element, canvas);
      this.setViewEndingInfo(element, canvas);
      const type = element.$type;
      if (type === "bpmn:SubProcess") {
         this.setViewStartingInfoSubprocess(element, canvas);
        this.setViewDefaultInfoSubProcess(element, canvas);
        // 处理子流程
        element.flowElements.forEach((subElement) => {
          this.setBpmnViewStyle(subElement, canvas);
        });
      } else {
        this.setViewStartingInfo(element, canvas);
        this.setViewDefaultInfo(element, canvas);
      }
    },
    setViewDefaultInfo(element, canvas) {
      let taskInfo = this.taskList.find(
        (task) =>
          task.activityId === element.id && task.activityInstanceState == 0
      );
      if (taskInfo) {
        canvas.addMarker(element.id, "highlight-todo");
      }
    },
    setViewDefaultInfoSubProcess(element, canvas) {
      let taskInfo = this.taskList.find(
        (task) =>
          task.activityId === element.id && task.activityInstanceState == 0
      );
      if (taskInfo) {
        canvas.addMarker(element.id, "highlight-todo-subprocess");
      }
    },
    setViewScopeCompleteInfo(element, canvas) {
      let taskInfo = this.taskList.find(
        (task) =>
          task.activityId === element.id && task.activityInstanceState == 1
      );
      if (taskInfo) {
        canvas.addMarker(element.id, "highlight");
      }
    },
    setViewCancleInfo(element, canvas) {
      let taskInfo = this.taskList.find(
        (task) =>
          task.activityId === element.id && task.activityInstanceState == 2
      );
      if (taskInfo) {
        canvas.addMarker(element.id, "cancled");
      }
    },
    setViewStartingInfo(element, canvas) {
      let taskInfo = this.taskList.find(
        (task) =>
          task.activityId === element.id && task.activityInstanceState == 3
      );
      if (taskInfo) {
        canvas.addMarker(element.id, "highlight-todo");
        element.outgoing.forEach((outgo) => {
          this.setViewOutgoDefaultInfo(outgo, canvas);
          this.setViewOutgoCancleInfo(outgo, canvas);
          this.setViewOutgoStartingInfo(outgo, canvas);
        });
      }
    },
    setViewStartingInfoSubprocess(element, canvas) {
      let taskInfo = this.taskList.find(
        (task) =>
          task.activityId === element.id && task.activityInstanceState == 3
      );
      if (taskInfo) {
        canvas.addMarker(element.id, "highlight-todo-subprocess");
        element.outgoing.forEach((outgo) => {
          this.setViewOutgoDefaultInfo(outgo, canvas);
          this.setViewOutgoCancleInfo(outgo, canvas);
          this.setViewOutgoStartingInfo(outgo, canvas);
        });
      }
    },
    setViewEndingInfo(element, canvas) {
      let taskInfo = this.taskList.find(
        (task) =>
          task.activityId === element.id && task.activityInstanceState == 4
      );
      if (taskInfo) {
        canvas.addMarker(element.id, "highlight");
        element.outgoing.forEach((outgo) => {
          this.setViewOutgoDefaultInfo(outgo, canvas);
          this.setViewOutgoScopeCompleteInfo(outgo, canvas);
          this.setViewOutgoCancleInfo(outgo, canvas);
          this.setViewOutgoStartingInfo(outgo, canvas);
          this.setViewOutgoEndingInfo(outgo, canvas);
        });
      }
    },
    setViewOutgoDefaultInfo(outgo, canvas) {
      let taskInfo = this.taskList.find(
        (task) =>
          task.activityId === outgo.targetRef.id &&
          task.activityInstanceState == 0
      );
      if (taskInfo) {
        canvas.addMarker(outgo.id, "highlight-todo");
        canvas.addMarker(outgo.targetRef.id, "highlight-todo");
      }
    },
    setViewOutgoScopeCompleteInfo(outgo, canvas) {
      let taskInfo = this.taskList.find(
        (task) =>
          task.activityId === outgo.targetRef.id &&
          task.activityInstanceState == 1
      );
      if (taskInfo) {
        canvas.addMarker(outgo.id, "highlight");
        canvas.addMarker(outgo.targetRef.id, "highlight");
      }
    },
    setViewOutgoCancleInfo(outgo, canvas) {
      let taskInfo = this.taskList.find(
        (task) =>
          task.activityId === outgo.targetRef.id &&
          task.activityInstanceState == 2
      );
      if (taskInfo) {
        canvas.addMarker(outgo.id, "cancled");
        canvas.addMarker(outgo.targetRef.id, "cancled");
      }
    },
    setViewOutgoStartingInfo(outgo, canvas) {
      let taskInfo = this.taskList.find(
        (task) =>
          task.activityId === outgo.targetRef.id &&
          task.activityInstanceState == 3
      );
      if (taskInfo) {
        canvas.addMarker(outgo.id, "highlight-todo");
        canvas.addMarker(outgo.targetRef.id, "highlight-todo");
      }
    },
    setViewOutgoEndingInfo(outgo, canvas) {
      let taskInfo = this.taskList.find(
        (task) =>
          task.activityId === outgo.targetRef.id &&
          task.activityInstanceState == 4
      );
      if (taskInfo) {
        canvas.addMarker(outgo.id, "highlight");
        canvas.addMarker(outgo.targetRef.id, "highlight");
      }
    },
  },
};
</script>
<style lang="less" scoped>
.containers {
  position: absolute;
  background-color: #ffffff;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}
.canvas {
  width: 100%;
  height: 100%;
}
// 已经完成样式
/deep/.highlight.djs-shape .djs-visual > :nth-child(1) {
  fill: green !important;
  stroke: green !important;
  fill-opacity: 0.2 !important;
}
/deep/.highlight.djs-shape .djs-visual > :nth-child(2) {
  fill: green !important;
}
/deep/.highlight.djs-shape .djs-visual > path {
  fill: green !important;
  fill-opacity: 0.2 !important;

  stroke: green !important;
}
/deep/.highlight.djs-connection > .djs-visual > path {
  stroke: green !important;
  marker-end: url(#sequenceflow-arrow-complete) !important;
}
/deep/#sequenceflow-arrow-complete {
  fill: green;
  stroke: green;
}
// 已经取消样式
/deep/.cancled.djs-shape .djs-visual > :nth-child(1) {
  fill: #eb0d0d !important;
  stroke: #eb0d0d !important;
  fill-opacity: 0.2 !important;
}
/deep/.cancled.djs-shape .djs-visual > :nth-child(2) {
  fill: #eb0d0d !important;
}
/deep/.cancled.djs-shape .djs-visual > path {
  fill: #eb0d0d !important;
  fill-opacity: 0.2 !important;
  stroke: #eb0d0d !important;
}
/deep/.cancled.djs-connection > .djs-visual > path {
  stroke: #eb0d0d !important;
  marker-end: url(#sequenceflow-arrow-cancled) !important;
}
/deep/#sequenceflow-arrow-cancled {
  fill: #eb0d0d;
  stroke: #eb0d0d;
}
// 待审批样式
/deep/.highlight-todo.djs-connection > .djs-visual > path {
  stroke: orange !important;
  fill-opacity: 0.2 !important;
  marker-end: url(#sequenceflow-arrow-waiting) !important;
  stroke-dasharray: 4;
  animation: highlight-todo 35s linear infinite;
}
/deep/#sequenceflow-arrow-waiting {
  fill: orange;
  stroke: orange;
}
/deep/.highlight-todo.djs-shape .djs-visual > :nth-child(1) {
  fill: orange !important;
  stroke: orange !important;
  fill-opacity: 0.2 !important;
  stroke-dasharray: 4;
  animation: highlight-todo 35s linear infinite;
}
// 待审批子流程
/deep/.highlight-todo-subprocess.djs-connection > .djs-visual > path {
  stroke: rgb(235, 99, 9) !important;
  fill-opacity: 0.2 !important;
  marker-end: url(#sequenceflow-arrow-waiting-subprocess) !important;
  stroke-dasharray: 4;
  animation: highlight-todo 35s linear infinite;
}
/deep/#sequenceflow-arrow-waiting-subprocess {
  fill: rgb(235, 99, 9);
  stroke: rgb(235, 99, 9);
}
/deep/.highlight-todo-subprocess.djs-shape .djs-visual > :nth-child(1) {
  fill: rgb(235, 99, 9) !important;
  stroke: rgb(235, 99, 9) !important;
  fill-opacity: 0.2 !important;
  stroke-dasharray: 4;
  animation: highlight-todo-subprocess 1.6s ease infinite;
}
// 旋转动画
@keyframes highlight-todo {
  to {
    stroke-dashoffset: -1000;
  }
}
@keyframes highlight-todo-subprocess {
  0% {
    opacity: 1;
  }
  50% {
    transform: scale(1.03);
    opacity: 0.8;
  }
  to {
    opacity: 1;
  }
}
</style>
