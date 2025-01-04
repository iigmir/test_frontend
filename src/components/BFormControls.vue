<template>
  <div>
    <label :for="id_bind" class="form-label">{{ props.label }}</label>
    <input :type="type_to_bind" class="form-control" :id="id_bind" placeholder="" />
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from "vue";
import { get_uuid } from "../assets/utils";

const props = defineProps({
  label: String,
  type: String
});

const type_to_bind = computed( () => props.type ?? "text" );

// Element ID module
const uuid_id = ref("");
const generate_uuid_id = (input = "") => {
  // If has element, generate uuid again
  if( document.querySelector(`#label-${input}`) != null ) {
    return generate_uuid_id(get_uuid());
  }
  return input;
};
uuid_id.value = generate_uuid_id( get_uuid() );

const id_bind = computed( () => `label-${uuid_id.value}` );
</script>

<style lang="scss" scoped>

</style>