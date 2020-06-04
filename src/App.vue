<template>
  <div id="app">
    <Menus refresh debug />
    <Panel>
      <brutalism-title title="" subtitle="BINARY CONVERTER" />
      <Wrapper>
        <File-Input
          label="File to convert to binary"
          :value="filePath"
          @update="updatePath"
          prefs-id="input-path"
          @prefs="kickstart"
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
    readContent: "",
  }),
  components: {
    "battleaxe-logo": require("./components/battleaxeLogo.vue").default,
    "brutalism-title": require("./components/brutalismTitle.vue").default,
  },
  methods: {
    async updatePath(val) {
      this.filePath = val.path;
      let code = `(function() {
        var fileContent, binaryString;
        var targPath = "${this.filePath.replace(/\\/gm, "/")}";
        //
        file = new File(targPath);
        file.encoding = "BINARY";
        file.open('r');
        fileContent = file.read();
        file.close();
        //
        binaryString = fileContent.toSource();
        binaryString = binaryString.replace(/^\\(new String\\(/, '');
        binaryString = binaryString.replace(/\\)\\)$/, ';');
        return binaryString;
      }())
      `;
      let result = await evalScript(code);
      this.readContent = result.replace(/\;$/, "");
    },
  },
};
</script>

<style>
input,
textarea,
select,
button {
  font-family: monospace;
}
</style>
