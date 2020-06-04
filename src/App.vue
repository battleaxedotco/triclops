<template>
  <div id="app">
    <Menus refresh debug />
    <Dropzone @drop="updateFilePath" paths-only single :accepts="['.ffx']" />
    <Panel>
      <brutalism-title title="" subtitle="BINARY CONVERTER" />
      <Wrapper>
        <File-Input
          label="File to convert to binary"
          :value="filePath"
          @update="updatePath"
          prefs-id="input-path"
        />
        <Input
          copy-content
          :disabled="!readContent.length"
          v-model="readContent"
          truncate
        />
        <TextArea
          copy-content
          disabled
          :rows="10"
          v-model="readContent"
          truncate
        />
      </Wrapper>
    </Panel>
  </div>
</template>

<script>
const path = require("path");
import { evalScript } from "brutalism";
export default {
  data: () => ({
    filePath: "",
    realPath: "",
    readContent: "",
  }),
  components: {
    "battleaxe-logo": require("./components/battleaxeLogo.vue").default,
    "brutalism-title": require("./components/brutalismTitle.vue").default,
  },
  methods: {
    updateFilePath(data) {
      this.filePath = data[0];
    },
    async updatePath(val) {
      this.realPath = typeof val === "string" ? val : val.path;
      let temp =
        this.realPath && this.realPath.length
          ? this.realPath.replace(/\\/gm, "/")
          : "";
      if (!temp.length) {
        console.error("NO LENGTH?");
        return false;
      }
      let code = `(function() {
        var fileContent, binaryString;
        var targPath = "${temp.replace(/\\/gm, "/")}";
        //
        file = new File(targPath);
        file.encoding = "BINARY";
        file.open('r');
        fileContent = file.read();
        file.close();
        //
        binaryString = fileContent.toSource();
        if (binaryString && binaryString.length) {
          binaryString = binaryString.replace(/^\\(new String\\(/, '');
          binaryString = binaryString.replace(/\\)\\)$/, ';');
          return binaryString;
        } else {
          return false;
        }
      }())
      `;
      let result = await evalScript(code);
      this.readContent = result.replace(/\;$/, "");
    },
  },
};
</script>
