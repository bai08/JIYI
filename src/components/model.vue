<template>
  <div class="hello"></div>
</template>

<script>
import * as THREE from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
import { FBXLoader } from 'three/examples/jsm/loaders/FBXLoader';
import { Water } from 'three/examples/jsm/objects/Water';
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  data() {
    return {
      scene: null,
      renderer: null,
      camera: null,
      water: null,
      waterArr: []
    };
  },
  mounted() {
    this.init();
  },
  methods: {
    async init() {
      this.createScene();
      await this.create();
      this.creatWater();
      this.createLight();
      this.createCamera();
      this.createRender();
      this.render();
      let controls = new OrbitControls(this.camera, this.renderer.domElement);
    },
    createScene() {
      this.scene = new THREE.Scene();
      // var axisHelper = new THREE.AxesHelper(250);
      // this.scene.add(axisHelper);
    },
    async create() {
      const fbxLoader = new FBXLoader();
      const fbx = await fbxLoader.loadAsync('../static/models/syg06.fbx');
      const fbx2 = await fbxLoader.loadAsync('../static/models/map26.fbx');
      console.log(fbx2);
      fbx.traverse(function (child) {
        if (child.isMesh) {
          child.castShadow = true;
          child.receiveShadow = true; 
        }
      });
      fbx.rotateX(-Math.PI / 2);
      fbx.translateY(1);
      fbx.scale.set(0.1, 0.1, 0.1);
      // fbx.translateZ(820);
      // fbx.translateX(-200);
      fbx2.traverse(function (child) {
        if (child.isMesh) {
          child.castShadow = true;
          child.receiveShadow = true;
        }
      });
      this.scene.add(fbx);
      this.scene.add(fbx2);
      console.log(this.scene);
    },
    createLight() {
      let point = new THREE.PointLight(0xffffff);
      point.position.set(4000, 2000, 3000);
      this.scene.add(point);
      let ambient = new THREE.AmbientLight(0xdddddd);
      this.scene.add(ambient);
    },
    createCamera() {
      let width = window.innerWidth;
      let height = window.innerHeight;
      let k = width / height;
      this.camera = new THREE.PerspectiveCamera(35, k, 0.1, 100000);
      this.camera.position.set(300, 400, 200);
      this.camera.lookAt(this.scene.position);
      this.scene.add(this.camera);
      window.addEventListener('resize', () => {
        this.camera.aspect = window.innerWidth / window.innerHeight;
        this.camera.updateProjectionMatrix();
        this.renderer.setSize(window.innerWidth, window.innerHeight);
      });
    },
    createRender() {
      this.renderer = new THREE.WebGLRenderer({
        antialias: true,
        logarithmicDepthBuffer: true
      });
      this.renderer.setSize(window.innerWidth, window.innerHeight);
      this.renderer.setClearColor(0xa0a0a0, 1);
      document.body.appendChild(this.renderer.domElement);
    },
    render() {
      this.water.material.uniforms['time'].value += 1.0 / 60;
      this.renderer.render(this.scene, this.camera);
      requestAnimationFrame(this.render);
    },
    creatWater() {
      const waterGeometry = new THREE.PlaneGeometry(244, 244);
      this.water = new Water(waterGeometry, {
        textureWidth: 512,
        textureHeight: 512,
        waterNormals: new THREE.TextureLoader().load(
          '../static/waternormals.jpg',
          function (texture) {
            texture.wrapS = texture.wrapT = THREE.RepeatWrapping;
          }
        ),
        sunDirection: new THREE.Vector3(),
        sunColor: 0xffffff,
        waterColor: 0x003896,
        distortionScale: 3.7,
        fog: this.scene.fog !== undefined,
        side: THREE.DoubleSide
      });
      this.water.rotation.x = -Math.PI / 2;
      this.water.position.y = 0.8;
      this.water.position.z = 65;
      this.scene.add(this.water);
      this.waterArr.push(this.water);
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
body {
  margin: 0;
  overflow: hidden;
}

h3 {
  margin: 40px 0 0;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}
</style>
