<script lang="ts">
  let {
    topicName = "/topic",
    messageType = "std_msgs/msg/String",
    isConnected = false,
    errorMessage = "",
    topicValue = null,
    formatTopicValue,
    autoConnect = false,
  } = $props();
</script>

<div class="card shadow-lg border-1 mb-3">
  <!-- BODY -->
  <div class="card-body">
    <!-- ERROR MESSAGE -->
    {#if errorMessage}
      <div class="alert alert-danger alert-dismissible fade show">
        <i class="bi bi-exclamation-triangle me-2"></i>
        {errorMessage}
        <button
          type="button"
          class="btn-close"
          aria-label="Close"
          onclick={() => (errorMessage = "")}
        ></button>
      </div>
    {/if}

    <!-- TOPIC INFO -->
    <div class="row mb-3">
      <div class="col">
        <div class="card bg-light">
          <div class="card-body py-2">
            <h6 class="card-title mb-1">
              <i class="bi bi-at me-2"></i>Message Type
            </h6>
            <code class="text-primary">{topicName}</code>
          </div>
        </div>
      </div>
      <div class="col">
        <div class="card bg-light">
          <div class="card-body py-2">
            <h6 class="card-title mb-1">
              <i class="bi bi-type me-2"></i>Message Type
            </h6>
            <code class="text-secondary">{messageType}</code>
          </div>
        </div>
      </div>
    </div>

    <!-- TOPIC DATA -->
    <div class="card">
      <div class="card-header bg-success text-white">
        <h6 class="mb-0">Topic Data</h6>
      </div>
      <div class="card-body">
        {#if topicValue !== null}
          <pre
            class="bg-dark text-light p-3 rounded"
            style="max-height: 300px; overflow-y: auto;">
            <code>{formatTopicValue(topicValue)}</code>
          </pre>
        {:else}
          <div class="text-center text-muted py-4">
            <i class="bi bi-inbox display-4"></i>
            <small>Waiting for messages on {topicName}</small>
          </div>
        {/if}
      </div>
    </div>

    <!-- META DATA -->
    <div class="row mt-3">
      <div class="col">
        <div class="card bg-light">
          <div class="card-body py-2">
            <div class="row text-center">
              <div class="col">
                <div class="border-end">
                  <h6 class="text-muted mb-1">Status</h6>
                  <span
                    class="badge {isConnected ? 'bg-success' : 'bg-danger'}"
                  >
                    {isConnected ? "Active" : "Inactive"}
                  </span>
                </div>
              </div>
              <div class="col">
                <div class="border-end">
                  <h6 class="text-muted mb-1">Messages</h6>
                  <span class="fw-bold">
                    {topicValue !== null ? "Received" : "None"}
                  </span>
                </div>
              </div>
              <div class="col">
                <div>
                  <h6 class="text-muted mb-1">Auto Connect</h6>
                  <span
                    class="badge {autoConnect ? 'bg-info' : 'bg-secondary'}"
                  >
                    {autoConnect ? "Enabled" : "Disabled"}
                  </span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</div>
