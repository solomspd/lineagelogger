<script>
  import { SvelteFlowProvider } from "@xyflow/svelte";

  import Canvas from "./Canvas.svelte";
</script>

<main>
  <SvelteFlowProvider>
    <Canvas />
  </SvelteFlowProvider>
</main>
