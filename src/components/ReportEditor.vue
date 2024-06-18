<template>
  <div>
    <div>
      <label for="font-select">フォントを選択:</label>
      <select id="font-select" v-model="selectedFont" @change="applyFont">
        <option v-for="font in availableFonts" :key="font" :value="font">{{ font }}</option>
      </select>
      <label for="font-size">フォントサイズ:</label>
      <input id="font-size" type="number" v-model="fontSize" @input="applyFontSize" />
      <label for="font-color">フォントカラー:</label>
      <input id="font-color" type="color" v-model="fontColor" @input="applyFontColor" />
    </div>
    <div>
      <label>テキストの横方向配置:</label>
      <select v-model="textAlign" @change="applyTextAlign">
        <option value="left">左寄せ</option>
        <option value="center">中央</option>
        <option value="right">右寄せ</option>
      </select>
      <label>テキストの縦方向配置:</label>
      <select v-model="verticalAlign" @change="applyVerticalAlign">
        <option value="top">上端</option>
        <option value="middle">中央</option>
        <option value="bottom">下端</option>
      </select>
    </div>
    <div>
      <label for="layout-name">レイアウト名:</label>
      <input id="layout-name" v-model="layoutName" />
      <button @click="saveLayout">レイアウトを保存</button>
      <label for="load-layout">レイアウトを読み込む:</label>
      <select id="load-layout" v-model="selectedLayout" @change="loadLayout">
        <option v-for="layout in savedLayouts" :key="layout.name" :value="layout.name">{{ layout.name }}</option>
      </select>
    </div>
    <div id="report" ref="report">
      <vue-draggable-resizable
        v-for="(element, index) in elements"
        :key="element.id"
        :x="element.style.left"
        :y="element.style.top"
        :w="element.style.width"
        :h="element.style.height"
        :style="{
          fontSize: element.style.fontSize,
          fontFamily: element.style.fontFamily,
          color: element.style.color,
          textAlign: element.style.textAlign,
          display: 'flex',
          alignItems: element.style.alignItems,
          justifyContent: element.style.justifyContent
        }"
        @dragging="(x, y) => updateElementPosition(index, x, y)"
        @resizing="(w, h) => updateElementSize(index, w, h)"
        @click="selectElement(index)"
      >
        <div contenteditable="true" :style="{ flexGrow: 1, textAlign: element.style.textAlign }" @focus="selectElement(index)">
          {{ element.content }}
        </div>
        <button class="remove-button" @click="removeElement(index)">削除</button>
      </vue-draggable-resizable>
    </div>
    <button @click="addText">テキストを追加</button>
    <button @click="generatePDF">PDFを生成</button>
    <button @click="printCustom">プレビューと印刷</button>
  </div>
</template>

<script>
import jsPDF from 'jspdf';
import html2canvas from 'html2canvas';
import VueDraggableResizable from 'vue-draggable-resizable';
import 'vue-draggable-resizable/dist/style.css';

export default {
  components: {
    VueDraggableResizable,
  },
  data() {
    return {
      availableFonts: [
        'Arial', 'Hiragino Mincho Pro', 'ヒラギノ明朝', 'Courier New', 'Georgia', 'Garamond', 'Comic Sans MS'
      ],
      selectedFont: 'Arial',
      fontSize: 12,
      fontColor: '#000000',
      textAlign: 'left',
      verticalAlign: 'top',
      elements: [
        { id: 1, content: 'テキスト1', style: { left: 10, top: 10, width: 100, height: 50, fontSize: '12px', color: '#000000', fontFamily: 'Arial', textAlign: 'left', alignItems: 'flex-start', justifyContent: 'flex-start' } },
      ],
      nextId: 2,
      selectedElementIndex: null,
      layoutName: '',
      selectedLayout: '',
      savedLayouts: []
    };
  },
  mounted() {
    this.loadSavedLayouts();
  },
  methods: {
    applyFont() {
      if (this.selectedElementIndex !== null) {
        this.elements[this.selectedElementIndex].style.fontFamily = this.selectedFont;
      }
    },
    applyFontSize() {
      if (this.selectedElementIndex !== null) {
        this.elements[this.selectedElementIndex].style.fontSize = `${this.fontSize}px`;
      }
    },
    applyFontColor() {
      if (this.selectedElementIndex !== null) {
        this.elements[this.selectedElementIndex].style.color = this.fontColor;
      }
    },
    applyTextAlign() {
      if (this.selectedElementIndex !== null) {
        this.elements[this.selectedElementIndex].style.textAlign = this.textAlign;
      }
    },
    applyVerticalAlign() {
      if (this.selectedElementIndex !== null) {
        this.elements[this.selectedElementIndex].style.alignItems = this.verticalAlign === 'top' ? 'flex-start' : this.verticalAlign === 'middle' ? 'center' : 'flex-end';
      }
    },
    addText() {
      this.elements.push({ id: this.nextId++, content: '新しいテキスト', style: { left: 50, top: 50, width: 100, height: 50, fontSize: '12px', color: '#000000', fontFamily: 'Arial', textAlign: 'left', alignItems: 'flex-start', justifyContent: 'flex-start' } });
    },
    removeElement(index) {
      this.elements.splice(index, 1);
      this.selectedElementIndex = null;
    },
    updateElementPosition(index, x, y) {
      this.elements[index].style.left = x;
      this.elements[index].style.top = y;
    },
    updateElementSize(index, width, height) {
      this.elements[index].style.width = width;
      this.elements[index].style.height = height;
    },
    async generatePDF() {
      const report = this.$refs.report;
      if (report) {
        // 一時的に削除ボタンと枠線を非表示にする
        const elements = report.querySelectorAll('.draggable, .remove-button');
        elements.forEach(el => {
          el.classList.add('no-border');
        });

        const canvas = await html2canvas(report, {
          scale: 2,
          ignoreElements: (element) => element.classList.contains('remove-button')
        });
        const imgData = canvas.toDataURL('image/png');
        const pdf = new jsPDF('p', 'mm', 'a4');
        const pdfWidth = pdf.internal.pageSize.getWidth();
        const pdfHeight = pdf.internal.pageSize.getHeight();
        pdf.addImage(imgData, 'PNG', 0, 0, pdfWidth, pdfHeight);
        pdf.save('report.pdf');

        // 元に戻す
        elements.forEach(el => {
          el.classList.remove('no-border');
        });
      }
    },
    printCustom() {
      const reportContent = this.$refs.report.innerHTML;
      const printWindow = window.open('', '_blank', 'width=800,height=600');
      printWindow.document.write(`
        <html>
          <head>
            <title>印刷プレビュー</title>
            <style>
              @media print {
                @page {
                  margin: 0;
                }
                body {
                  margin: 0;
                  -webkit-print-color-adjust: exact;
                }
                .remove-button {
                  display: none;
                }
                .vue-draggable-resizable {
                  border: none;
                }
              }
              body {
                font-family: ${this.selectedFont};
                padding: 20px;
              }
              .report-header {
                display: flex;
                justify-content: space-between;
                align-items: center;
                margin-bottom: 20px;
              }
              .title {
                font-size: 24px;
                font-weight: bold;
              }
              .info {
                display: flex;
                flex-direction: column;
              }
              .info-item {
                margin: 5px 0;
              }
              .report-section {
                margin-bottom: 20px;
                border: 1px solid #ccc;
                padding: 10px;
              }
              .section-header {
                font-weight: bold;
                background-color: #eee;
                padding: 5px;
              }
              .section-content {
                padding: 10px;
                min-height: 100px;
              }
            </style>
          </head>
          <body onload="window.print();window.close()">
            ${reportContent}
          </body>
        </html>
      `);
      printWindow.document.close();
    },
    selectElement(index) {
      this.selectedElementIndex = index;
      const selectedElement = this.elements[index];
      this.selectedFont = selectedElement.style.fontFamily;
      this.fontSize = parseInt(selectedElement.style.fontSize, 10);
      this.fontColor = selectedElement.style.color;
      this.textAlign = selectedElement.style.textAlign;
      this.verticalAlign = selectedElement.style.alignItems === 'flex-start' ? 'top' : selectedElement.style.alignItems === 'center' ? 'middle' : 'bottom';
    },
    saveLayout() {
      if (this.layoutName.trim() === '') {
        alert('レイアウト名を入力してください。');
        return;
      }

      const layout = {
        name: this.layoutName,
        elements: this.elements
      };

      let layouts = JSON.parse(localStorage.getItem('layouts')) || [];
      const existingLayoutIndex = layouts.findIndex(l => l.name === this.layoutName);

      if (existingLayoutIndex !== -1) {
        layouts[existingLayoutIndex] = layout;
      } else {
        layouts.push(layout);
      }

      localStorage.setItem('layouts', JSON.stringify(layouts));
      this.loadSavedLayouts();
    },
    loadSavedLayouts() {
      this.savedLayouts = JSON.parse(localStorage.getItem('layouts')) || [];
    },
    loadLayout() {
      const selectedLayout = this.savedLayouts.find(layout => layout.name === this.selectedLayout);
      if (selectedLayout) {
        this.elements = selectedLayout.elements;
        this.nextId = this.elements.length > 0 ? Math.max(...this.elements.map(el => el.id)) + 1 : 1;
      }
    }
  }
};
</script>

<style>
#report {
  width: 210mm; /* A4サイズ */
  height: 297mm; /* A4サイズ */
  position: relative;
  border: 1px solid #000;
  background-color: #fff; /* 背景色を白に設定 */
  padding: 20px;
  box-sizing: border-box;
  font-family: Arial, 'Hiragino Mincho Pro', 'ヒラギノ明朝', sans-serif;
}

body {
  background-color: #f0f0f0; /* ページ全体の背景色を設定 */
}

button {
  margin-top: 10px;
}

.report-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.title {
  font-size: 24px;
  font-weight: bold;
}

.info {
  display: flex;
  flex-direction: column;
}

.info-item {
  margin: 5px 0;
}

.report-section {
  margin-bottom: 20px;
  border: 1px solid #ccc;
  padding: 10px;
}

.section-header {
  font-weight: bold;
  background-color: #eee;
  padding: 5px;
}

.section-content {
  padding: 10px;
  min-height: 100px;
}

.vue-draggable-resizable {
  border: 1px dashed #000;
}

.no-border {
  border: none !important;
}

@media print {
  .remove-button {
    display: none;
  }
  .vue-draggable-resizable {
    border: none;
  }
}
</style>
