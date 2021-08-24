<template>
  <a-tooltip :title="type.name">
    <img v-if="type" :src="type.image" :alt="type.name" />
  </a-tooltip>
</template>
<script>
import { defineComponent } from "vue";
import TypeData from "../data/type.js";

export default defineComponent({
  props: {
    data: {
      type: String,
      default: "鬼族"
    }
  },
  setup(props) {
    let type = TypeData.find(data => data.name === props.data)

    if (type) {
      type = {
        ...type,
        image: `/assets/images/${type.id}.png`,
      }
    }
    return {
      type
    }
  },
})
</script>

<style scoped>
img {
  min-width: 20px;
  width: 20px;
}
@media (min-width: 768px) {
  img {
    width: 32px;
  }
}
</style>
