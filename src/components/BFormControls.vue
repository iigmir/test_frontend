<template>
  <div>
    <label
      class="form-label"
      :for="element_id"
    >
      {{ props.label }}
    </label>
    <input
      class="form-control"
      placeholder="輸入..."
      :id="element_id"
      :type="input_type"
      :value="props.modelValue"
      @input="emit_model"
    />
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from "vue";
import { get_uuid } from "../assets/utils";

// Props & emits module
/**
 * Why we use such a scrawl: 
 * [Type-only props/emit declarations](https://vuejs.org/api/sfc-script-setup.html#type-only-props-emit-declarations)
 */
const props = defineProps({
  label: String,
  type: String,
  /**
   * We don't really know what will sent from parents, so...
   */
  modelValue: [Number, String],
});
/**
 * @see: [Component v-model](https://vuejs.org/guide/components/v-model)
 */
const emits = defineEmits([
  "update:modelValue"
]);
/**
 * Default to "text" if no type is provided.
 * @see https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#input_types
 */
const input_type = computed( () => props.type ?? "text" );

/**
 * Easy: When the value of the input element changes, the component emits.
 * @see These articles:
 * * [Property 'value' does not exist on type EventTarget in TypeScript](https://stackoverflow.com/a/44321394)
 * * [Vue 3 Typescript warning on Vue $emit and $event "Object is possibly 'null'"](https://stackoverflow.com/a/67441785)
 * * [Error occurring when using v-model in Vue 3](https://stackoverflow.com/a/71136784)
 */
const emit_model = (e: Event) => {
  const source_value = (e.target as HTMLInputElement).value;
  /**
   * Convert the input value based on the input type.
   * @param input - The raw value from the input element.
   * @param input_type - The input type (e.g., "number", "checkbox").
   * @returns The parsed value.
  */
  const get_emitted_value = (input = "", input_type = "text") => {
    switch (input_type) {
      // Special cases
      case "number": return parseInt(input, 10);
      case "checkbox": return Boolean(input);
      // In general cases we just return the value as-is
      default: return input;
    }
  };
  /**
   * Just let parent components decide how to deal with them
   */
  const emitted_value: any = get_emitted_value(source_value, input_type.value);
  emits("update:modelValue", emitted_value);
};

// Element ID module
const uuid_id = ref("");
/**
 * Generate a unique element ID. Ensures no duplicate IDs in the page.
 * @param input - An initial value for the UUID generation.
 */
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