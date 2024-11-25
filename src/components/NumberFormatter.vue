<template>
    <v-text-field
      ref="textField"
      v-model="formattedNumber"
      label="label"
      variant="solo"
      @input="handleInput"
      :rules="[numberOnly, ...rules]"
      type="text"
    />
  </template>
  
  <script setup>
  import { ref, watch, nextTick } from "vue";
  
  const props = defineProps({
    modelValue: {
      type: [String, Number],
      default: ""
    },
    rules: {
      type: Array,
      default: () => []
    }
  });
  
  const emit = defineEmits(["update:modelValue"]);
  
  defineExpose({
    validate
  });
  
  const textField = ref(null);
  
  const rawValue = ref(props.modelValue);
  const formattedNumber = ref("");
  
  const formatNumber = (value) => {
    if (value == null) return "";
    const parts = String(value).split(".");
    parts[0] = parts[0].replace(/\B(?=(\d{3})+(?!\d))/g, ",");
    return parts.join(".");
  };
  
  const parseNumber = (value) => {
    if (value == null) return "";
    return value.replace(/,/g, "");
  };
  
  function numberOnly(v) {
    const num = parseNumber(v);
  
    return isNaN(Number(num)) ? "Invalid: input must be a valid number" : true;
  }
  
  const handleInput = (event) => {
    const input = event.target.value;
    const cursorPosition = event.target.selectionStart;
  
    const unformattedInput = parseNumber(input);
    const unformattedCursorPosition =
      cursorPosition - (input.slice(0, cursorPosition).match(/,/g) || []).length;
  
    const formattedInput = formatNumber(unformattedInput);
  
    const newFormattedCursorPosition =
      unformattedCursorPosition +
      (formattedInput.slice(0, cursorPosition).match(/,/g) || []).length;
  
    emit("update:modelValue", unformattedInput);
  
    formattedNumber.value = formattedInput;
  
    nextTick(() => {
      event.target.setSelectionRange(
        newFormattedCursorPosition,
        newFormattedCursorPosition
      );
    });
  };
  
  watch(
    () => props.modelValue,
    (newValue) => {
      rawValue.value = newValue;
      formattedNumber.value = formatNumber(newValue);
    },
    { immediate: true }
  );
  
  function validate() {
    textField.value.validate();
  }
  </script>
  