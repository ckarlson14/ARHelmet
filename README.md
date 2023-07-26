# ARExample

THREE.js with jeelizFaceFilter in Vue.js.

## Integrated packages

[Three.js](https://threejs.org/) + [jeelizFaceFilter](https://github.com/jeeliz/jeelizFaceFilter/tree/master)

## Customize configuration

To run with a different .gltf or .glb model:
1. Place the model file in the public/threeJS/models directory.
2. Change the helmetFile property to reference the new model. For example, it's set to load "/threeJS/models/Helmet_Eagle.gltf";


To run on mobile:
1. Start the vite local server
2. Port through ngrok using: http port <PORT>.
3. View the app through the https:// link it gives you

## Project Setup

```sh
npm install
```

### Compile and Hot-Reload for Development

```sh
npm run dev
```

### Compile and Minify for Production

```sh
npm run build
```

