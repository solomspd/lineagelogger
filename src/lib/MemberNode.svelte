<script lang="ts">
  import { Handle, Position, type NodeProps, useNodes, type Node } from "@xyflow/svelte";
  import type { Writable } from "svelte/store";

  type $$Props = NodeProps;

  export let data: { firstName: Writable<string>; lastName: Writable<string> };
  export let id: $$Props['id'];

  const nodes = useNodes();

  const { firstName, lastName } = data;
  const deleteNode = ()  => {
    console.log("delete node", id);
    console.log("delete node", nodes);
    nodes.update((nodes) => nodes.filter((node) => node.id !== id));
  }
</script>

<div class="membernode">
  <Handle type="source" id="L" position={Position.Left} />
  <Handle type="source" id="R" position={Position.Right} />

  <div class="column-flex">
    <div class="input-row">
      <label class="label" for="firstName">First Name:</label>
      <input
        class="nodrag"
        id="firstName"
        type="text"
        bind:value={$firstName}
      />
    </div>
    <div class="input-row">
      <label class="label" for="lastName">Last Name:</label>
      <input class="nodrag" id="lastName" type="text" bind:value={$lastName} />
    </div>
    <div class="input-row">
      <button on:click={deleteNode}>Delete Node</button>
    </div>
  </div>
  <Handle type="target" position={Position.Top} />
</div>

<style>
  .label {
    width: 60pt; /* Adjust this value as needed */
    display: inline-block;
  }

  .column-flex {
    display: flex;
    flex-direction: column;
  }

  .input-row {
    display: flex;
    flex-direction: row;
  }

  .membernode {
    background-color: #f4f4f4;
    padding: 10px;
    border: 1px solid #111;
    border-radius: 5px;
  }
</style>
