<template>
    <div
      :class="`font-weight-medium text-subtitle-1 ${
        !sub && 'text-uppercase'
      } ${styleClasses}`"
    >
      <!-- Flex container to center the title -->
      <div class="d-flex justify-center align-center">
        <div class="title-wrap">{{ title }}</div>
        <slot></slot>
      </div>
      <v-divider v-if="showDivider" class="center-divider"></v-divider>
    </div>
  </template>
  
  <script setup>
  const props = defineProps({
    title: String,
    sub: Boolean,
    styleClasses: String,
    showDivider: {
      default: true,
      type: Boolean
    }
  });
  </script>
  
  <style scoped>
  .title-wrap {
    white-space: normal;
    overflow: hidden;
    text-overflow: ellipsis;
    text-align: center; 
  }
  
  .d-flex {
    display: flex;
  }
  
  .justify-center {
    justify-content: center; 
  }
  
  .align-center {
    align-items: center;
  }
  
  .center-divider {
    width: 50%; 
    margin: 10px auto; 
  }
  </style>
  