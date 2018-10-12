<template>
  <div class="fbx-viewer"></div>
</template>

<script>
import * as THREE from "three"
import FBXLoader from "three-fbxloader-offical"
import OrbitControls from "three-orbitcontrols"
import DragControls from "three-dragcontrols"
import TransformControls from "three-transformcontrols"
import * as dat from "dat.gui"

import Stats from "stats.js"

import Plankton from "@/components/Plankton.js"

let mixers = []
let camera, scene, renderer, light, controls, action, stats, splineCamera, cameraHelper, model
var curve
var followCamera = false
let clock = new THREE.Clock()

var camPosIndex = 0
export default {
  name: "fbx-viewer",
  props: {
    url: {
      type: String,
      default: "shark.fbx"
    }
  },
  methods: {
    init() {
      camera = new THREE.PerspectiveCamera(
        45,
        window.innerWidth / window.innerHeight,
        1,
        2000
      )
      camera.position.set(600, 200, 300);
      
      scene = new THREE.Scene();
      scene.background = new THREE.Color(0x045797);
      scene.fog = new THREE.Fog(0x045797, 200, 1000)

      light = new THREE.HemisphereLight(0xffffff, 0x444444);
      light.position.set(0, 200, 0);
      scene.add(light);

      splineCamera = new THREE.PerspectiveCamera(
        45,
        window.innerWidth / window.innerHeight,
        1,
        2000
      )
      cameraHelper = new THREE.CameraHelper( splineCamera )
      scene.add( cameraHelper )
      scene.add( splineCamera )

      // ground
      var mesh = new THREE.Mesh(
        new THREE.PlaneBufferGeometry(2000, 2000),
        new THREE.MeshPhongMaterial({ color: 0x045797, depthWrite: false })
      );
      mesh.rotation.x = -Math.PI / 2;
      mesh.receiveShadow = true;
      scene.add(mesh);

      // var grid = new THREE.GridHelper(2000, 20, 0x000000, 0x000000);
      // grid.material.opacity = 0.2;
      // grid.material.transparent = true;
      // scene.add(grid);

      // model
      let loader = new FBXLoader();
      loader.load(this.url, function(object) {
        object.mixer = new THREE.AnimationMixer(object);
        mixers.push(object.mixer);

        action = object.mixer.clipAction(object.animations[0]);
        action.play();

        object.traverse(function(child) {
          if (child.isMesh) {
            child.castShadow = true;
            child.receiveShadow = true;
          }
        });
        object.scale.multiplyScalar(0.4)
        // object.position.y = 100
        model = object
        scene.add(model)
      });

      // set up plankton
      let plankton = new Plankton();
      plankton.position.y = 600;
      scene.add(plankton);

      // renderer
      renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
      renderer.setPixelRatio(window.devicePixelRatio);
      renderer.setSize(window.innerWidth, window.innerHeight);
      renderer.shadowMap.enabled = true;
      this.$el.appendChild(renderer.domElement);

      String.prototype.format = function() {
        var str = this;

        for (var i = 0; i < arguments.length; i++) {
          str = str.replace("{" + i + "}", arguments[i]);
        }
        return str;
      };

      var splineHelperObjects = [];
      var splinePointsLength = 4;
      var positions = [];
      var point = new THREE.Vector3();

      var geometry = new THREE.BoxBufferGeometry(20, 20, 20);
      var transformControl;

      var ARC_SEGMENTS = 200;

      var splines = {};

      var params = {
        uniform: true,
        tension: 0.5,
        centripetal: true,
        chordal: true,
        animationView: false,
        addPoint: addPoint,
        removePoint: removePoint,
        exportSpline: exportSpline
      };

      var gui = new dat.GUI();

      gui.add(params, 'addPoint')
      gui.add(params, 'removePoint')
      gui.add(params, 'exportSpline')
      gui.add(params, 'animationView').onChange(function () {
        animateCamera()
      })
      gui.open()

      // Controls
      controls = new OrbitControls(camera, renderer.domElement)
      controls.damping = 0.2
      controls.addEventListener('change', render)

      controls.addEventListener('start', function() {
        cancelHideTransorm()
      })

      controls.addEventListener('end', function() {
        delayHideTransform()
      })

      transformControl = new TransformControls(camera, renderer.domElement);
      transformControl.addEventListener('change', render)
      transformControl.addEventListener('dragging-changed', function() {
        controls.enabled = !event.value
      })
      scene.add(transformControl)

      // Hiding transform situation is a little in a mess :()
      transformControl.addEventListener('change', function() {
        cancelHideTransorm();
      })

      transformControl.addEventListener('mouseDown', function() {
        cancelHideTransorm();
      })

      transformControl.addEventListener('mouseUp', function() {
        delayHideTransform();
      })

      transformControl.addEventListener('objectChange', function() {
        updateSplineOutline();
      })

      var dragcontrols = new DragControls(
        splineHelperObjects,
        camera,
        renderer.domElement
      ); //
      dragcontrols.enabled = false;
      dragcontrols.addEventListener('hoveron', function(event) {
        transformControl.attach(event.object);
        cancelHideTransorm();
      });

      dragcontrols.addEventListener('hoveroff', function() {
        delayHideTransform()
      });

      var hiding

      function delayHideTransform() {
        cancelHideTransorm()
        hideTransform()
      }

      function hideTransform() {
        hiding = setTimeout(function() {
          transformControl.detach(transformControl.object)
        }, 2500)
      }

      function cancelHideTransorm() {
        if (hiding) clearTimeout(hiding)
      }

      for (var i = 0; i < splinePointsLength; i++) {
        addSplineObject(positions[i])
      }

      positions = []

      for (var j = 0; j < splinePointsLength; j++) {
        positions.push(splineHelperObjects[j].position)
      }

      geometry = new THREE.BufferGeometry()
      geometry.addAttribute(
        'position',
        new THREE.BufferAttribute(new Float32Array(ARC_SEGMENTS * 3), 3)
      );

      curve = new THREE.CatmullRomCurve3(positions)
      curve = new THREE.CatmullRomCurve3(positions)
      curve.curveType = "centripetal"
      curve.mesh = new THREE.Line(
        geometry.clone(),
        new THREE.LineBasicMaterial({
          color: 0x00ff00,
          opacity: 0.35
        })
      )
      curve.mesh.castShadow = true
      splines.centripetal = curve

      for (var k in splines) {
        var spline = splines[k]
        scene.add(spline.mesh)
      }

      load([
        new THREE.Vector3(
          -17.895335933485768,
          -183.78246670863535,
          46.40722438024568
        ),
        new THREE.Vector3(
          31.532072380355828,
          -38.599241009846025,
          -130.5180120519768
        ),
        new THREE.Vector3(
          55.18118816565621,
          135.5274012702808,
          103.20807238070202
        ),
        new THREE.Vector3(
          -54.92263971605058,
          180.26441311832957,
          -64.18336979324854
        )
      ]);
      function animateCamera() {
        followCamera = !followCamera
      }
      function addSplineObject(position) {
        var material = new THREE.MeshLambertMaterial({
          color: Math.random() * 0xffffff
        });
        var object = new THREE.Mesh(geometry, material)

        if (position) {
          object.position.copy(position);
        } else {
          object.position.x = Math.random() * 1000 - 500;
          object.position.y = Math.random() * 600;
          object.position.z = Math.random() * 800 - 400;
        }

        object.castShadow = true
        object.receiveShadow = true
        scene.add(object)
        splineHelperObjects.push(object)
        return object
      }

      function addPoint() {
        splinePointsLength++
        positions.push(addSplineObject().position)
        updateSplineOutline()
      }

      function removePoint() {
        if (splinePointsLength <= 4) {
          return;
        }
        splinePointsLength--
        positions.pop()
        scene.remove(splineHelperObjects.pop())
        updateSplineOutline()
      }

      function updateSplineOutline() {
        for (var k in splines) {
          var spline = splines[k]

          var splineMesh = spline.mesh
          var position = splineMesh.geometry.attributes.position

          for (var i = 0; i < ARC_SEGMENTS; i++) {
            var t = i / (ARC_SEGMENTS - 1)
            spline.getPoint(t, point)
            position.setXYZ(i, point.x, point.y, point.z)
          }

          position.needsUpdate = true
        }
      }

      function exportSpline() {
        var strplace = []

        for (var j = 0; j < splinePointsLength; j++) {
          var p = splineHelperObjects[j].position
          strplace.push(
            "new THREE.Vector3({0}, {1}, {2})".format(p.x, p.y, p.z)
          );
        }

        // console.log(strplace.join(",\n"))
        var code = "[" + strplace.join(",\n\t") + "]";
        prompt("copy and paste code", code)
      }

      function load(new_positions) {
        while (new_positions.length > positions.length) {
          addPoint()
        }

        while (new_positions.length < positions.length) {
          removePoint()
        }

        for (var i = 0; i < positions.length; i++) {
          positions[i].copy(new_positions[i])
        }

        updateSplineOutline()
      }

      window.addEventListener("resize", this.onWindowResize, false)

      stats = new Stats()
      this.$el.appendChild(stats.dom)
      function render() {
        renderer.render(scene, camera)
      }
    },
    animate() {
      requestAnimationFrame(this.animate);
      if (mixers.length > 0) {
        for (var i = 0; i < mixers.length; i++) {
          mixers[i].update(clock.getDelta())
        }
      }

      camPosIndex += 10
      if (camPosIndex > 10000) {
        camPosIndex = 0
      }
      var camPos = curve.getPoint(camPosIndex / 10000)
      var camRot = curve.getTangent(camPosIndex / 10000)

      splineCamera.position.x = camPos.x
      splineCamera.position.y = camPos.y
      splineCamera.position.z = camPos.z

      splineCamera.rotation.x = camRot.x
      splineCamera.rotation.y = camRot.y
      splineCamera.rotation.z = camRot.z

      if ( model ) {
        splineCamera.lookAt(model.position)
      }

      renderer.render(scene, followCamera ? splineCamera: camera)
      controls.update()
      stats.update()
    },
    onWindowResize() {
      camera.aspect = window.innerWidth / window.innerHeight
      camera.updateProjectionMatrix()
      renderer.setSize(window.innerWidth, window.innerHeight)
    }
  },
  mounted() {
    this.init();
    this.animate();
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
