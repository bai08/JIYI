<template>
  <div class="hello">
    <!-- <h1>{{ msg }}111</h1> -->
    <!-- <p>
      For a guide and recipes on how to configure / customize this project,<br>
      check out the
      <a href="https://cli.vuejs.org" target="_blank" rel="noopener">vue-cli documentation</a>.
    </p> -->
    <!-- <h3>Installed CLI Plugins</h3>
    <ul>
      <li><a href="https://github.com/vuejs/vue-cli/tree/dev/packages/%40vue/cli-plugin-babel" target="_blank" rel="noopener">babel</a></li>
      <li><a href="https://github.com/vuejs/vue-cli/tree/dev/packages/%40vue/cli-plugin-router" target="_blank" rel="noopener">router</a></li>
      <li><a href="https://github.com/vuejs/vue-cli/tree/dev/packages/%40vue/cli-plugin-vuex" target="_blank" rel="noopener">vuex</a></li>
      <li><a href="https://github.com/vuejs/vue-cli/tree/dev/packages/%40vue/cli-plugin-eslint" target="_blank" rel="noopener">eslint</a></li>
    </ul>
    <h3>Essential Links</h3>
    <ul>
      <li><a href="https://vuejs.org" target="_blank" rel="noopener">Core Docs</a></li>
      <li><a href="https://forum.vuejs.org" target="_blank" rel="noopener">Forum</a></li>
      <li><a href="https://chat.vuejs.org" target="_blank" rel="noopener">Community Chat</a></li>
      <li><a href="https://twitter.com/vuejs" target="_blank" rel="noopener">Twitter</a></li>
      <li><a href="https://news.vuejs.org" target="_blank" rel="noopener">News</a></li>
    </ul>
    <h3>Ecosystem</h3> -->
    <!-- <ul>
      <li><a href="https://router.vuejs.org" target="_blank" rel="noopener">vue-router</a></li>
      <li><a href="https://vuex.vuejs.org" target="_blank" rel="noopener">vuex</a></li>
      <li><a href="https://github.com/vuejs/vue-devtools#vue-devtools" target="_blank" rel="noopener">vue-devtools</a></li>
      <li><a href="https://vue-loader.vuejs.org" target="_blank" rel="noopener">vue-loader</a></li>
      <li><a href="https://github.com/vuejs/awesome-vue" target="_blank" rel="noopener">awesome-vue</a></li>
    </ul> -->
  </div>
</template>

<script>
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'
import { FBXLoader } from 'three/examples/jsm/loaders/FBXLoader'
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  data() {
    return {
      scene: null,
      renderer: null,
      camera: null
    }
  },
  mounted() {
    this.init()
  },
  methods: {
    async init() {
      this.createScene();
      await this.create();
      this.createLight();
      this.createCamera();
      this.createRender();
      this.render();
      let controls = new OrbitControls(this.camera, this.renderer.domElement);
      controls.addEventListener("change", this.render);
    },
    createScene() {
      this.scene = new THREE.Scene(); 
    },
    async create() {
      const fbxLoader = new FBXLoader();
      const fbx = await fbxLoader.loadAsync('../static/models/syg06.fbx');
      console.log(fbx);
      fbx.traverse(function (child) {
        if (child.isMesh) {
          child.castShadow = true;
          child.receiveShadow = true;
        }
      });
      this.scene.add(fbx);
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
      let s = 200;
      this.camera = new THREE.OrthographicCamera(-s * k, s * k, s, -s, 1, 1000);
      this.camera.position.set(200, 300, 200);
      this.camera.lookAt(this.scene.position);
    },
    createRender() {
      this.renderer = new THREE.WebGLRenderer();
      this.renderer.setSize(window.innerWidth, window.innerHeight);
      this.renderer.setClearColor(0xa0a0a0, 1);
      document.body.appendChild(this.renderer.domElement);
    },
    render() {
      this.renderer.render(this.scene, this.camera);
      requestAnimationFrame(this.render);
    }
  }

}
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
