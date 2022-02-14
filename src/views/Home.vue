<template>
  <div>
    <div class="toolbar">
      <button @click="addField('text', '文本')">增加文本</button>
      <button @click="addField('barcode', '条形码')">增加条形码</button>
      <button @click="addField('qrcode', '二维码')">增加二维码</button>
      <button @click="generateLodop">生成LODOP代码</button>
    </div>
    <div style="display: flex">
      <div class="container">
        <VueDragResize
          v-for="(comp, index) in compList"
          :key="comp.compId"
          :isActive="comp.active"
          :w="comp.w"
          :h="comp.h"
          :x="comp.x"
          :y="comp.y"
          :minw="25"
          :minh="25"
          :aspectRatio="comp.aspectRatio"
          :parent-limitation="true"
          @activated="activateEv(index)"
          @deactivated="deactivateEv(index)"
          @dragging="resize($event, index)"
          @resizing="resize($event, index)"
        >
          <div class="comp">
            <div v-if="comp.type == 'text'">
              <span :style="{ fontSize: comp.fontSize + 'px' }">{{
                comp.display
              }}</span>
              {{ comp.field }}
            </div>
            <img
              v-if="comp.type == 'barcode'"
              src="barcode.svg"
              style="height: 100%; width: 100%"
            />
            <img v-if="comp.type == 'qrcode'" src="qrcode.svg" />
          </div>
        </VueDragResize>
      </div>
      <div class="props" v-if="activeComp" @mousedown.stop>
        <h2>{{ activeComp.display }}</h2>
        <div class="prop">
          <div class="prop">
            <label>内容</label>
            <input
              type="text"
              v-model="activeComp.content"
              style="width: 160px"
            />
          </div>

          <label>字号</label>
          <select v-model="activeComp.fontSize" style="width: 160px">
            <option v-for="i in 50" :key="i" :value="i + 8">{{ i + 8 }}</option>
          </select>
        </div>

        <div class="prop">
          <label>字号</label>
          <input type="text" v-model="activeComp.fontSize" />
        </div>

        <div style="margin-top: 20px">
          <button @click="deleteComp">删除当前组件</button>
        </div>
      </div>
      <div class="code" style="margin: 10px">
        <pre>{{ this.lodopCode }}</pre>
      </div>
    </div>
  </div>
</template>

<script>
import VueDragResize from 'vue-drag-resize';

export default {
  components: {
    VueDragResize,
  },

  data() {
    return {
      compList: [],
      lodopCode: '',
    };
  },
  computed: {
    activeComp() {
      return this.compList.find((x) => x.active);
    },
  },
  methods: {
    addField(fieldType, fieldDisplay) {
      let idx = this.compList.length;
      let comp = {
        compId: 'comp' + new Date().getTime(),
        type: fieldType,
        display: fieldDisplay + idx,
        active: true,
        w: 120,
        h: 60,
        x: 10 * idx,
        y: 10 * idx,
        z: idx + 1,
        field: '',
        content: fieldDisplay,
        aspectRatio: false,
        fontSize: 14,
      };
      if (fieldType == 'qrcode') {
        comp.w = 150;
        comp.h = 150;
        comp.aspectRatio = true;
      }
      if (fieldType == 'text') {
        comp.h = 25;
        comp.style = {
          fontName: '',
          fontSize: '14pt',
          fontWeight: 'normal',
          textAlign: 'left',
        };
      }
      this.compList.push(comp);
    },
    deleteComp() {
      if (!this.activeComp) {
        return;
      }
      let pos = this.compList.findIndex((x) => x === this.activeComp);
      if (pos >= 0) {
        this.compList.splice(pos, 1);
      }
    },
    generateLodop() {
      let code = [];
      for (let comp of this.compList) {
        if (comp.type == 'text') {
          code.push(`LODOP.SET_PRINT_STYLEA(0,"FontSize",${comp.fontSize});`);
          code.push(
            `LODOP.ADD_PRINT_TEXT(${comp.t},${comp.l},${comp.w},${comp.h}, "${comp.display}");`
          );
        }
        if (comp.type == 'barcode') {
          code.push(
            `LODOP.ADD_PRINT_BARCODE(${comp.t},${comp.l},${comp.w},${comp.h}, "128B", "${comp.field}");`
          );
        }
        if (comp.type == 'qrcode') {
          code.push(
            `LODOP.ADD_PRINT_BARCODE(${comp.t},${comp.l},${comp.w},${comp.h}, "qrcode", "${comp.field}");`
          );
        }
      }
      this.lodopCode = code.join('\n');
    },
    resize(newRect, index) {
      let comp = this.compList[index];
      comp.w = newRect.width;
      comp.h = newRect.height;
      comp.t = newRect.top;
      comp.l = newRect.left;
    },
    activateEv(index) {
      console.log('active', index);
      for (let i = 0; i < this.compList.length; i++) {
        let comp = this.compList[i];
        comp.active = i == index;
        // if (i == index) {
        //   comp.z = this.compList.length;
        // } else {
        //   if (comp.z > 1) {
        //     comp.z = comp.z - 1;
        //   }
        // }
      }
    },
    deactivateEv(index) {
      console.log('deactive', index);
      this.compList[index].active = false;
    },
  },
};
</script>

<style>
.container {
  position: relative;
  border: 1px solid #ccc;
  margin-top: 10px;
  height: 600px;
  width: 600px;
}

.comp {
  /* background-color: #eee; */
  width: 100%;
  height: 100%;
  display: inline-block;
  /* position: absolute; */
}

.props {
  font-size: 12px;
  margin-top: 10px;
  margin-left: 10px;
  padding: 5px;
  border: 1px solid #eee;
}

.props .prop {
  margin-bottom: 5px;
}

.prop label {
  margin: 5px;
}
</style>
