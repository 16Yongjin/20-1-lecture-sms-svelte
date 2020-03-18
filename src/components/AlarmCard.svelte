<script>
  import { createEventDispatcher } from "svelte";
  import Icon from "fa-svelte";
  import { faTrash } from "@fortawesome/free-solid-svg-icons/faTrash";

  export let alarm;

  const dispatch = createEventDispatcher();
  const onAlarmDelete = () => dispatch("delete", alarm);
</script>

<style>
  :global(.trash) {
    font-size: 1.75rem;
  }
</style>

<div class="column is-4">
  <div class="box">
    <article class="media">
      <div class="media-content">
        <div class="content">
          <p class="title is-4">{alarm.name}</p>
          <p>{alarm.professor} / {alarm.time}</p>
        </div>

      </div>
      <div class="media-right">

        <button class="button is-rounded is-white" on:click={onAlarmDelete}>
          <span class="icon is-medium has-text-danger">
            <Icon class="trash" icon={faTrash} />
          </span>
        </button>
      </div>
    </article>
  </div>
</div>
