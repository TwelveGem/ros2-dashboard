<script lang="ts">
  import TopicEditForm from "./TopicEditForm.svelte";
  import "./TopicCard.css";

  let {
    topicName = "/topic",
    messageType = "std_msgs/msg/String",
    index,
    topicValue = null,
    smartFormatTopicValue,
    onTopicUpdate,
    deleteTopic,
  } = $props();

  let showEditForm = $state(false);

  function handleCardClick() {
    showEditForm = true;
  }

  function handleSave(newTopicName: string, newMessageType: string) {
    if (onTopicUpdate) {
      onTopicUpdate(newTopicName, newMessageType);
    }
    showEditForm = false;
  }

  function handleCancel() {
    showEditForm = false;
  }
</script>

<div class="card shadow-lg border-1 mt-3">
  <div
    class="card-body"
    role="button"
    tabindex="0"
    onclick={handleCardClick}
    onkeydown={(e) => {
      if (e.key === "Enter") {
        handleCardClick();
      }
    }}
    style="cursor: pointer;"
  >
    <!-- TOPIC INFO -->
    <div class="row mb-3">
      <div class="card-bg-light">
        <div class="card bg-light">
          <div class="d-flex align-items-center">
            <i class="bi bi-at me-2"></i>
            <code class="text-primary">{topicName}</code>
          </div>
          <div class="d-flex align-items-center">
            <i class="bi bi-type me-2"></i>
            <code class="text-secondary">{messageType}</code>
          </div>
        </div>
      </div>
    </div>
    <div class="card">
      <div class="card-header bg-success text-white">
        <h6 class="mb-0">Topic Data</h6>
      </div>
      <div class="card-body">
        {#if topicValue !== null}
          <code>{smartFormatTopicValue(topicValue, messageType)}</code>
        {:else}
          <div class="text-center text-muted py-4">
            <i class="bi bi-inbox display-4"></i>
            <small>Waiting for message on {topicName}</small>
          </div>
        {/if}
      </div>
    </div>
  </div>

  <button
    class="btn bg-danger close text-white"
    onclick={() => {
      deleteTopic(index);
    }}
  >
    X
  </button>
</div>

<!-- Edit Form Modal -->
<TopicEditForm
  {topicName}
  {messageType}
  isOpen={showEditForm}
  onSave={handleSave}
  onCancel={handleCancel}
/>
