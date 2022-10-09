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
      this.creatWater();
      this.creatWaterPoint();
      await this.create();
      this.createLight();
      this.createCamera();
      this.createRender();
      this.render();
      let controls = new OrbitControls(this.camera, this.renderer.domElement);
      controls.addEventListener('change', this.render);
    },
    createScene() {
      this.scene = new THREE.Scene();
      var axisHelper = new THREE.AxesHelper(250);
      this.scene.add(axisHelper);
    },
    async create() {
      const fbxLoader = new FBXLoader();
      const fbx = await fbxLoader.loadAsync('../static/models/syg06.fbx');
      const fbx2 = await fbxLoader.loadAsync('../static/models/all.fbx');
      console.log(fbx2);
      fbx.traverse(function (child) {
        if (child.isMesh) {
          child.castShadow = true;
          child.receiveShadow = true;
        }
      });
      fbx.rotateX(-Math.PI / 2);
      fbx.translateY(48);
      fbx.translateZ(820);
      fbx.translateX(-200);
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
      point.position.set(400, 200, 300);
      this.scene.add(point);
      let ambient = new THREE.AmbientLight(0xdddddd);
      this.scene.add(ambient);
    },
    createCamera() {
      let width = window.innerWidth;
      let height = window.innerHeight;
      let k = width / height;
      this.camera = new THREE.PerspectiveCamera(35, k, 0.1, 100000);
      this.camera.position.set(3000, 4000, 2000);
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
      this.waterArr.forEach((ele) => {
        ele.material.uniforms['time'].value += 1.0 / 60.0;
      });
      // this.water.material.uniforms['time'].value += 1.0 / 60.0;
      this.renderer.render(this.scene, this.camera);
      requestAnimationFrame(this.render);
    },
    creatWaterPoint() {
      let points1 = [
        new THREE.Vector2(650, 100),
        new THREE.Vector2(650, 150),
        new THREE.Vector2(760, 150),
        new THREE.Vector2(760, 100),
        new THREE.Vector2(2195, 140),
        new THREE.Vector2(2195, 50),
        new THREE.Vector2(2220, 50),
        new THREE.Vector2(2220, 120),
        new THREE.Vector2(2500, 25),
        new THREE.Vector2(2500, -2500),
        new THREE.Vector2(-2500, -2500),
        new THREE.Vector2(-2500, 0)
      ];
      let points2 = [
        new THREE.Vector2(670, 300),
        new THREE.Vector2(760, 300),
        new THREE.Vector2(760, 330),
        new THREE.Vector2(770, 370),
        new THREE.Vector2(1560, 370),
        new THREE.Vector2(1550, 720),
        new THREE.Vector2(1580, 720),
        new THREE.Vector2(1580, 780),
        new THREE.Vector2(1430, 935),
        new THREE.Vector2(840, 960),
        new THREE.Vector2(830, 990),
        new THREE.Vector2(810, 1000),
        new THREE.Vector2(760, 1220),
        new THREE.Vector2(680, 1220),
        new THREE.Vector2(640, 960),
        new THREE.Vector2(630, 950),
        new THREE.Vector2(620, 530),
        new THREE.Vector2(670, 480)
      ];
      let points3 = [
        new THREE.Vector2(680, 1280),
        new THREE.Vector2(760, 1280),
        new THREE.Vector2(770, 1350),
        new THREE.Vector2(810, 1375),
        new THREE.Vector2(810, 1860),
        new THREE.Vector2(650, 1860),
        new THREE.Vector2(650, 1375)
      ];
      let points4 = [
        new THREE.Vector2(810, 1900),
        new THREE.Vector2(640, 1900),
        new THREE.Vector2(615, 1970),
        new THREE.Vector2(810, 1990)
      ];
      let points5 = [
        new THREE.Vector2(620, 2000),
        new THREE.Vector2(795, 2025),
        new THREE.Vector2(690, 2500),
        new THREE.Vector2(490, 2500),
        new THREE.Vector2(580, 2100)
      ];
      // let curve = new THREE.SplineCurve([
      let points6 = [
        new THREE.Vector2(460, 2480),
        new THREE.Vector2(460, 2440),
        new THREE.Vector2(-400, 2350),
        new THREE.Vector2(-365, 2000),
        new THREE.Vector2(-370, 2000),
        new THREE.Vector2(-450, 2350),
        new THREE.Vector2(-650, 2370),
        new THREE.Vector2(-900, 2420),

        new THREE.Vector2(-920, 2420),
        new THREE.Vector2(-920, 2200),
        new THREE.Vector2(-1050, 2080),
        new THREE.Vector2(-1180, 2030),
        new THREE.Vector2(-1600, 1950),
        new THREE.Vector2(-1740, 1950),
        new THREE.Vector2(-1740, 2000),
        new THREE.Vector2(-1200, 2050),
        new THREE.Vector2(-1050, 2120),
        new THREE.Vector2(-950, 2200),
        new THREE.Vector2(-950, 2400),

        new THREE.Vector2(-1100, 2400),
        new THREE.Vector2(-1200, 2500),
        new THREE.Vector2(-1150, 2500),

        new THREE.Vector2(-1100, 2430),

        new THREE.Vector2(-900, 2440),
        new THREE.Vector2(-620, 2410),

        new THREE.Vector2(-620, 2500),
        new THREE.Vector2(-600, 2500),
        new THREE.Vector2(-600, 2410),
        new THREE.Vector2(-590, 2410)
        // ]);
      ];
      // this.creatWater(curve.getPoints(1000));
      this.creatWater(points6);
      this.creatWater(points1);
      this.creatWater(points2);
      this.creatWater(points3);
      this.creatWater(points4);
      this.creatWater(points5);
    },
    creatWater(points) {
      let shape = new THREE.Shape(points);
      let waterGeometry = new THREE.ShapeGeometry(shape, 25);
      // const waterGeometry = new THREE.PlaneGeometry(6000, 2550);
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
        waterColor: 0x0043ac,
        distortionScale: 3.7,
        fog: this.scene.fog !== undefined,
        side: THREE.DoubleSide
      });
      this.water.rotation.x = -Math.PI / 2;
      this.water.position.y = 5;
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
