<script lang="ts">
  const onDragStart = (event: DragEvent, nodeType: string) => {
    if (!event.dataTransfer) {
      return null;
    }

    event.dataTransfer.setData("application/svelteflow", nodeType);
    event.dataTransfer.effectAllowed = "move";
  };
</script>

<aside>
  <div class="label">You can drag these nodes to the pane below</div>
  <div class="nodes-container">
    <div
      class="input-node node"
      on:dragstart={(event) => onDragStart(event, "member")}
      draggable={true}
    >
      Member
    </div>
    <div
      class="default-node node"
      on:dragstart={(event) => onDragStart(event, "marrige")}
      draggable={true}
    >
      Marrige
    </div>
  </div>
</aside>

<style>
  aside {
    /* background: #a6f5c3; */
    font-size: 1rem;
    display: flex;
    flex-direction: row;
    justify-content: center;
    align-items: center;
  }

  .label {
    margin: 1rem 0;
    font-size: 0.9rem;
  }

  .nodes-container {
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .node {
    margin: 0.5rem;
    border: 1px solid #111;
    padding: 0.5rem 1rem;
    font-weight: 700;
    border-radius: 10px;
    cursor: grab;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.15);
    transition: all 0.3s ease;
    text-align: center;
  }

  .node:hover {
    box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
  }
</style>
