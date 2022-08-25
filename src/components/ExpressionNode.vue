<script setup lang="ts">
import { NTag } from "naive-ui";
const props = defineProps({
  node: {
    type: Object,
    required: true,
  },
});
const idColor = {
  color: "#FFF",
  textColor: "#F8F8F8",
  borderColor: "#FFFF",
};
</script>

<template>
  <li v-if="props.node.type == 'list' && props.node.value.length > 0">
    <details open>
      <template v-if="props.node.value[0].type == 'list'">
        <summary>
          &nbsp; <NTag size="small" :color="idColor">#{{ props.node.id }}</NTag>
        </summary>
        <ul>
          <ExpressionNode
            v-for="node in props.node.value"
            :node="node"
            :key="node.id"
          />
        </ul>
      </template>
      <template v-else>
        <summary>
          {{ props.node.value[0].value ?? "null" }}
          <NTag size="small" :color="idColor">#{{ props.node.id }}</NTag>
          <!-- <sub>[{{ props.node.value[0].id }}]</sub> -->
        </summary>
        <ul>
          <ExpressionNode
            v-for="node in props.node.value.slice(1)"
            :node="node"
            :key="node.id"
          />
        </ul>
      </template>
    </details>
  </li>
  <li v-if="props.node.type == 'list' && props.node.value.length == 0">
    <details open>
      <summary>
        &nbsp; <NTag size="small" :color="idColor">#{{ props.node.id }}</NTag>
      </summary>
    </details>
  </li>
  <li v-if="props.node.type == 'symbol'">
    {{ props.node.value }}
    <!-- <sub>[{{ props.node.id }}]</sub> -->
  </li>
  <li v-if="props.node.type == 'object'">
    {{ props.node.value ?? "null" }}
    <!-- <sub>[{{ props.node.id }}]</sub> -->
  </li>
  <li v-if="props.node.type == 'array'">
    {{ props.node.value }}
    <!-- <sub>[{{ props.node.id }}]</sub> -->
  </li>
</template>
