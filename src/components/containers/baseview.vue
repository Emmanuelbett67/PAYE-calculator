<template>
  <div class="ma-5 pa-4">
    <!--  -->
    <v-row no-gutters class="mb-4">
      <v-col cols="12" sm="12" class="text-h6">
        <Title :title="title" />
      </v-col>
    </v-row>

    <slot></slot>
    <!--  -->
  </div>
</template>

<script setup>
import Title from "@/components/Texts/Title.vue";

const props = defineProps({
  title: {
    type: String,
    default: "Untitled",
  },
});
</script>
