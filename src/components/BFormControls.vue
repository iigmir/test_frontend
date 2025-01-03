<template>
  <div>
    <label :for="id_bind" class="form-label">{{ props.label }}</label>
    <input :type="type_to_bind" class="form-control" :id="id_bind" placeholder="" />
  </div>
</template>

<script setup lang="ts">
import { computed } from "vue";

const props = defineProps({
  label: String,
  type: String
});

const type_to_bind = computed( () => props.type ?? "text" );
const id_bind = computed( () => `label-${props.label}-${type_to_bind.value}` );
</script>

<style lang="scss" scoped>

</style>