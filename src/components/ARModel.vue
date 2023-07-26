<template>
  <div>
    <div class="color-picker">
      <div>
        <Chrome showButtons="false" v-model="helmetColor" />
        <label class="label" for="helmetColor">Helmet Color</label>
      </div>

      <div>
        <Chrome showButtons="false" v-model="padsColor" />
        <label class="label" for="padsColor">Pads Color</label>
      </div>

      <div>
        <Chrome showButtons="false" v-model="facemaskColor" />
        <label class="label" for="facemaskColor">Facemask Color</label>
      </div>

      <div>
        <Chrome showButtons="false" v-model="stripe1Color" />
        <label class="label" for="stripe1Color">Stripe 1 Color</label>
      </div>

      <div>
        <Chrome showButtons="false" v-model="stripe2Color" />
        <label class="label" f for="stripe2Color">Stripe 2 Color</label>
      </div>
      <!-- <button @click="setImage" role="button">Add Image</button> -->
      <input type="file" @change="onFileChange" />
    </div>

    <!-- <div id="container"> -->
    <canvas id="jeeFaceFilterCanvas" ref="faceFilterCanvas"></canvas>
    <canvas id="videoCanvas"></canvas>
    <!-- </div> -->
  </div>
</template>

<script>
// import * as THREE from 'three';
// import { RGBAToHexA } from "@/util/colorFix.js";
import { Chrome } from "@lk77/vue3-color";
import { GLTFLoader } from "three/addons/loaders/GLTFLoader.js";
// import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";

/* global JEELIZFACEFILTER, JeelizThreeHelper, THREE, JeelizResizer  */

// Your existing code goes here

const SETTINGS = {
  offsetXYZ: [0, 0.5, -0.5], // offset of the model in 3D along vertical and depth axis
  scale: 2.0,
  pivotOffsetYZ: [-0.2, 0],
};

let THREECAMERA = null;

function init_videoCanvas(videoElement) {
  const videoCanvas = document.getElementById("videoCanvas");
  const videoCanvas2 = document.getElementById("jeeFaceFilterCanvas");
  const videoContext = videoCanvas.getContext("2d");
  console.log("init video canvas");

  function updateVideoCanvas() {
    // console.log(
    //   "update video canvas",
    //   videoElement.videoWidth,
    //   videoElement.videoHeight
    // );
    videoCanvas.width = videoElement.videoWidth;
    videoCanvas.height = videoElement.videoHeight;
    videoCanvas2.width = videoElement.videoWidth;
    videoCanvas2.height = videoElement.videoHeight;
    videoContext.drawImage(
      videoElement,
      0,
      0,
      videoCanvas.width,
      videoCanvas.height
    );
    // if (THREECAMERA) JeelizThreeHelper.update_camera(THREECAMERA);
    JEELIZFACEFILTER.resize();
    requestAnimationFrame(updateVideoCanvas);
  }
  updateVideoCanvas();
}

// build the 3D. called once when Jeeliz Face Filter is OK

let loadedModel = null;

function init_threeScene(spec, modelURL) {
  console.log("init threejs", modelURL);
  JEELIZFACEFILTER.resize();
  const threeStuffs = JeelizThreeHelper.init(spec, null, THREE);

  JeelizThreeHelper.set_pivotOffsetYZ(SETTINGS.pivotOffsetYZ);

  // Create a group to hold the helmet model. We will add this to the faceObject instead of the model itself.
  const helmetGroup = new THREE.Object3D();
  threeStuffs.faceObject.add(helmetGroup);

  const gltfLoader = new GLTFLoader();
  gltfLoader.load(modelURL, function (gltf) {
    console.log("load model URL", modelURL);
    gltf.scene.traverse(function (child) {
      console.log("gltf child", child);
      // if (child.isMesh) {
      //   // child.material.depthWrite = true;
      // }

      // if (child.isMesh && child.material && child.material.name === "Logo") {
      //   console.log("found logo", child);
      //   child.renderOrder = 3;
      // } else {
      //   child.renderOrder = 1;
      // }
    });

    // gltf.scene.frustumCulled = false;

    var pointLight1 = new THREE.PointLight(0xffffff, 0.9);
    pointLight1.position.set(5, 0, 0);
    gltf.scene.add(pointLight1);

    var pointLight2 = new THREE.PointLight(0xffffff, 0.9);
    pointLight2.position.set(-5, 0, 0);
    gltf.scene.add(pointLight2);

    var pointLight3 = new THREE.PointLight(0xffffff, 0.9);
    pointLight3.position.set(0, 5, 0);
    gltf.scene.add(pointLight3);

    var pointLight4 = new THREE.PointLight(0xffffff, 0.9);
    pointLight4.position.set(0, -5, 0);
    gltf.scene.add(pointLight4);

    // var ambientLight = new THREE.AmbientLight(0xffffff, 0.9);
    // gltf.scene.add(ambientLight);

    // center and scale the object:
    const bbox = new THREE.Box3().expandByObject(gltf.scene);

    gltf.scene.position.add(
      new THREE.Vector3(
        SETTINGS.offsetXYZ[0],
        SETTINGS.offsetXYZ[1],
        SETTINGS.offsetXYZ[2]
      )
    );

    // scale the model according to its width:
    const sizeX = bbox.getSize(new THREE.Vector3()).x;
    gltf.scene.scale.multiplyScalar(SETTINGS.scale / sizeX);

    gltf.scene.rotation.y = -Math.PI / 2;
    // gltf.scene.position.z -= 2;
    // Add the model to the helmet group instead of the faceObject.
    helmetGroup.add(gltf.scene);
    loadedModel = helmetGroup;
    console.log("added faceobject");
  });


  THREECAMERA = JeelizThreeHelper.create_camera();
}

function changeMaterialColor(object, materialName, newColor) {
  if (object) {
    object.traverse((child) => {
      if (child.isMesh)
        console.log(
          "found child mesh",
          child,
          child.material?.name,
          materialName,
          child.material?.name === materialName
        );
      if (child.isMesh && child.material.name === materialName) {
        console.log("set material color", newColor);
        child.material.color.set(newColor);
        child.material.needsUpdate = true;
      }
    });
  }
}
function addImage(object, URL) {
  console.log("add image", object, URL);
  if (object) {
    object.traverse((child) => {
      // console.log("child", child);
      // if (child.isMesh && child.material.name === "Helmet") {
      if (child.isMesh && child.material && child.material.name === "Logo") {
        console.log("found logo", child);
        const texture = new THREE.TextureLoader().load(URL);
        console.log("loaded texture", texture)
        child.material.map = texture;
        child.material.needsUpdate = true;
      } 
    });
  }
}

const destroyJeeliz = async () => {
  await JEELIZFACEFILTER.destroy();
};

function main(modelURL) {
  console.log("main load jeeliz", modelURL);
  // await JEELIZFACEFILTER?.destroy();
  console.log("existing destroyed");
  try {
    JeelizResizer.size_canvas({
      canvasId: "jeeFaceFilterCanvas",
      isFullScreen: true,

      callback: function (isError, bestVideoSettings) {
        start(modelURL, bestVideoSettings);
      },
      onResize: function () {
        // Add this line
        console.log("resize video canvas");
        document.getElementById("videoCanvas").style.transform =
          document.getElementById("jeeFaceFilterCanvas").style.transform;
        // JEELIZFACEFILTER.resize();
      },
    });
  } catch (e) {
    console.log("error resizing", e);
  }
}

function start(modelURL, bestVideoSettings) {
  console.log("start facefilter", modelURL, bestVideoSettings);
  JEELIZFACEFILTER.init({
    videoSettings: bestVideoSettings,
    followZRot: true,
    canvasId: "jeeFaceFilterCanvas",
    NNCPath: "/threeJS/neuralNets/", //root of NN_DEFAULT.json file
    callbackReady: function (errCode, spec) {
      if (errCode) {
        console.log("AN ERROR HAPPENS. SORRY BRO :( . ERR =", errCode);
        return;
      }
      console.log("INFO: JEELIZFACEFILTER IS READY", spec);

      const videoElement = spec.videoElement;
      console.log("found video element", videoElement);
      let videoInitialized = false;
      let timeOutID;
      //   init_videoCanvas(videoElement);
      const checkVideoPlaying = () => {
        console.log("initialized", videoInitialized);
        if (
          !videoInitialized &&
          videoElement.readyState >= 2 &&
          videoElement.videoWidth > 0 &&
          videoElement.videoHeight > 0
        ) {
          console.log("set video initialized");
          videoInitialized = true;
          init_videoCanvas(videoElement);
        } else if (!videoInitialized) {
          timeOutID = setTimeout(checkVideoPlaying, 100);
        } else {
          console.log("clear timeout");
          clearTimeout(timeOutID);
        }
      };
      if (!videoInitialized) {
        checkVideoPlaying();
      } else {
        console.log("clear timeout2");
        clearTimeout(timeOutID);
      }

      init_threeScene(spec, modelURL);
      //   JEELIZFACEFILTER.render_video();
    }, //end callbackReady()

    // called at each render iteration (drawing loop):
    callbackTrack: function (detectState) {
      JeelizThreeHelper.render(detectState, THREECAMERA);
    },
  }); //end JEELIZFACEFILTER.init call
} //end start()

export default {
  components: { Chrome },

  data() {
    return {
      selectedColor: "blue",
      helmetColor: {
        hex: "#FFFFFF",
      },
      facemaskColor: {
        hex: "#FFFFFF",
      },
      padsColor: {
        hex: "#FFFFFF",
      },
      stripe1Color: {
        hex: "#FFFFFF",
      },
      stripe2Color: {
        hex: "#FFFFFF",
      },

      imageURL: "",
    };
  },
  mounted() {
    this.addCanvas();
  },
  computed: {
    helmetFile() {
      return "/threeJS/models/Helmet_Eagle.gltf";
    },
  },
  watch: {
    imageURL: {
      deep: true,
      handler(newValue) {
        console.log("image added", newValue.hex);

        this.addImage(loadedModel, this.imageURL);

        // this.changeMaterialColor(newValue.hex);
        // this.changeMaterialColor(newValue.hex);

        // this.changeMaterialColor(loadedModel, "Helmet", newValue.hex);
      },
    },
    helmetColor: {
      deep: true,
      handler(newValue) {
        console.log("helmet color changed", newValue);
        this.changeMaterialColor(loadedModel, "Outer_Wrap", newValue.hex);
      },
    },
    facemaskColor: {
      deep: true,
      handler(newValue) {
        console.log("facemaskColor color changed", newValue);
        this.changeMaterialColor(loadedModel, "Facemask", newValue.hex);
      },
    },
    stripe1Color: {
      deep: true,
      handler(newValue) {
        console.log("stripe1Color color changed", newValue);
        this.changeMaterialColor(loadedModel, "Stripe1", newValue.hex);
      },
    },
    stripe2Color: {
      deep: true,
      handler(newValue) {
        console.log("stripe2Color color changed", newValue);
        this.changeMaterialColor(loadedModel, "Stripe2 ", newValue.hex);
      },
    },
    padsColor: {
      deep: true,
      handler(newValue) {
        console.log("pads color changed", newValue);
        this.changeMaterialColor(loadedModel, "Padding", newValue.hex);
      },
    },
  },
  methods: {
    addCanvas() {
      //   this.$refs.faceFilterCanvas.width = 600;
      //   this.$refs.faceFilterCanvas.height = 600;
      this.main(this.helmetFile);
    },
    start,
    main,
    destroyJeeliz,
    changeMaterialColor,
    addImage,
    onFileChange(e) {
      const file = e.target.files[0];
      this.createImageUrl(file);
    },
    createImageUrl(file) {
      // Create a URL for the uploaded image file
      this.imageURL = URL.createObjectURL(file);
      // this.addImage(loadedModel, this.imageURL);
    },
    // setImage(){
    //   this.addImage(this.loadedModel, this.imageURL)
    // },
  },
};
</script>

<style scoped>
/* .helmet-view {
  z-index: 999;
} */

#videoCanvas {
  z-index: 10;
}

#jeeFaceFilterCanvas {
  z-index: 20;
}

.color-picker {
  display: flex;
  flex: 1;
  flex-direction: row;
  z-index: 30;
}

.label{
  color: white
}

canvas {
  /* z-index: 20; */
  position: absolute;
  width: 100%;
  height: 100%;
  margin-top: 150px;
  top: 150px;
  left: 0px;
}
</style>
