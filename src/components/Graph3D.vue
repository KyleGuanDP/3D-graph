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
  const centerNode = {
    id: "太阳",
    val: 20,
    fx: 0,
    fy: 0,
    fz: 0,
    group: "center",
  };
  const orbitNodes = categories.map((name, i) => ({
    id: name,
    val: 10,
    group: "orbit",
    angle: Math.random() * 2 * Math.PI,
    orbitRadius: 80 + i * 30,
    orbitSpeed: 0.01 + i * 0.002,
  }));

  const nodes = [centerNode, ...orbitNodes];
  const data = { nodes, links: [] };

  graph = ForceGraph3D()(graphContainer.value)
    .graphData(data)
    .nodeLabel("id")
    .nodeThreeObject((node) => {
      const geometry = new THREE.SphereGeometry(
        node.group === "center" ? 6 : 3,
        16,
        16
      );
      const material = new THREE.MeshPhongMaterial({
        color: node.group === "center" ? 0xffdd55 : 0xffffff,
        emissive: node.group === "center" ? 0xffcc00 : 0x888888,
        emissiveIntensity: 0.8,
        shininess: 30,
        transparent: true,
        opacity: 0.9,
      });
      const sphere = new THREE.Mesh(geometry, material);
      sphere.userData = {
        scale: 1,
        speed: 0.004,
      };
      return sphere;
    });

  const scene = graph.scene();
  scene.background = new THREE.Color(0x111111);

  scene.add(new THREE.AmbientLight(0x404040));

  const pointLight = new THREE.PointLight(0xffffff, 1.5, 300);
  pointLight.position.set(100, 100, 100);
  scene.add(pointLight);

  // 🌌 星空背景
  const starsGeometry = new THREE.BufferGeometry();
  const starCount = 3000;
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

  // 🌟 添加轨道环
  orbitNodes.forEach((node) => {
    const ringGeometry = new THREE.RingGeometry(
      node.orbitRadius - 0.5,
      node.orbitRadius + 0.5,
      64
    );
    const ringMaterial = new THREE.MeshBasicMaterial({
      color: 0x888888,
      side: THREE.DoubleSide,
      transparent: true,
      opacity: 0.3,
    });
    const ring = new THREE.Mesh(ringGeometry, ringMaterial);
    ring.rotation.x = -Math.PI / 2;
    scene.add(ring);
  });

  // 🌀 动画循环
  const animate = () => {
    const now = Date.now();

    // 轨道旋转更新位置
    graph.graphData().nodes.forEach((node) => {
      if (node.group === "orbit") {
        node.angle += node.orbitSpeed;
        const x = node.orbitRadius * Math.cos(node.angle);
        const z = node.orbitRadius * Math.sin(node.angle);
        if (node.__threeObj) {
          node.__threeObj.position.set(x, 0, z);
        }
      }
    });

    // 呼吸动画
    graph.scene().traverse((obj) => {
      if (obj instanceof THREE.Mesh && obj.userData.scale !== undefined) {
        obj.userData.scale =
          0.8 + (0.4 * (Math.sin(now * obj.userData.speed) + 1)) / 2;
        obj.scale.set(
          obj.userData.scale,
          obj.userData.scale,
          obj.userData.scale
        );
      }
    });

    // 星星流动
    const posArray = starsGeometry.attributes.position.array;
    for (let i = 1; i < posArray.length; i += 3) {
      posArray[i] -= 3;
      if (posArray[i] < -500) {
        posArray[i] = 500;
        posArray[i - 1] = (Math.random() - 0.5) * 2000;
        posArray[i + 1] = (Math.random() - 0.5) * 2000;
      }
    }
    starsGeometry.attributes.position.needsUpdate = true;

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
