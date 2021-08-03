<template>
  <div id="app">
    <!-- <BpmnModeler ref='modeler' v-model="modeler" :diagramXML="propXmlData" propertiesPanel></BpmnModeler> -->
    <BpmnViewer :xmlData="viewData" :taskData="propTaskList"></BpmnViewer>
    <button @click="addTask" class="button" v-if="$refs.modeler">
      add task
    </button>
  </div>
</template>
<script>
import demo from "./demo.bpmn";
import view from "./diagram_2.bpmn";
export default {
  data() {
    return {
      propXmlData: demo,
      viewData: view,
      /**
       * activityInstanceState
       * 0-默认(开始但未处理)
       * 1- 全局完成(一般是整个流程据结束)
       * 2- 取消
       * 3- 开始(一般是签收后等)
       * 4- 完成(一般是当前任务处理完成)
       */
      propTaskList: [
        {
          activityId: "StartEvent_1",
          activityInstanceState: 4,
        },
        {
          activityId: "Activity_0qoitk1",
          activityInstanceState: 4,
        },
        {
          activityId: "Gateway_1e7j2lk",
          activityInstanceState: 4,
        },
        {
          activityId: "Activity_03altrj",
          activityInstanceState: 2,
        },
        {
          activityId: "Activity_1l44qhl",
          activityInstanceState: 4,
        },
        {
          activityId: "Activity_155q03i",
          activityInstanceState: 3,
        },
         {
          activityId: "qqqq",
          activityInstanceState: 4,
        },
         {
          activityId: "Activity_024b1ei",
          activityInstanceState: 0,
        },
      ],
      modeler: {
        xmlData: "",
        svgImage: "",
      },
    };
  },
  watch: {
    modeler(val) {
      console.log(val);
    },
  },
  methods: {
    test(list) {
      console.log(list);
    },
    addTask() {
      let taskList = [];
      if (Math.round(Math.random()) === 0) {
        taskList = [
          {
            label: "test task1",
          },
          {
            label: "通用节点",
          },
        ];
      } else {
        taskList = [
          // {
          //   label: 'test task1'
          // }
        ];
      }
      let addOrReplace = {
        replaceActivity: "UserTask_0hkfnx2",
        taskList: taskList,
      };
      this.$refs.modeler.replace(addOrReplace).then((data) => {
        // new task list
        console.log(data);
      });
    },
  },
};
</script>
<style lang="less">
.button {
  position: absolute;
  right: 15px;
  bottom: 100px;
  height: 40px;
  width: 80px;
  z-index: 100;
  color: green;
  background: rgba(0, 128, 0, 0.2);
  border: 1px solid rgb(0, 128, 0);
  border-radius: 4px;
}
</style>
