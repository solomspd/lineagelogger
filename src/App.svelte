<script>
  import { SvelteFlowProvider } from "@xyflow/svelte";
  import { SvelteToast } from "@zerodevx/svelte-toast";
  import Fa from "svelte-fa";
  import { faHeart } from "@fortawesome/free-solid-svg-icons";
  import { faGithub } from "@fortawesome/free-brands-svg-icons";

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
  <footer>
    <div class="attribution">
      <p class="attribution-paragraph">Made with&nbsp;</p>
      <Fa style="color: red" icon={faHeart} />&nbsp;
      <p class="attribution-paragraph">
        by <a href="https://solom.dev">Solom</a>
        <a href="https://github.com/solomspd/lineagelogger"
          ><Fa icon={faGithub} /></a
        >
      </p>
    </div>
  </footer>
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

  footer {
    position: fixed;
    bottom: 0;
    width: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .attribution {
    background: white;
    display: flex;
    flex-direction: row;
    justify-content: center;
    align-items: center;
    padding: 0.2rem;
    border-radius: 1rem;
    margin-bottom: 0.5rem;
  }

  .attribution-paragraph {
    margin: 0;
  }
</style>
