<template>
  <canvas ref="canvasRef"></canvas>
</template>

<script setup>
import { ref, onMounted, onUnmounted} from "vue"
import * as THREE from "three"
import { matchedRouteKey } from "vue-router";


//CURSOR
//retrieving the coordinate of the mouse for uptading the camera position in accordance to mouse position
const cursor = {
  x:0,
  y:0
}
window.addEventListener("mousemove", (e)=>{
  cursor.x = e.clientX / sizes.width - 0.5
  cursor.y = - (e.clientY / sizes.height - 0.5) // qui invertiamo perche in threeJS l'asse y ha valore alto quando saliamo e basso quindi scendiamo(il contrario dell'asse x)
  console.log(cursor.x)
})

let canvasRef = ref();

const sizes = {
  width: 800,
  height: 600
}

let scene = new THREE.Scene()

let renderer

// BOX
const boxGeometry = new THREE.BoxGeometry(1,1,1)
const boxMaterial = new THREE.MeshBasicMaterial({color: 0xff0000}) 
const box = new THREE.Mesh(boxGeometry,boxMaterial)


box.position.set(0 , 0, 0)

scene.add(box)

//CAMERAS

//1) CAMERA è una classe astratta sulla quale sono costruite le classi che poi utilizzeremo noi,
// ma questa non è una classe che dovremo utilizzare.

//2) ARRAY CAMERA: viene usata per fare il render della scena da piu telecamere, in specifiche aree del render
// Quest è utile quando per esempio si vuole create un gioco multiplayer in cui i giocatori condividono lo stesso schermo

//3) STEREO CAMERA: viene usata per fare il render della scena da due telecamere, che imitano gli occhi creando un "effetto parallax"
// Viene usato per creare effetti di profondità per i dispositivi VR, gli occhiali rossi e blue, cardboard, ecc..

//4) CUBE CAMERA: la cube camera genera 6 renders, uno per ogni "faccia" del cubo, quindi crea un render del "circondario"

//5) ORTOGRAPHIC CAMERA: viene usata per renderizzare la scena, ma senza prospettiva, quindi se ad esempio un oggetto è a molta distanza 
// dalla telecamera, avra comunque le stesse dimensioni che avrebbe se si trovasse vicino alla telecamera. In sostanza le dimensioni 
// non cambiano anche se è piu lontano.

//Al posto del field of view , andremo ad impostare quanto lontano puo veder la telecamera in ogni direzione (left, right, top and bottom), poi near e far plane
/* 
const aspectRatio = sizes.width/sizes.height // utilizziamo questo parametro , moltiplicandolo per i valor idel campo di vista orizzontale
                                            // cosi da mantenere le proporzioni degli oggetti e non deformarli
const camera = new THREE.OrthographicCamera(
  -1 * aspectRatio, //Left 
  1 * aspectRatio, //Right
  1,              //Top
  -1,             //Bottom
  0.1,            //Near Plane
  100)            //Far Plane
camera.position.set(2,2,2)
camera.lookAt(box.position)
scene.add(camera)
*/

//6) PERSPECTIVE CAMERA: viene usata per renderizzare la scena, con prospettiva.
let camera = new THREE.PerspectiveCamera(
  75, //vertical field of view (quanto puoi vedere, espresso come angolo, in altezza) , valore raccomandato 45 < x < 75
  sizes.width / sizes.height, //aspect ratio
  //Optional
  0.1, //near plane (quanto vicino puo vedere, tutti gli oggetti piu vicini non verranno mostrati)
  100 //far plane (quanto lontano puo vedere, tutti gli oggetti piu lontani non verranno mostrati) 
      //n.b. non impostare i valori di near e far plane troppo alti altrimenti si crea un glitch chiamato z-fighting
      // in pratica , se impostiamo dei valori cosi "estremi come 0,000001 per near plane e 9999999 per far plane, ed dobbiamo mostrare un piano
      // che ha due facce, una rossa e una verde, è possibile che la telecamera  non capisca quale sia il lato che deve renderizzare se il piano si trova
      // troppo vicino o troppo distante dalla telecamera
  
);

camera.position.set(0,0,3)
camera.lookAt(box.position)
scene.add(camera);


const clock = new THREE.Clock()

const animate = () =>{
  const elapsedTime = clock.getElapsedTime()

  //box.rotation.y = elapsedTime

  //update the camera position
  
  //vede i lati del cubo, ma la parte posteriore non è visibile
  //camera.position.x = cursor.x * 10
  //camera.position.y = cursor.y * 10

  //rotazione completa usando seno e coseno
  camera.position.x = Math.sin(cursor.x * Math.PI * 2) * 2
  camera.position.z = Math.cos(cursor.x * Math.PI * 2) * 2
  camera.position.y = cursor.y * 5

  //diciamo alla camera si guardare alla posizione del cubo ad ogni frame
  camera.lookAt(box.position)
  
  renderer.render(scene, camera)
}

onMounted(() => {
 renderer = new THREE.WebGLRenderer({
    canvas: canvasRef.value
  });

 
  renderer.setSize(sizes.width, sizes.height);
  renderer.setPixelRatio(window.devicePixelRatio);
  renderer.render(scene, camera);
  renderer.setAnimationLoop(animate)

});

onUnmounted(() => {
  renderer.setAnimationLoop(null);
});

</script>

<style>
canvas {
  width: 100%;
  height: 100%;
  display: block;
}
</style>