<template>
  <div class="turtle">
    <fbx :url="'turtle.fbx'"></fbx>
  </div>
</template>

<script>
// @ is an alias to /src
import fbx from '@/components/fbx-viewer.vue'

export default {
  name: 'home',
  components: {
    fbx
  }
}
</script>
