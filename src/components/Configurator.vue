<template>
  <div>
    <v-container>
      <v-row no-gutters align="center" justify="center">
        <v-col sm="0" md="1" lg="1" xl="1"></v-col>
        <v-col sm="6" md="5" lg="5" xl="5">
          <p class="text-left text-h1">Product Configurator for a Wooden Chair</p>
        </v-col>
        <v-col sm="6" md="5" lg="5" xl="5">
          <p class="text-left text-h5 pl-14">
            Make this wooden chair your own by
            <br>selecting different materials.
          </p>
        </v-col>
        <v-col sm="0" md="1" lg="1" xl="1"></v-col>
      </v-row>
    </v-container>
    <v-container>
      <v-row no-gutters>
        <v-col sm="0" md="1"></v-col>
        <v-col sm="12" md="5">
          <v-card elevation="2">
            <div id="container"></div>
          </v-card>
        </v-col>

        <v-col sm="12" md="5">
          <v-card class="ui-card" elevation="2">
            <v-row class="space-row" no-gutters>
              <v-col sm="12" md="12">
                <p class="text-center text-h4 pt-5">Choose Material</p>
              </v-col>
            </v-row>
            <v-divider></v-divider>
            <v-row no-gutters align="center" justify="center" class="pt-10">
              <v-col sm="0" md="2"></v-col>
              <v-col sm="12" md="5">
                <button icon v-on:click="changeMaterial(`N01_BaseColor_Beech`)">
                  <v-avatar size="62">
                    <img src="/textures/beech.jpg">
                  </v-avatar>&nbsp;Original
                </button>
              </v-col>
              <v-col sm="12" md="5">
                <button icon v-on:click="changeMaterial(`N01_BaseColor_Oak`)">
                  <v-avatar size="62">
                    <v-hover>
                      <img src="/textures/oak.jpg">
                    </v-hover>
                  </v-avatar>&nbsp;Oak
                </button>
              </v-col>
              <v-col class="space-col" sm="0" md="2"></v-col>
            </v-row>

            <v-row id="row2" no-gutters align="center" justify="center">
              <v-col sm="0" md="2"></v-col>
              <v-col sm="12" md="5">
                <button v-on:click="changeMaterial(`dark`)">
                  <v-avatar size="62">
                    <img src="/textures/dark.jpg">
                  </v-avatar>&nbsp;Dark Wood
                </button>
              </v-col>
              <v-col sm="12" md="5">
                <button icon v-on:click="changeMaterial(`light`)">
                  <v-avatar color="black" size="62">
                    <img src="/textures/light.jpg">
                  </v-avatar>&nbsp;Light Wood
                </button>
              </v-col>
            </v-row>
          </v-card>
        </v-col>
        <v-col class="space-col" sm="0" md="2"></v-col>
      </v-row>
    </v-container>
  </div>
</template>

<script>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";
export default {
  name: "Configurator",
  data() {
    return {
      camera: null,
      scene: null,
      renderer: null,
      mesh: null,
      controls: null,
      chair: null,
      container: null,
      beechTexture: null,
      oakTexture: null,
      darkTexture: null,
      lightTexture: null,
      loadingScreen: null,
      resources_loaded: null
    };
  },
  methods: {
    init: function() {
      //set resources loaded to false
      this.resources_loaded = false;
      //get canvas container element
      this.container = document.getElementById("container");
      //set up loading screen
      this.loadingScreen = {
        scene: new THREE.Scene(),
        camera: new THREE.PerspectiveCamera(
          70,
          this.container.clientWidth / this.container.clientHeight,
          0.01,
          10
        ),
        box: new THREE.Mesh(
          new THREE.BoxGeometry(0, 0.1, 1),
          new THREE.MeshBasicMaterial({ color: 0x4444ff })
        ),
        box2: new THREE.Mesh(
          new THREE.BoxGeometry(0, 0.1, 1),
          new THREE.MeshBasicMaterial({ color: 0xff0000 })
        )
      };
      this.loadingScreen.scene.background = new THREE.Color(
        "hsl(0, 100%, 100%)"
      );
      //set up loading boxes
      this.loadingScreen.box.position.set(0, 0, 5);
      this.loadingScreen.box2.position.set(0, 0, 5);
      this.loadingScreen.camera.lookAt(this.loadingScreen.box.position);
      this.loadingScreen.scene.add(
        this.loadingScreen.box,
        this.loadingScreen.box2
      );

      //set up loading manager
      const loadingManager = new THREE.LoadingManager();

      loadingManager.onProgress = function(item, loaded, total) {
        console.log(item, loaded, total);
      };

      loadingManager.onLoad = function() {
        console.log("done loading textures");
      };

      //load texture images
      let textureLoader = new THREE.TextureLoader(loadingManager);
      this.beechTexture = textureLoader.load(
        "/textures/N01_BaseColor_Beech.png"
      );
      this.oakTexture = textureLoader.load("/textures/N01_BaseColor_Oak.png");
      this.darkTexture = textureLoader.load("/textures/dark.jpg");
      this.lightTexture = textureLoader.load("/textures/light.jpg");

      //load chair model
      const loader = new GLTFLoader(loadingManager);
      loader.load(
        // resource URL
        "/models/chair/N01_Beech.gltf",
        // called when the resource is loaded
        gltf => {
          this.resources_loaded = true; //set resources loaded to true
          this.chair = gltf.scene;
          this.scene.add(this.chair);
          this.chair.position.set(0, -0.3, 0);
        },
        // called while loading is progressing
        function(xhr) {
          console.log("chair model "+(xhr.loaded / xhr.total) * 100 + "% loaded");
        },
        // called when loading has errors
        function() {
          console.log("An error happened when loading model");
        }
      );

      //set up renderer
      this.renderer = new THREE.WebGLRenderer({ antialias: true });
      this.renderer.setPixelRatio(window.devicePixelRatio);
      this.renderer.setSize(
        this.container.clientWidth,
        this.container.clientHeight
      );
      //set up configurator scene
      this.camera = new THREE.PerspectiveCamera(
        70,
        this.container.clientWidth / this.container.clientHeight,
        0.01,
        10
      );
      this.camera.position.set(0, 0.6, 1);
      this.scene = new THREE.Scene();

      this.scene.background = new THREE.Color("hsl(0, 100%, 100%)");

      const ambientLight = new THREE.HemisphereLight(0xffffff, 0xffffff, 1);
      const mainLight = new THREE.DirectionalLight(0xffffff, 0.4);
      mainLight.position.set(0, 10, 10);
      this.scene.add(ambientLight, mainLight);
      //orbital controls
      this.container.appendChild(this.renderer.domElement);
      this.controls = new OrbitControls(this.camera, this.renderer.domElement);
      this.controls.autoRotate = true;
      this.controls.autoRotateSpeed = 0.5;
      this.controls.enableDamping = true;
      this.controls.dampingFactor = 0.1;
      this.controls.maxDistance = 3;
      this.controls.minDistance = 1;
      this.controls.enablePan = false;
    },
    animate: function() {
      //render loading screen if resources are loading
      if (this.resources_loaded == false) {
        requestAnimationFrame(this.animate);

        this.loadingScreen.box.position.x -= 0.05;
        this.loadingScreen.box2.position.x += 0.05;
        if (this.loadingScreen.box.position.x < -6.5) {
          this.loadingScreen.box.position.x = 6.5;
        }
        if (this.loadingScreen.box2.position.x > 6.5) {
          this.loadingScreen.box2.position.x = -6.5;
        }
        this.loadingScreen.box.position.y = Math.sin(
          this.loadingScreen.box.position.x
        );
        this.loadingScreen.box2.position.y = Math.sin(
          -this.loadingScreen.box.position.x
        );
        this.renderer.render(
          this.loadingScreen.scene,
          this.loadingScreen.camera
        );
        return;
      }

      // configurator if resources are loaded
      requestAnimationFrame(this.animate);
      this.renderer.render(this.scene, this.camera);
    },

    //set new texture material to object on button click
    changeMaterial: function(path) {
      let texture = null;
      switch (path) {
        case "N01_BaseColor_Beech":
          texture = this.beechTexture;
          break;
        case "N01_BaseColor_Oak":
          texture = this.oakTexture;
          break;
        case "dark":
          texture = this.darkTexture;
          break;
        case "light":
          texture = this.lightTexture;
          break;
        default:
          texture = this.beechTexture;
      }

      texture.flipY = false;
      texture.encoding = THREE.sRGBEncoding;

      this.chair.traverse(function(object) {
        if (object instanceof THREE.Mesh) {
          object.material.map = texture;
        }
      });
    },
    //Handle window resize, set camera aspect/renderer size
    resizeHandler: function() {
      this.camera.aspect =
        this.container.clientWidth / this.container.clientHeight;
      this.camera.updateProjectionMatrix();
      this.renderer.setSize(
        this.container.clientWidth,
        this.container.clientHeight
      );
    }
  },
  mounted() {
    this.init();
    this.animate();
  },
  created() {
    //detect window resize
    window.addEventListener("resize", this.resizeHandler);
  },
  destroyed() {
    //remove event listener when resize over
    window.removeEventListener("resize", this.resizeHandler);
  }
};
</script>

<style scoped>
#container {
  width: 100%;
  height: 50vh;
  cursor: grab;
}

.ui-card {
  width: 100%;
  height: 50vh;
}
#row2 {
  padding-top: 10%;
}
button:hover {
  font-weight: bold;
}
@media (max-width: 600px) {
  .space-col {
    display: none;
  }
}
</style>
