<script lang="ts">
//import TheWelcome from "@/components/TheWelcome.vue";
import ExpressionTree from "@/components/ExpressionTree.vue";
import { defineComponent, ref, computed, reactive } from "vue";
import { watch } from "vue";
import { NCard, NGrid, NGi, NCode } from "naive-ui";
import { NInput } from "naive-ui";
import { NButton } from "naive-ui";
import { NTable } from "naive-ui";
import _ from "lodash";

interface EvalStep {
  depth: number;
  form: string;
  id: number;
  value: string;
  us: number;
}

interface EvalSteps {
  value: EvalStep[];
}

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
    // Run steps
    const runSteps: EvalSteps = reactive({ value: [] });

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
      runSteps,
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
        if (!res.ok) {
          throw "Response is not ok";
        }
        // validate
        JSON.parse(codeJson.value);
        await runCode(codeJson.value);
      } catch (e) {
        runStatus.value = "warning";
        runResult.value = "N/A";
        runSteps.value = [];
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
        const json = await res.json();
        runResult.value = json.value;
        runSteps.value = json.steps;
      } else {
        runStatus.value = "warning";
        runResult.value = await res.text();
        runSteps.value = [];
      }
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
    NTable,
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
    </NGi>
    <NGi>
      <NCard title="可视化" :bordered="false">
        <ExpressionTree :node="visualize"></ExpressionTree>
      </NCard>
    </NGi>
  </NGrid>
  <NCard v-if="runSteps.value.length > 0" title="运算过程" :bordered="false">
    <NTable size="small" :single-line="false">
      <thead>
        <tr>
          <th>ID</th>
          <th>命令</th>
          <th>结果</th>
          <th>时间（微秒）</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(step, index) in runSteps.value" :key="index">
          <td>{{ step.id }}</td>
          <td>
            <code>{{ "\xa0".repeat(step.depth) + step.form }}</code>
          </td>
          <td>{{ step.value }}</td>
          <td>{{ step.us.toLocaleString() }}</td>
        </tr>
      </tbody>
    </NTable>
  </NCard>
  <NCard title="JSON" :bordered="false">
    <NCode :code="codeJson" language="JSON" />
  </NCard>
</template>

<style scoped>
.n-code {
  font-size: 10px;
}
</style>
