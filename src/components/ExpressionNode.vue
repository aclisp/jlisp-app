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
        <summary>
          &nbsp; <sub>[{{ props.node.id }}]</sub>
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
          &nbsp; <sub>[{{ props.node.id }}]</sub>
          {{ props.node.value[0].value ?? "null" }}
          <sub>[{{ props.node.value[0].id }}]</sub>
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
        &nbsp; <sub>[{{ props.node.id }}]</sub>
      </summary>
    </details>
  </li>
  <li v-if="props.node.type == 'symbol'">
    {{ props.node.value }} <sub>[{{ props.node.id }}]</sub>
  </li>
  <li v-if="props.node.type == 'object'">
    {{ props.node.value ?? "null" }} <sub>[{{ props.node.id }}]</sub>
  </li>
  <li v-if="props.node.type == 'array'">
    {{ props.node.value }} <sub>[{{ props.node.id }}]</sub>
  </li>
</template>
