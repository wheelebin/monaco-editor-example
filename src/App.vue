<template>
  <div id="app">
    <nav id="nav">
      <div class="button" @click="togglePanel('editor')">
        <span>Editor</span>
      </div>
      <div class="button" @click="togglePanel('tutorial')">
        <span>Tutorial</span>
      </div>
      <div class="navResizer" @mousedown="initResize('nav')"></div>
    </nav>

    <div id="wrap">
      <div class="editor" v-show="panels['editor'].visible">
        <div class="closePanel" @click="togglePanel('editor')">x</div>
        <div id="container"></div>
        <div
          class="panelResizer"
          @mousedown="initResize('panels')"
          v-if="visiblePanels == 2"
        ></div>
      </div>
      <div class="tutorial" v-show="panels['tutorial'].visible">
        <div class="closePanel" @click="togglePanel('tutorial')">x</div>
        <div id="container2"></div>
      </div>
    </div>
  </div>
</template>

<script>
import * as monaco from "monaco-editor";

export default {
  name: "App",
  data() {
    return {
      draggableSelection: "",
      panels: {
        editor: {
          visible: false,
          editor: null,
          DOMid: "container",
          defaultContent: "function hello() {\n\talert('Hello world!');\n}",
        },
        tutorial: {
          visible: false,
          editor: null,
          DOMid: "container2",
          defaultContent:
            "/*\nHere we have an array of objects which we will loop through using different iteration techniques.*/\n\nconst items = [\n\t{ name: 'a', content: { /* ... */ }},\n\t{ name: 'b', content: { /* ... */ }},\n\t{ name: 'c', content: { /* ... */ }}\n]\n\n/*This is an example of using 'reduce'*/\nconst a = items.reduce((result, item) => {\n\tif (item.name === 'b') { result = item }\n\treturn result\n}, null)\n\n/*This is an example of using 'find'*/\nconst b = items.find((item) => item.name === 'b')\n\n/*This is an example of using 'filter'*/\nconst c = items.filter((item) => item.name === 'b').shift()\n\n/*This is an example of using 'map'*/\nconst d = items.map((item) => performSomething(item))",
        },
      },
    };
  },
  computed: {
    visiblePanels() {
      return Object.values(this.panels).filter((obj) => obj.visible == true)
        .length;
    },
  },
  mounted() {
    this.initMonaco();

    window.addEventListener("resize", () => this.resizeAllPanels());

    /* Inserts dragged selection into editor on mouse up & clears dragged selection after. */
    this.panels.editor.editor.onMouseUp((e) => {
      if (this.draggableSelection == "") return false;

      this.panels.editor.editor.executeEdits("", [
        {
          range: new monaco.Range(
            e.target.position.lineNumber,
            e.target.position.column,
            e.target.position.lineNumber,
            e.target.position.column
          ),
          text: this.draggableSelection,
        },
      ]);

      this.draggableSelection = "";
    });

    /* Saves selected text from tutorial panel into draggableSelection */
    this.panels.tutorial.editor.onMouseDown(() => {
      this.draggableSelection = this.panels.tutorial.editor
        .getModel()
        .getValueInRange(this.panels.tutorial.editor.getSelection());
    });
  },
  methods: {
    initMonaco() {
      Object.keys(this.panels).forEach((key) => {
        const obj = this.panels[key];
        obj.editor = monaco.editor.create(document.getElementById(obj.DOMid), {
          value: obj.defaultContent,
          language: "javascript",
          theme: "vs-dark",
          minimap: {
            enabled: false,
          },
        });
      });
    },
    togglePanel(panel) {
      this.panels[panel].visible = !this.panels[panel].visible;
      this.resizeAllPanels();
    },
    initResize(resizeType) {
      const mousemove = (e) => this.resize(e, resizeType);
      const mouseup = () => {
        document.removeEventListener("mousemove", mousemove);
        document.removeEventListener("mouseup", mouseup);
      };
      document.addEventListener("mousemove", mousemove, false);
      document.addEventListener("mouseup", mouseup, false);
    },
    resizeAllPanels() {
      /* Resizes visible panels to share even width */
      const wrapper = document.getElementById("wrap");
      const newPanelSize = wrapper.offsetWidth / this.visiblePanels;

      Object.keys(this.panels).forEach((key) => {
        if (this.panels[key].visible) {
          this.panels[key].editor.layout({
            width: newPanelSize,
            height: wrapper.offsetHeight,
          });
        }
      });
    },
    resize(e, resizeType) {
      const wrapper = document.getElementById("wrap");
      const nav = document.getElementById("nav");

      var editorW, tutorialW;

      switch (resizeType) {
        case "nav":
          editorW = document.getElementById("container").offsetWidth;
          tutorialW = document.getElementById("container2").offsetWidth;

          nav.style.height = `${e.clientY}px`;
          wrapper.style.height = `calc(100% - ${nav.offsetHeight}px)`;
          break;

        case "panels":
          editorW = e.clientX;
          tutorialW = wrapper.offsetWidth - e.clientX;
          break;
      }

      this.panels.editor.editor.layout({
        width: editorW,
        height: wrapper.offsetHeight,
      });
      this.panels.tutorial.editor.layout({
        width: tutorialW,
        height: wrapper.offsetHeight,
      });
    },
  },
};
</script>

<style lang="scss">
html,
body,
#app {
  height: 100%;
  width: 100%;
  margin: 0;
  background-color: #1e1e1e;
}

#nav {
  display: flex;
  z-index: 20;
  background-color: #1e1e1e;
  width: 100%;
  position: relative;
  font-family: mono;

  .button {
    height: 50px;
    display: flex;
    justify-content: center;
    width: 100px;
    align-items: center;
    margin-top: auto;
    color: white;
    cursor: pointer;
    margin-right: 15px;
    opacity: 0.7;
    &:hover {
      opacity: 1;
    }
  }
}

#wrap {
  display: flex;
  box-sizing: border-box;
  width: 100%;
  height: calc(100% - 50px);
  padding-top: 10px;

  .tutorial,
  .editor {
    height: 100%;
    position: relative;
    flex: 1 1 auto;

    .closePanel {
      position: absolute;
      height: 30px;
      width: 30px;
      color: white;
      cursor: pointer;
      top: 0px;
      right: 15px;
      z-index: 10;
    }
  }
}

.panelResizer {
  position: absolute;
  right: 0px;
  top: 0px;
  height: 100%;
  width: 4px;
  background-color: #0d0d0d;
  cursor: ew-resize;
}
.navResizer {
  position: absolute;
  right: 0px;
  bottom: 0px;
  height: 4px;
  width: 100%;
  z-index: 20;
  background-color: #0d0d0d;
  cursor: n-resize;
}
</style>
