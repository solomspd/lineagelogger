<script lang="ts">
  import { writable, get, derived } from "svelte/store";
  import { onMount } from "svelte";
  import {
    SvelteFlow,
    Controls,
    Background,
    BackgroundVariant,
    MiniMap,
    useSvelteFlow,
    Position,
    type Node,
    type Edge,
  } from "@xyflow/svelte";
  import MemberNode from "./lib/MemberNode.svelte";
  import MarrigeNode from "./lib/MarrigeNode.svelte";
  import SideBar from "./lib/SideBar.svelte";
  import ButtonEdge from "./lib/buttonEdge.svelte";
  import base64 from "base64-js";
  import { decompressSync, zlibSync } from "fflate";
  import Fa from "svelte-fa";
  import {
    faUpload,
    faDownload,
    faLink,
  } from "@fortawesome/free-solid-svg-icons";
  import { toast } from "@zerodevx/svelte-toast";
  import { unpack, pack } from "msgpackr";
  import dagre from "@dagrejs/dagre";
  import { saveAs } from "file-saver";

  import "@xyflow/svelte/dist/style.css";

  const nodeTypes = {
    member: MemberNode,
    marrige: MarrigeNode,
  };
  const edgeTypes = {
    default: ButtonEdge,
  };

  let nextId = 0;

  const nodes = writable<Node[]>([]);

  const edges = writable<Edge[]>([]);

  const snapGrid = [25, 25];

  const dagreGraph = new dagre.graphlib.Graph();
  dagreGraph.setDefaultEdgeLabel(() => ({}));
  dagreGraph.setGraph({ rankdir: "TB", ranker: "longest-path" });

  const nodeWidth = 172;
  const nodeHeight = 36;

  const getLayoutedGraph = (nodes: Node[], edges: Edge[]) => {
    if (nodes.length === 0 || edges.length === 0) {
      return { nodes, edges };
    }

    nodes.forEach((node) => {
      dagreGraph.setNode(node.id, { width: nodeWidth, height: nodeHeight });
    });

    edges.forEach((edge) => {
      dagreGraph.setEdge(edge.source, edge.target);
    });
    dagre.layout(dagreGraph);
    nodes.forEach((node) => {
      const nodeWithPosition = dagreGraph.node(node.id);
      node.targetPosition = Position.Top;
      node.sourcePosition = Position.Bottom;

      // We are shifting the dagre node position (anchor=center center) to the top left
      // so it matches the React Flow node anchor point (top left).
      node.position = {
        x: nodeWithPosition.x - nodeWidth / 2,
        y: nodeWithPosition.y - nodeHeight / 2,
      };
    });

    return { nodes, edges };
  };

  const updateLayout = () => {
    const { nodes, edges } = getLayoutedGraph($nodes, $edges);
    $nodes = nodes;
    $edges = edges;
    fitView();
  };

  const { screenToFlowPosition, fitView } = useSvelteFlow();
  const onDragOver = (event: DragEvent) => {
    event.preventDefault();

    if (event.dataTransfer) {
      event.dataTransfer.dropEffect = "move";
    }
  };

  const onDrop = (event: DragEvent) => {
    event.preventDefault();

    if (!event.dataTransfer) {
      return null;
    }

    const type = event.dataTransfer.getData("application/svelteflow");

    const position = screenToFlowPosition({
      x: event.clientX,
      y: event.clientY,
    });

    const newNode = {
      id: `${nextId}`,
      type,
      position,
      data: writable<MemberInfo>({
        firstName: "",
        lastName: "",
      }),
      origin: [0.5, 0.0],
    } satisfies Node;

    nextId += 1;

    $nodes.push(newNode);
    $nodes = $nodes;
  };

  const serializeData = () => {
    return {
      nodes: $nodes.map((node) => {
        return { ...node, data: get(node.data) };
      }),
      edges: $edges.map((edge) => {
        return { ...edge, data: get(edge.data) };
      }),
    };
  };

  const deserializeData = (data: any) => {
    $nodes = data.nodes.map((node: any) => {
      return { ...node, data: writable(node.data) };
    });
    $edges = data.edges.map((edge: any) => {
      return { ...edge, data: writable(edge.data) };
    });
    nextId = Math.max(...data.nodes.map((node: any) => parseInt(node.id))) + 1;
  };

  const saveAndDownload = () => {
    const data = serializeData();
    saveAs(
      new Blob([pack(data)], {
        type: "application/msgpack",
      }),
      "family tree.mpk"
    );
  };

  let uploadTrigger: HTMLInputElement;

  const upload = (e) => {
    const file = (e.target as HTMLInputElement).files?.[0];
    if (file) {
      const reader = new FileReader();
      reader.onload = (e) => {
        const data = unpack(e.target?.result);
        deserializeData(data);
      };
      reader.readAsArrayBuffer(file);
    }
  };

  const generateLink = () => {
    const data = serializeData();
    const dataStr = encodeURIComponent(
      base64.fromByteArray(zlibSync(pack(data), { level: 9 }))
    );
    const url = new URL(window.location.href);
    url.searchParams.set("data", dataStr);
    if (url.href.length > 2000) {
      alert(
        "The URL is too long, might cause issues, please save and upload the data instead"
      );
    }
    navigator.clipboard.writeText(url.href);
    toast.pop(0);
    toast.push("Link copied to clipboard");
  };

  onMount(() => {
    const url = new URL(window.location.href);
    const data = url.searchParams.get("data");
    if (data !== null) {
      const decodedData = unpack(
        decompressSync(base64.toByteArray(decodeURIComponent(data)))
      );
      deserializeData(decodedData);
    }
  });
</script>

<div class="main-container">
  <div class="custom-controls-container">
    <div class="buttons-container">
      <button on:click={saveAndDownload}
        ><Fa icon={faDownload} /> Save and Download</button
      >
      <button
        on:click={() => {
          uploadTrigger.click();
        }}><Fa icon={faUpload} /> Upload</button
      >
      <button on:click={generateLink}
        ><Fa icon={faLink} /> Get sharable Link</button
      >
      <button on:click={updateLayout}>Update Layout</button>
      <input type="file" bind:this={uploadTrigger} on:change={upload} hidden />
    </div>
    <div>
      <SideBar />
    </div>
  </div>
  <div class="canvas-container">
    <SvelteFlow
      {nodes}
      {edges}
      {snapGrid}
      {nodeTypes}
      {edgeTypes}
      fitView
      on:dragover={onDragOver}
      on:drop={onDrop}
      on:nodeclick={(event) => console.log("on node click", event.detail.node)}
    >
      <Controls />
      <Background variant={BackgroundVariant.Dots} />
      <MiniMap />
    </SvelteFlow>
  </div>
</div>

<style>
  .main-container {
    height: 100vh;
    display: flex;
    flex-direction: column;
  }

  .canvas-container {
    flex: 1;
  }

  .custom-controls-container {
    position: fixed;
    top: 0;
    z-index: 100;
    width: 100%;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
  button {
    margin: 10px;
    padding: 10px 20px;
    font-size: 16px;
    border: none;
    border-radius: 5px;
    background-color: #007bff;
    color: white;
    cursor: pointer;
    transition: background-color 0.3s ease;
  }

  button:hover {
    background-color: #0056b3;
  }

  button:active {
    background-color: #004085;
  }
</style>
