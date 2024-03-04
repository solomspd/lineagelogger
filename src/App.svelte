<script>
  import { SvelteFlowProvider } from "@xyflow/svelte";
  import { SvelteToast } from "@zerodevx/svelte-toast";

  import Canvas from "./Canvas.svelte";

  const toastOptions = {
    duration: 3000,
    dismissable: false,
    intro: { x: 0, y: 0 },
  };
</script>

<main>
  <SvelteToast options={toastOptions} />
  <SvelteFlowProvider>
    <Canvas />
  </SvelteFlowProvider>
</main>

<style>
  :root {
    --toastContainerTop: 1rem;
    --toastContainerRight: auto;
    --toastContainerBottom: auto;
    --toastContainerLeft: calc(50vw - 8rem);
    --toastBarHeight: 0;
    --toastWidth: auto;
    --toastPadding: 0 0.5rem 0 0.5rem;
    --toastBorderRadius: 0.5rem;
    --toastBackground: rgba(66, 66, 66, 0.98);
  }
</style>
