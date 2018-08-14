<template>
  <div class="fbx-viewer">
  </div>
</template>

<script>
import * as THREE from 'three'
import FBXLoader from 'three-fbxloader-offical'
import OrbitControls from 'three-orbitcontrols'

let mixers = []
let camera, scene, renderer, light, controls, action

let clock = new THREE.Clock();

export default {
  name: 'fbx-viewer',
  props: {
    play: {
      type: Boolean,
      default: false
    }
  },
  methods: {
    init () {
      camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 1, 2000)
      camera.position.set(600, 200, 300)

      controls = new OrbitControls(camera)
      controls.target.set(0, 100, 0)
      controls.update()

      scene = new THREE.Scene()
      scene.background = new THREE.Color(0x045797)
      scene.fog = new THREE.Fog(0x045797, 200, 1000)

      light = new THREE.HemisphereLight(0xffffff, 0x444444)
      light.position.set(0, 200, 0)
      scene.add(light)

      light = new THREE.DirectionalLight(0xffffff)
      light.position.set(0, 200, 100)
      light.castShadow = true
      light.shadow.camera.top = 180
      light.shadow.camera.bottom = -100
      light.shadow.camera.left = -120
      light.shadow.camera.right = 120
      // scene.add(light)

      // scene.add( new THREE.CameraHelper( light.shadow.camera ) );

      // ground

      var grid = new THREE.GridHelper( 2000, 20, 0x000000, 0x000000 );
      grid.material.opacity = 0.2;
      grid.material.transparent = true;
      scene.add( grid );

      // model
      let loader = new FBXLoader()
      loader.load('riggedshark.fbx', function (object) {
        object.mixer = new THREE.AnimationMixer(object)
        mixers.push(object.mixer)

        action = object.mixer.clipAction(object.animations[ 0 ])
        action.play()
        object.traverse(function (child) {
          if (child.isMesh) {
            child.castShadow = true
            child.receiveShadow = true
          }
        })
        object.position.y = 100
        scene.add(object)
      })

      renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true })
      renderer.setPixelRatio(window.devicePixelRatio)
      renderer.setSize(window.innerWidth, window.innerHeight)
      renderer.shadowMap.enabled = true
      this.$el.appendChild(renderer.domElement)

      window.addEventListener( 'resize', this.onWindowResize, false )
    },
    animate () {
      requestAnimationFrame(this.animate)
      if (mixers.length > 0) {
        for (var i = 0; i < mixers.length; i++) {
          mixers[ i ].update(clock.getDelta())
        }
      }
      renderer.render(scene, camera)
    },
    onWindowResize () {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix()
      renderer.setSize( window.innerWidth, window.innerHeight )
    }
  },
  watch: {
    play () {
      if (this.play) {
        action.play()
      } else {
        action.stop()
      }
    }
  },
  mounted () {
    this.init()
    this.animate()
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>