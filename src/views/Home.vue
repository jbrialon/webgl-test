<template>
  <div class="shark">
    <fbx :url="'shark.fbx'"></fbx>
  </div>
</template>

<script>
// @ is an alias to /src
import fbx from '../components/fbx-viewer.vue'

export default {
  name: 'home',
  components: {
    fbx
  }
}
</script>
