<script>
import { reactive } from 'vue';
import beholderData from '@/assets/beholder.json'; // Adjust the path as necessary
import Beholder from '@/components/Beholder.vue'; // Import the Beholder component

export default {
  components: {
    Beholder
  },
  data() {
    return {
      beholder: reactive(beholderData)
    };
  }
}
</script>

<template>
    <Beholder :beholder="beholder"/>
</template>

<style scoped>
/* Add your styles here */
</style>
