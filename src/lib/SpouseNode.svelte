<script lang="ts">
  import { Handle, Position, type NodeProps, useNodes } from "@xyflow/svelte";
  import type { Writable } from "svelte/store";

  type $$Props = NodeProps;

  export let data: { startDate: Writable<Date>; endDate: Writable<Date> };
  export let id: $$Props["id"];

  const nodes = useNodes();

  const { startDate, endDate } = data;

  const deleteNode = () => {
    console.log("delete node", id);
    console.log("delete node", nodes);
    nodes.update((nodes) => nodes.filter((node) => node.id !== id));
  };
</script>

<div class="spousenode">
  <Handle type="target" id="L" position={Position.Left} />
  <Handle type="target" id="R" position={Position.Right} />

  <div class="column-flex">
    <div class="input-row nodrag">
      <label class="label" for="startDate">Start Date:</label>
      <input
        class="nodrag"
        id="startDate"
        type="date"
        bind:value={$startDate}
      />
    </div>
    <div class="input-row">
      <label class="label" for="endDate">End Date:</label>
      <input class="nodrag" id="endDate" type="date" bind:value={$endDate} />
    </div>
    <div class="input-row">
      <button on:click={deleteNode}>Delete Node</button>
    </div>
  </div>
  <Handle type="source" position={Position.Bottom} />
</div>

<style>
  .label {
    width: 55pt; /* Adjust this value as needed */
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

  .spousenode {
    background-color: #f4f4f4;
    padding: 10px;
    border: 1px solid #111;
    border-radius: 5px;
  }
</style>
