<template>
  <div class="containers">
    <div ref="canvas" class="canvas"></div>
  </div>
</template>

<script>
import BpmnViewer from 'bpmn-js/lib/NavigatedViewer';
export default {
  name: "BpmnViewer",
  props: {
    xmlData: {
      type: String,
      default: ''
    },
    taskData: {
      type: Array,
      default: () => []
    }
  },
  data() {
    return {
      taskList: []
    };
  },
  mounted() {
    this.taskList = this.taskData
    let bpmnViewer = new BpmnViewer({ container: this.$refs["canvas"] });
    let _self = this;
    bpmnViewer.importXML(this.xmlData, function(err) {
      if (err) {
        console.log('加载失败', err);
      } else {
        let canvas = bpmnViewer.get('canvas');
        canvas.zoom('fit-viewport')
        if (_self.taskList && _self.taskList.length > 0) {
          // let overlays = bpmnViewer.get('overlays');
          // let overlayHtml = document.createElement('div');
          // overlays.add('StartEvent_1', {
          //   position: {
          //     top: 0,
          //     right: 0,
          //   },
          //   html: overlayHtml
          // });
          // 判断开始节点或结束节点完成
          bpmnViewer._definitions.rootElements[0].flowElements.forEach(n => {
            if (n.$type === 'bpmn:UserTask') {
                let completeTask = _self.taskList.find(m => m.key === n.id)
                let todoTask = _self.taskList.find(m => !m.completed)
                let endTask = _self.taskList[_self.taskList.length - 1]
                if (completeTask) {
                  canvas.addMarker(n.id, completeTask.completed ? 'highlight' : 'highlight-todo');  
                  n.outgoing.forEach(nn => {
                    let targetTask = _self.taskList.find(m => m.key === nn.targetRef.id)
                    if (targetTask) {
                      canvas.addMarker(nn.id, targetTask.completed ? 'highlight' : 'highlight-todo');  
                    } else if (nn.targetRef.$type === 'bpmn:ExclusiveGateway') {
                      // canvas.addMarker(nn.id, 'highlight');
                      canvas.addMarker(nn.id, completeTask.completed ? 'highlight' : 'highlight-todo');  
                      canvas.addMarker(nn.targetRef.id, completeTask.completed ? 'highlight' : 'highlight-todo');  
                    } else if (nn.targetRef.$type === 'bpmn:EndEvent') {
                      if (!todoTask && endTask.key === n.id) {
                        canvas.addMarker(nn.id, 'highlight');  
                        canvas.addMarker(nn.targetRef.id, 'highlight');  
                      }
                      if (!completeTask.completed) {
                        canvas.addMarker(nn.id, 'highlight-todo');  
                        canvas.addMarker(nn.targetRef.id, 'highlight-todo');  
                      }
                    }
                  });
                }
            } else if (n.$type === 'bpmn:ExclusiveGateway') {
              n.outgoing.forEach(nn => {
                let targetTask = _self.taskList.find(m => m.key === nn.targetRef.id)
                if (targetTask) {
                  canvas.addMarker(nn.id, targetTask.completed ? 'highlight' : 'highlight-todo');
                }
              })
            }
            if (n.$type === 'bpmn:StartEvent') {
              n.outgoing.forEach(nn => {
                let completeTask = _self.taskList.find(m => m.key === nn.targetRef.id)
                if (completeTask) {
                  canvas.addMarker(nn.id, 'highlight');
                  canvas.addMarker(n.id, 'highlight');
                  return
                }
              });
            }
          })
        }
      }
    });
  },
  methods: {
  }
};
</script>
<style lang="less" scoped>
.containers {
  position: absolute;
  background-color: #ffffff;
  top:0;
  left: 0;
  width: 100%;
  height: 100%;
}
.canvas {
  width: 100%;
  height: 100%;
}
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
}
/deep/.highlight-todo.djs-connection > .djs-visual > path {
  stroke: orange !important;
  stroke-dasharray: 4px !important;
  fill-opacity: 0.2 !important;
}
/deep/.highlight-todo.djs-shape .djs-visual > :nth-child(1) {
  fill: orange !important; 
  stroke: orange !important;
  stroke-dasharray: 4px !important; 
  fill-opacity: 0.2 !important;
}
/deep/.overlays-div {
  font-size: 10px;
  color: red;
  width: 100px;
  top: -20px !important;
}
</style>