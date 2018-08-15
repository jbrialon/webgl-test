<template>
  <div class="turtle">
    <fbx :url="'diver.fbx'"></fbx>
  </div>
</template>

<script>
// @ is an alias to /src
import fbx from '@/components/fbx-viewer.vue'

export default {
  name: 'Diver',
  components: {
    fbx
  }
}
</script>
