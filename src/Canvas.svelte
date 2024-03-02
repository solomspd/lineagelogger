<script lang="ts">
  import { writable, get } from "svelte/store";
  import { onMount } from "svelte";
  import {
    SvelteFlow,
    Controls,
    Background,
    BackgroundVariant,
    MiniMap,
    useSvelteFlow,
  } from "@xyflow/svelte";
  import MemberNode from "./lib/MemberNode.svelte";
  import SpouseNode from "./lib/SpouseNode.svelte";
  import SideBar from "./lib/SideBar.svelte";
  import ButtonEdge from "./lib/buttonEdge.svelte";
  import base64 from "base64-js";
  import { deflate, inflate } from "pako";

  import "@xyflow/svelte/dist/style.css";

  const nodeTypes = {
    member: MemberNode,
    spouse: SpouseNode,
  };
  const edgeTypes = {
    default: ButtonEdge,
  };

  let nextId = 0;

  const nodes = writable([]);

  const edges = writable([]);

  const snapGrid = [25, 25];

  const { screenToFlowPosition } = useSvelteFlow();
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
    const dataStr =
      "data:text/json;charset=utf-8," +
      encodeURIComponent(JSON.stringify(data));
    console.log(dataStr);
    const downloadAnchorNode = document.createElement("a");
    downloadAnchorNode.setAttribute("href", dataStr);
    downloadAnchorNode.setAttribute("download", "data.json");
    document.body.appendChild(downloadAnchorNode); // required for firefox
    downloadAnchorNode.click();
    downloadAnchorNode.remove();
  };

  const upload = () => {
    const input = document.createElement("input");
    input.type = "file";
    input.onchange = (e) => {
      const file = (e.target as HTMLInputElement).files?.[0];
      if (file) {
        const reader = new FileReader();
        reader.onload = (e) => {
          const data = JSON.parse(e.target?.result as string);
          deserializeData(data);
        };
        reader.readAsText(file);
      }
    };
    input.click();
  };

  const generateLink = () => {
    const data = serializeData();
    const dataStr = encodeURIComponent(
      base64.fromByteArray(deflate(JSON.stringify(data)))
    );
    const url = new URL(window.location.href);
    url.searchParams.set("data", dataStr);
    if (url.href.length > 2000) {
      alert(
        "The URL is too long, might cause issues, please save and upload the data instead"
      );
    }
    navigator.clipboard.writeText(url.href);
  };

  onMount(() => {
    const url = new URL(window.location.href);
    const data = url.searchParams.get("data");
    if (data !== null) {
      const decodedData = JSON.parse(
        inflate(base64.toByteArray(decodeURIComponent(data)), { to: "string" })
      );
      deserializeData(decodedData);
    }
  });
</script>

<!--
👇 By default, the Svelte Flow container has a height of 100%.
This means that the parent container needs a height to render the flow.
-->
<main>
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
  <SideBar />
  <div class="button-container">
    <button on:click={saveAndDownload}>Save and Download</button>
    <button on:click={upload}>Upload</button>
    <button on:click={generateLink}>Get sharable Link</button>
  </div>
</main>

<style>
  main {
    height: calc(100vh - 50px);
    display: flex;
    flex-direction: column-reverse;
  }
  .button-container {
    display: flex;
  }
</style>
