<template>
  <div id="app">
    <Menus refresh debug />
    <Dropzone 
      @drop="readFiles" 
      auto-read multiple :accepts="['.ttf', '.ttc', '.dfont', '.otf']" />
    <Panel>
      <brutalism-title title="" subtitle="Drop font files into this panel to get Postscript names" />
      <Wrapper>
        <progress
          id="file" 
          :value="progress" 
          max="100" 
          style="width: 100%"
          :style="{opacity: (progress) ? 100: 0}" />
        <!-- <TextArea
          copy-content
          :rows="10"
          v-model="readContent"
          auto-select
        /> -->
        <pre>{{fontInfo | json}} </pre>
        <div class="bottom">
          <Button block :label="(progress < 100) ? copyButton : 'Copy font info'" @click="copyJSON" :disabled="progress < 100" />
        </div>
      </Wrapper>
    </Panel>
  </div>
</template>

<script>
const path = require("path");
import FontName from 'fontname'
let fontInfo = {}

import { evalScript, copy} from "brutalism";
import { log } from 'util';
export default {
  data: () => ({
    filePath: "",
    realPath: "",
    readContent: "",
    fontInfo: {},
    progress: 0,
    copyButton: 'Drop files'
  }),
  components: {
    "battleaxe-logo": require("./components/battleaxeLogo.vue").default,
    "brutalism-title": require("./components/brutalismTitle.vue").default,
  },
  filters: {
        json (obj) {
            return JSON.stringify(obj, false, 2)
        },
  },
  methods: {
    copyJSON() {
        copy(JSON.stringify(this.fontInfo, false, 4))

        this.footerMessage = 'JSON copied'
    },
    readFiles(files) {
      // reset 
      this.progress = 0
      this.fontInfo = {}

      const reader = new FileReader()
      let fileData = {}
      let fileCount = files.length
      let fileNum = 0
      
      files.forEach(file => {
        const reader = new FileReader();
        reader.onload = e => {
          const buffer = e.target.result; // ArrayBuffer
          try {
            const fontMeta = FontName.parse(e.target.result)[0];
            console.log(fontMeta);
            if (fontMeta.fontFamily.charAt(0) != '.') {

              if (!this.fontInfo[fontMeta.fontFamily]) {
                this.fontInfo[fontMeta.fontFamily] = {}
              }
              this.fontInfo[fontMeta.fontFamily][fontMeta.fontSubfamily] = fontMeta.postScriptName
              console.log(this.fontInfo);
              this.readContent = JSON.stringify(this.fontInfo, false, 2)
            }
          } catch(e) {
            // FontName may throw an Error
          }

          // increment the progress bar
          fileNum++
          this.copyButton = `${parseInt(this.progress)}%`
          this.progress = (fileNum / fileCount) * 100
        };
        reader.readAsArrayBuffer(file)
      });
        this.readContent = JSON.stringify(fileData, false, 2)
    },
  },
};
</script>
<style scoped>
.bottom {
    position: fixed;
    bottom: 6px;
    width: calc(100% - 16px);
    /* -webkit-box-shadow: 0px 0px 16px 0px rgba(0,0,0,0.6); 
    box-shadow: 0px 0px 16px 0px rgba(0,0,0,0.6); */
}
pre {
  height: calc(100vh - 132px);
  overflow-y: scroll;
  overflow-x: hidden;
}
</style>