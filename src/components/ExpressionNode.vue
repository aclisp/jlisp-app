<script setup lang="ts">
const props = defineProps({
  node: {
    type: Object,
    required: true,
  },
});
</script>

<template>
  <li v-if="props.node.type == 'list' && props.node.value.length > 0">
    <details open>
      <template v-if="props.node.value[0].type == 'list'">
        <summary>&nbsp;</summary>
        <ul>
          <ExpressionNode
            v-for="(node, index) in props.node.value"
            :node="node"
            :key="index"
          />
        </ul>
      </template>
      <template v-else>
        <summary>
          {{ props.node.value[0].value }}
        </summary>
        <ul>
          <ExpressionNode
            v-for="(node, index) in props.node.value.slice(1)"
            :node="node"
            :key="index"
          />
        </ul>
      </template>
    </details>
  </li>
  <li v-if="props.node.type == 'list' && props.node.value.length == 0">
    <details open>
      <summary>&nbsp;</summary>
    </details>
  </li>
  <li v-if="props.node.type == 'symbol'">
    {{ props.node.value }}
  </li>
  <li v-if="props.node.type == 'object'">
    {{ props.node.value ?? "null" }}
  </li>
  <li v-if="props.node.type == 'array'">
    {{ props.node.value }}
  </li>
</template>
