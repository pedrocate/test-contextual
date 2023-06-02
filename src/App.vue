<template>
  <div class="w-full h-full">
    <div ref="graphPanel"></div>
    <div v-if="loading" class="w-full h-full z-50 bg-slate-400 absolute left-0 top-0 opacity-60 flex items-center">
      <Loading />
    </div>
  </div>
</template>

<script setup>
import { onMounted, ref } from "vue";
import ForceGraph3D from "3d-force-graph";
import SpriteText from "three-spritetext";
import Loading from "./icons/Loading.vue";
import { UnrealBloomPass } from "three/examples/jsm/postprocessing/UnrealBloomPass";

const graphPanel = ref(null);
const myGraph = ForceGraph3D();

const data = {
  nodes: [
    {
      id: "id1",
      name: "Real Madrid C.F.",
      val: 10,
      color: "rgb(218, 211, 211)",
    },
    {
      id: "id11",
      name: "Champions",
      val: 8,
      color: "rgb(192, 192, 21)",
      childs: [
        {
          id: "id112",
          name: "Manchester City",
          color: "gray",
          val: 5,
        },
      ],
    },
    {
      id: "id12",
      name: "F.C. Barcelona",
      val: 8,
      color: "rgb(91, 161, 227)",
      childs: [
        {
          id: "id121",
          name: "Dembélé",
          color: "rgb(244, 103, 103)",
          val: 5,
        },
        {
          id: "id122",
          name: "Xavi",
          color: "greenyellow",
          val: 5,
          childs: [
            {
              id: "id1221",
              name: "Iniesta",
              color: "pink",
              val: 3,
            },
          ],
        },
      ],
    },
    {
      id: "id13",
      name: "Valencia C.F.",
      val: 8,
      color: "orange",
    },
  ],
  links: [
    {
      source: "id1",
      target: "id11",
    },
    {
      source: "id1",
      target: "id12",
    },
    {
      source: "id1",
      target: "id13",
    },
  ],
};

const loading = ref(false);

const expandNode = (node) => {
  loading.value = true;
  Promise.resolve(
    setTimeout(() => {
      if (node.childs?.length > 0) {
        node.childs.forEach((child) => {
          data.nodes.push(child);
          data.links.push({
            source: node.id,
            target: child.id,
          });
        });
        myGraph.graphData(data);
      }
      loading.value = false;
    }, 2000)
  );
};

const bloomPass = new UnrealBloomPass();
bloomPass.strength = 2;
bloomPass.radius = 1.2;
bloomPass.threshold = 0.2;

const initGraph = () => {
  myGraph(graphPanel.value)
    .graphData(data)
    .linkOpacity(0.5)
    .nodeThreeObject((node) => {
      const sprite = new SpriteText(node.name);
      sprite.material.depthWrite = false;
      sprite.color = node.color;
      sprite.textHeight = node.val || 10;
      return sprite;
    })
    .linkDirectionalParticles(2)
    .onNodeClick((node) => {
      expandNode(node);
    })
    .postProcessingComposer()
    .addPass(bloomPass);
  /* Hacer fijo el movimiento del nodo
    .onNodeDragEnd((node) => {
      node.fx = node.x;
      node.fy = node.y;
      node.fz = node.z;
    })
    */
};

onMounted(() => {
  initGraph();
});
</script>
