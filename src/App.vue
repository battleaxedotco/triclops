<template>
  <div id="app">
    <!-- <div 
      class="drop-zone"
      @drop="readFiles">

    </div> -->
    <Dropzone 
      @drop="readFiles" 
      auto-read multiple :accepts="['.ttf', '.ttc', '.dfont', '.otf']" />
    <div>
      <progress
          id="file" 
          :value="progress" 
          max="100" 
          style="width: 100%"
          :style="{opacity: (progress) ? 100: 0}" />
      <pre>{{fontInfo | json}} </pre>
    </div>
  </div>
</template>

<script>
// import HelloWorld from './components/HelloWorld.vue'
import Dropzone from './components/Dropzone.vue'
// const path = require("path");
import FontName from 'fontname'
// import vue2Dropzone from 'vue2-dropzone'
// import 'vue2-dropzone/dist/vue2Dropzone.min.css'

export default {
  name: 'App',
  data: () => ({
    // filePath: "",
    // realPath: "",
    // readContent: "",
    fontInfo: {},
    progress: 0,
    copyButton: 'Drop files'
  }),
  components: {
    Dropzone
    // vueDropzone: vue2Dropzone
    // HelloWorld
  },
  methods: {
    copyJSON() {
        this.$copyText(JSON.stringify(this.fontInfo, false, 4))
            .then(() => {
                setTimeout(() => {
                    this.keyCopied = null
                }, 2000);
            })

        // this.footerMessage = 'JSON copied'
    },
    readFiles(files) {
      // reset 
      this.progress = 0
      this.fontInfo = {}

      let fileData = {}
      let fileCount = files.length
      let fileNum = 0
      
      files.forEach(file => {
        const reader = new FileReader();
        reader.onload = e => {
          // const buffer = e.target.result; // ArrayBuffer
          try {
            const fontMeta = FontName.parse(e.target.result)[0];
            console.log(fontMeta);
            let familyName = fontMeta.fontFamily
            let styleName = fontMeta.fontSubfamily

            let concatName = `${familyName.replace(/ /g, '')}-${styleName.replace(/ /g, '')}`
            if (fontMeta.postScriptName != concatName) {

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
  filters: {
    json (obj) {
      return JSON.stringify(obj, false, 2)
    },
  },
  mounted() {
    // dont redirect if file dropped
    window.addEventListener(
      "dragover",
      function (e) {
        e = e || event;
        e.preventDefault();
      },
      false
    );
    window.addEventListener(
      "drop",
      function (e) {
        e = e || event;
        e.preventDefault();
      },
      false
    );
    window.addEventListener("dragenter", () => {
      if (this.fullscreen) {
        if (this.debug) console.log("dragover");
        this.leaveCount = 0;
        if (!this.isDragging) this.enter();
      } else {
        if (!this.isDraggingInWindow) this.isDraggingInWindow = true;
      }
    });
    window.addEventListener("dragleave", () => {
      if (this.fullscreen) {
        this.leaveCount++;
        if (this.leaveCount >= 2) this.exit();
      }
    });
    if (this.fullscreen) {
      window.addEventListener("drop", (e) => {
        e.preventDefault();
        this.drop(e);
      });
    }
  },
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.drop-zone {
  width: 300px;
  height: 300px;
  border: 1px solid salmon;
}
</style>
