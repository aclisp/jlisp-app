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
    const code = ref("");
    const json = ref("");
    const run = ref("");
    const status = ref("success");

    const debouncedGetCodeJson = _.debounce(getCodeJson, 500);
    watch(code, debouncedGetCodeJson);

    const debouncedRunCode = _.debounce(runCode, 500);
    watch(code, debouncedRunCode);

    const node = computed(() => {
      try {
        return JSON.parse(json.value);
      } catch (e) {
        return {};
      }
    });

    return {
      code,
      json,
      node,
      run,
      formatCode,
      oneLineCode,
      runCode,
      status,
    };

    async function getCodeJson(newCode: string) {
      const res = await fetch("http://localhost:8080/formular/json", {
        method: "POST",
        body: newCode,
      });
      json.value = await res.text();
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

    async function runCode() {
      const res = await fetch("http://localhost:8080/formular/eval", {
        method: "POST",
        body: code.value,
      });
      if (res.ok) {
        status.value = "success";
      } else {
        status.value = "warning";
      }
      run.value = await res.text();
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
          :status="status"
          :autosize="{
            minRows: 3,
          }"
        />
        <NButton @click="formatCode" size="small">格式化</NButton>
        <NButton @click="oneLineCode" size="small">单行化</NButton>
      </NCard>
      <NCard title="运算结果" :bordered="false">
        <NCode :code="run" />
      </NCard>
      <NCard title="JSON" :bordered="false">
        <NCode :code="json" language="JSON" />
      </NCard>
    </NGi>
    <NGi>
      <NCard title="可视化" :bordered="false">
        <ExpressionTree :node="node"></ExpressionTree>
      </NCard>
    </NGi>
  </NGrid>
</template>

<style scoped>
.n-code {
  font-size: 10px;
}
</style>
