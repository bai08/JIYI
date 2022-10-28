<template>
  <div id="hello">
    <div></div>
  </div>
</template>

<script>
import * as THREE from 'three';
import { newWebSocket } from '@/utils/bwWebScocket';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
import { FBXLoader } from 'three/examples/jsm/loaders/FBXLoader';
import { Water } from 'three/examples/jsm/objects/Water';
import { Sky } from 'three/examples/jsm/objects/Sky.js';
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
      waterArr: [],
      clock: new THREE.Clock(),
      FPS: 45,
      timeS: 0,
      trackGroup: null,
      dlList: []
    };
  },
  mounted() {
    this.init();
    // this.translateWs();
    this.resizeObserver = new ResizeObserver(() => {
      this.resizeHandle();
    });
    this.resizeObserver.observe(document.getElementById('hello'));
  },
  destroyed() {
    this.destroyWebGL();
    // this.resizeObserver.disconnect();
    newWebSocket.close();
  },
  methods: {
    translateWs() {
      newWebSocket.init({
        url: 'ws://192.168.1.199:8080/test', // 自己的ws 地址192.168.1.115:8082
        onopen: (msg, data) => {
          console.log(msg, data);
          this.isConnect = true;
        },
        onmessage: (data) => {
          console.log(data);
          this.handleRefresh('swim');
        },
        onclose: (data) => {
          console.log(data);
        }
      });
      this.senData('123');
    },
    senData(data) {
      if (this.sendTimeout) clearTimeout(this.sendTimeout);
      this.sendTimeout = setTimeout(() => {
        if (this.isConnect) {
          newWebSocket.send(data);
        } else {
          this.senData(data);
        }
      }, 1000);
    },
    async init() {
      this.element = document.getElementById('hello');
      this.createScene();
      this.createCamera();

      await this.creatDL();
      this.createBgc();
      this.createLight();
      this.createRender();
      this.creatTrack();
      this.render();
    },
    createScene() {
      this.scene = new THREE.Scene();
    },
    async createBgc() {
      const group = new THREE.Group();
      const fbxLoader = new FBXLoader();
      const fbx2 = await fbxLoader.loadAsync('../static/models/map26.fbx');
      fbx2.position.z = -65;
      const waterGeometry = new THREE.PlaneBufferGeometry(244, 244);
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
      this.water.position.y = 0.5;
      group.add(fbx2);
      group.add(this.water);
      group.scale.set(5, 5, 5);
      this.scene.add(group);

      const sky = new Sky();
      sky.scale.setScalar(10000);
      this.scene.add(sky);

      const sun = new THREE.Vector3();
      const skyUniforms = sky.material.uniforms;

      skyUniforms['turbidity'].value = 10;
      skyUniforms['rayleigh'].value = 2;
      skyUniforms['mieCoefficient'].value = 0.005;
      skyUniforms['mieDirectionalG'].value = 0.3;

      const pmremGenerator = new THREE.PMREMGenerator(this.renderer);
      const phi = THREE.MathUtils.degToRad(87);
      const theta = THREE.MathUtils.degToRad(180);

      sun.setFromSphericalCoords(1, phi, theta);

      sky.material.uniforms['sunPosition'].value.copy(sun);
      this.water.material.uniforms['sunDirection'].value.copy(sun).normalize();

      const renderTarget = pmremGenerator.fromScene(sky);
      this.scene.environment = renderTarget.texture;
    },
    creatDL() {
      this.create('1206', 1, 20, -120);
      this.create('1207', -200, 20, -120);
      this.create('1208', 1, 20, -170);
      this.create('1209', -200, 20, -170);
    },
    async create(index, x, y, z) {
      // for (let i = 0; i < 4; i++) {
      let tmpData = {};
      const fbxLoader = new FBXLoader();
      // this.dlList.push({});
      tmpData.allObject = await fbxLoader.loadAsync(
        `../static/models/dushangang${index}.fbx`
      );
      console.log(tmpData);
      tmpData.allObject.scale.set(0.1, 0.1, 0.1);
      tmpData.zuoyouObject = tmpData.allObject.children.find(
        (item) => item.name.indexOf('zuoyou') > -1
      );
      tmpData.shangxiaObject = tmpData.zuoyouObject.children.find(
        (item) => item.name.indexOf('shangxia') > -1
      );
      tmpData.xuanbiObject = tmpData.shangxiaObject.children.find(
        (item) => item.name.indexOf('xuanbi') > -1
      );
      const belt = tmpData.xuanbiObject.children.find(
        (item) => item.name.indexOf('pidai') > -1
      );
      let beltBox = new THREE.Box3().setFromObject(belt);
      console.log(beltBox);
      const obj = this.setTexture(
        beltBox.max.x - beltBox.min.x,
        beltBox.max.z - beltBox.min.z,
        0,
        beltBox.max.x - beltBox.min.x,
        'Left',
        belt
      );
      tmpData.beltBox = {
        X: beltBox.max.x - beltBox.min.x,
        Z: beltBox.max.z - beltBox.min.z
      };
      obj.position.set(0, beltBox.max.y - beltBox.min.y + 1, 0);
      // obj.rotation.set(-0.5 * Math.PI, 0, 0);
      obj.positionStatus = 1;
      tmpData.macBelt = obj;
      // belt.add(obj);
      tmpData.doulunObject = tmpData.shangxiaObject.children.find(
        (item) => item.name.indexOf('doulun') > -1
      );
      this.scene.add(tmpData.allObject);
      // tmpData.allObject.position.set(
      //   i % 2 === 0 ? 1 : -200,
      //   20,
      //   i < 2 ? -120 : -170
      // );
      tmpData.allObject.position.set(x, y, z);
      this.dlList.push(tmpData);
      // }
      // this.runMac();
    },
    createLight() {
      const direLight = new THREE.DirectionalLight(0xffffff, 1.0); // 平行光 DirectionalLight (光源颜色的RGB数值, 光源强度数值)
      direLight.position.set(0, 1000, 0);
      this.scene.add(direLight);
      let ambient = new THREE.AmbientLight(0xdddddd);
      this.scene.add(ambient);
    },
    createCamera() {
      const width = this.element.clientWidth; // 窗口宽度
      const height = this.element.clientHeight; // 窗口高度
      let k = width / height;
      this.camera = new THREE.PerspectiveCamera(45, k, 1, 20000);
      this.camera.position.set(0, 150, 300);
      this.scene.add(this.camera);
    },
    resizeHandle() {
      this.camera.aspect = this.element.clientWidth / this.element.clientHeight;
      this.camera.updateProjectionMatrix();
      this.renderer.setSize(
        this.element.clientWidth,
        this.element.clientHeight
      );
    },
    createRender() {
      this.renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
      this.renderer.setSize(
        this.element.clientWidth,
        this.element.clientHeight
      );
      this.renderer.setClearColor(0xa0a0a0, 1);
      this.element.appendChild(this.renderer.domElement);
      this.controls = new OrbitControls(this.camera, this.renderer.domElement);
      this.controls.target = new THREE.Vector3(0, 0, -200);
      this.controls.minDistance = 120; // 离中心物体的最近距离
      this.controls.maxDistance = 800; //  离中心物体的最远距离
      this.controls.maxPolarAngle = (Math.PI / 3) * 1.2; // 上下翻转的最大角度
      this.controls.minPolarAngle = -Math.PI; // 上下翻转的最小角度
      this.controls.update();
    },
    render() {
      this.animateId = requestAnimationFrame(this.render);
      var T = this.clock.getDelta();
      this.timeS = this.timeS + T;
      if (this.timeS > 1 / this.FPS) {
        this.renderer.render(this.scene, this.camera);
        if (this.water) {
          this.water.material.uniforms['time'].value +=
            1.0 / 60 / (this.FPS / 60);
        }
        this.timerRail();
        this.timeS = 0;
      }
    },
    destroyWebGL() {
      try {
        cancelAnimationFrame(this.animateId);
        this.scene.traverse((child) => {
          if (child.material) {
            child.material.dispose();
          }
          if (child.geometry) {
            child.geometry.dispose();
          }
          child = null;
        });

        // 场景中的参数释放清理或者置空等
        this.renderer.forceContextLoss();
        this.renderer.dispose();
        this.scene.clear();
        this.flows = [];
        this.scene = null;
        this.camera = null;
        this.controls = null;
        this.renderer.content = null;
        this.renderer.domElement = null;
        this.renderer = null;
      } catch (e) {
        console.log('Failed to destroy homepage background:', e);
      }
    },
    deleteObject(obj) {
      // 递归遍历组对象group释放所有后代网格模型绑定几何体占用内存
      obj.geometry.dispose();
      obj.material.dispose();

      // 删除场景对象scene的子对象group
      this.scene.remove(obj);
    },
    drawTrack(length, width, x, z, position) {
      const geo = new THREE.BoxGeometry(length, width, 1);
      const material = new THREE.MeshLambertMaterial({ color: 0xcccccc });
      const track = new THREE.Mesh(geo, material);
      track.rotation.x = -Math.PI / 2;
      track.position.set(x, 10, z);
      this.trackGroup.add(track);
      const obj = this.setTexture(length, width, 1, length, position);
      this.$set(obj, 'railPosition', position);
      track.add(obj);
      this.scene.add(this.trackGroup);
      return obj;
    },
    creatTrack() {
      this.trackGroup = new THREE.Group();
      this.trackGroup.name = 'track';
      this.drawTrack(380, 5, -95, -120, 'Right');
      this.drawTrack(380, 5, -95, -170, 'Right');
      this.drawTrack(295, 5, -45, -230, 'Left');
    },
    timerRail() {
      this.trackGroup.children.forEach((obj) => {
        if (obj.type === 'Mesh') {
          obj.children[0].material.map.offset.x +=
            obj.children[0].railPosition === 'Right' ? -0.05 : 0.05;
        }
      });
      this.dlList.forEach((item) => {
        item.macBelt.material.map.offset.x +=
          item.macBelt.positionStatus === 1 ? 0.05 : -0.05;
      });
    },
    /**
     * @description: 添加皮带
     * @param {*} length    皮带的长度
     * @param {*} width     皮带的宽度
     * @param {*} z         皮带的垂直位置
     * @param {*} num       皮带的贴图个数
     * @param {*} position  皮带的运行方向 上Up 下Down 左Left 右Right
     * @param {*} obj       需要切换方向时传入对应的皮带（可以为空）
     * @return {*} {texture: texture, model: groundBelt}
     */
    setTexture(length, width, z, num, position, obj) {
      const url = require(`@/assets/3d_coalyard_images/arrow${position}.png`);
      const texture = new THREE.TextureLoader().load(url);
      texture.wrapS = texture.wrapT = THREE.RepeatWrapping;
      texture.repeat.set(Math.round((1 / 5) * num), 1);
      let belt;
      if (obj) {
        obj.material = new THREE.MeshBasicMaterial({
          map: texture
        });
        belt = obj;
      } else {
        belt = new THREE.Mesh(
          new THREE.BoxGeometry(length, width, 1),
          new THREE.MeshBasicMaterial({
            map: texture
          })
        );
      }
      belt.position.set(0, 0, z);
      return belt;
    },
    async runMac() {
      this.statusData = (await getDljRunningInfo(this.selectTime)).result[0]; //{ dateTime: date && this.moment(date).format('yyyyMMddhh')}
      this.statusData.forEach((item, index) => {
        this.setRotation(this.dlList[index].xuanzhuanObject, item.hzjd, '100');
        this.setRotation(this.dlList[index].fuyangObject, item.fyjd, '100');
        this.setPosition(this.dlList[index].allObject, item.dljwz);
      });
    },
    /**
     * @description: 设置传入对象旋转角度
     * @param {*} obj 对象
     * @param {*} r 角度
     * @param {*} p 旋转轴
     * @return {*}
     */
    setRotation(obj, r, p) {
      obj.rotation[p] = -(r * Math.PI) / 180;
    },
    /**
     * @description: 设置传入对象移动位置
     * @param {*} obj 对象
     * @param {*} position 位置
     * @return {*}
     */
    setPosition(obj, position) {
      obj.position.x = -500 / 2 + position;
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
