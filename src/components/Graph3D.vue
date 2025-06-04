<template>
  <div ref="graphContainer" style="width: 100%; height: 100vh"></div>
</template>

<script setup>
import { onMounted, ref, onUnmounted } from "vue";
import ForceGraph3D from "3d-force-graph";
import * as THREE from "three";

const graphContainer = ref(null);
let graph = null;
let animationId = null;

const categories = ["科技", "艺术", "教育", "医疗", "娱乐"];

const initGraph = () => {
  // 让中间的球在原点
  const nodes = categories.map((name, i) => ({
    id: name,
    val: 10,
    group: "main",
    fx: (i - (categories.length - 1) / 2) * 60,
    fy: 0,
    fz: 0,
  }));

  const data = {
    nodes,
    links: [],
  };

  graph = ForceGraph3D()(graphContainer.value)
    .graphData(data)
    .nodeLabel("id")
    .nodeThreeObject((node) => {
      let geometry;
      let color;

      // Create different geometries and colors based on category
      switch (node.id) {
        case "科技":
          geometry = new THREE.OctahedronGeometry(5);
          color = 0x00ff88; // Cyan-green
          break;
        case "艺术":
          geometry = new THREE.TorusGeometry(4, 2, 16, 32);
          color = 0xff3366; // Pink-red
          break;
        case "教育":
          geometry = new THREE.DodecahedronGeometry(5);
          color = 0x4488ff; // Blue
          break;
        case "医疗":
          geometry = new THREE.IcosahedronGeometry(5);
          color = 0xffff00; // Yellow
          break;
        case "娱乐":
          geometry = new THREE.TetrahedronGeometry(5);
          color = 0xff8800; // Orange
          break;
        default:
          geometry = new THREE.SphereGeometry(4, 16, 16);
          color = 0xffffff;
      }

      const material = new THREE.MeshPhongMaterial({
        color: color,
        emissive: color,
        emissiveIntensity: 0.4,
        specular: 0x111111,
        shininess: 30,
        transparent: true,
        opacity: 0.9,
      });

      const mesh = new THREE.Mesh(geometry, material);
      mesh.userData = {
        scale: 1,
        speed: 0.002,
      };
      return mesh;
    });

  const scene = graph.scene();
  scene.background = new THREE.Color(0x111111);

  // 灯光
  scene.add(new THREE.AmbientLight(0x404040));

  const pointLight1 = new THREE.PointLight(0xffffff, 1.5, 300);
  pointLight1.position.set(100, 100, 100);
  scene.add(pointLight1);

  const pointLight2 = new THREE.PointLight(0xffffff, 1.5, 300);
  pointLight2.position.set(-100, -100, -100);
  scene.add(pointLight2);

  const starsGeometry = new THREE.BufferGeometry();
  const starCount = 1000;
  const positions = [];

  for (let i = 0; i < starCount; i++) {
    const x = (Math.random() - 0.5) * 2000;
    const y = Math.random() * 1000;
    const z = (Math.random() - 0.5) * 2000;
    positions.push(x, y, z);
  }

  starsGeometry.setAttribute(
    "position",
    new THREE.Float32BufferAttribute(positions, 3)
  );

  const starsMaterial = new THREE.PointsMaterial({
    color: 0xffffff,
    size: 2,
    transparent: true,
    opacity: 0.8,
  });

  const starField = new THREE.Points(starsGeometry, starsMaterial);
  scene.add(starField);

  const animate = () => {
    // 球体呼吸动画
    graph.scene().traverse((obj) => {
      if (obj instanceof THREE.Mesh && obj.userData.scale !== undefined) {
        obj.userData.scale =
          0.8 + (0.4 * (Math.sin(Date.now() * obj.userData.speed) + 1)) / 2;
        obj.scale.set(
          obj.userData.scale,
          obj.userData.scale,
          obj.userData.scale
        );
      }
    });

    // 流星运动动画
    const posArray = starsGeometry.attributes.position.array;
    for (let i = 1; i < posArray.length; i += 3) {
      posArray[i] -= 1; // y轴下落

      if (posArray[i] < -500) {
        posArray[i] = 500;
        posArray[i - 1] = (Math.random() - 0.5) * 2000; // x
        posArray[i + 1] = (Math.random() - 0.5) * 2000; // z
      }
    }
    starsGeometry.attributes.position.needsUpdate = true;

    // 渲染
    graph.cameraPosition(graph.cameraPosition());

    animationId = requestAnimationFrame(animate);
  };

  animate();
};

onMounted(() => {
  initGraph();
});

onUnmounted(() => {
  if (animationId) cancelAnimationFrame(animationId);
  if (graph) graph._destructor();
});
</script>

<style scoped>
body {
  margin: 0;
  overflow: hidden;
  background-color: #111;
}
</style>
