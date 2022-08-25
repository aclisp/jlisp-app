<script lang="ts">
//import TheWelcome from "@/components/TheWelcome.vue";
import ExpressionTree from "@/components/ExpressionTree.vue";
import { defineComponent, ref, computed } from "vue";
import { watch } from "vue";
import { NCard, NGrid, NGi, NCode } from "naive-ui";
import { NInput } from "naive-ui";
import { NButton } from "naive-ui";
import _ from "lodash";

export default defineComponent({
  setup() {
    // User input code
    const code = ref("");
    // JSON representation of user input code
    const codeJson = ref("");
    // Run result of the JSON representation
    const runResult = ref("");
    // Run status
    const runStatus = ref("success");

    // Convert user input code to JSON representation and run with this JSON representation.
    const debouncedRun = _.debounce(getCodeJsonAndRun, 500);
    watch(code, debouncedRun);

    // Visualization of the the JSON representation
    const visualize = computed(() => {
      try {
        return JSON.parse(codeJson.value);
      } catch (e) {
        return {};
      }
    });

    return {
      code,
      codeJson,
      visualize,
      runResult,
      runStatus,
      formatCode,
      oneLineCode,
    };

    async function getCodeJsonAndRun(newCode: string) {
      const res = await fetch("http://localhost:8080/formular/json", {
        method: "POST",
        body: newCode,
      });
      codeJson.value = await res.text();

      // run it
      try {
        // validate
        JSON.parse(codeJson.value);
        await runCode(codeJson.value);
      } catch (e) {
        runResult.value = "N/A";
        runStatus.value = "warning";
      }
    }

    async function formatCode() {
      const res = await fetch("http://localhost:8080/formular/fmt", {
        method: "POST",
        body: code.value,
      });
      if (res.ok) {
        code.value = await res.text();
      }
    }

    async function oneLineCode() {
      const res = await fetch("http://localhost:8080/formular/oneline", {
        method: "POST",
        body: code.value,
      });
      if (res.ok) {
        code.value = await res.text();
      }
    }

    async function runCode(json: string) {
      const res = await fetch("http://localhost:8080/formular/eval", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: json,
      });
      if (res.ok) {
        runStatus.value = "success";
      } else {
        runStatus.value = "warning";
      }
      runResult.value = await res.text();
    }
  },
  components: {
    NInput,
    NButton,
    ExpressionTree,
    NCard,
    NGrid,
    NGi,
    NCode,
  },
});
</script>

<template>
  <!-- <main>
    <TheWelcome />
  </main> -->
  <NGrid :cols="2">
    <NGi>
      <NCard title="输入公式">
        <NInput
          v-model:value="code"
          type="textarea"
          :status="runStatus"
          :autosize="{
            minRows: 3,
          }"
        />
        <NButton @click="formatCode" size="small">格式化</NButton>
        <NButton @click="oneLineCode" size="small">单行化</NButton>
      </NCard>
      <NCard title="运算结果" :bordered="false">
        <NCode :code="runResult" />
      </NCard>
      <NCard title="JSON" :bordered="false">
        <NCode :code="codeJson" language="JSON" />
      </NCard>
    </NGi>
    <NGi>
      <NCard title="可视化" :bordered="false">
        <ExpressionTree :node="visualize"></ExpressionTree>
      </NCard>
    </NGi>
  </NGrid>
</template>

<style scoped>
.n-code {
  font-size: 10px;
}
</style>
