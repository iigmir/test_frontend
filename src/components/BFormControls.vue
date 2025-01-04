<template>
  <div>
    <label
      :for="element_id"
      class="form-label"
    >
      {{ props.label }}
    </label>
    <input
      :type="type_to_bind"
      class="form-control"
      :id="element_id"
      placeholder="輸入..."
      @input="emit_model"
    />
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from "vue";
import { get_uuid } from "../assets/utils";

const props = defineProps({
  label: String,
  type: String,
  /**
   * We don't really know what will sent from parents, so...
   */
  modelValue: [Boolean, Array, Number, String],
});

/**
 * @see [Component v-model](https://vuejs.org/guide/components/v-model)
 */
const emits = defineEmits([
  "update:modelValue"
]);

/**
 * @see These articles:
 * * [Property 'value' does not exist on type EventTarget in TypeScript](https://stackoverflow.com/a/44321394)
 * * [Vue 3 Typescript warning on Vue $emit and $event "Object is possibly 'null'"](https://stackoverflow.com/a/67441785)
 * * [Error occurring when using v-model in Vue 3](https://stackoverflow.com/a/71136784)
 */
 const emit_model = (e: Event) => {
  const source_value = (e.target as HTMLInputElement).value;
  const get_emitted_value = (source_value = "", input_type = "text") => {
    switch (input_type) {
      // Special cases
      case "number": return parseInt(source_value, 10);
      case "checkbox": return Boolean(source_value);
      // In general cases we just return the value as-is
      default: return source_value;
    }
  };
  // Just let parent components decide how to deal with it
  const emitted_value: any = get_emitted_value(source_value, type_to_bind.value);
  emits("update:modelValue", emitted_value);
};

/**
 * Out input type
 * @see [<input> types](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#input_types)
 */
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

const element_id = computed( () => `label-${uuid_id.value}` );
</script>

<style lang="scss" scoped>

</style>