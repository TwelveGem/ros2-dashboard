<script lang="ts">
  let {
    topicName = "",
    messageType = "",
    isOpen = false,
    onSave,
    onCancel,
  } = $props();

  let editedTopicName = $state(topicName);
  let editedMessageType = $state(messageType);

  function handleSave() {
    onSave(editedTopicName, editedMessageType);
  }

  function handleCancel() {
    onCancel();
  }

  function handleKeydown(event: KeyboardEvent) {
    if (event.key === "Escape") {
      handleCancel();
    } else if (event.key === "Enter" && event.ctrlKey) {
      handleSave();
    }
  }

  $effect(() => {
    if (topicName && messageType) {
      editedTopicName = topicName;
      editedMessageType = messageType;
    }
  });
</script>

{#if isOpen}
  <!-- Modal Backdrop -->
  <div
    class="modal-backdrop show"
    style="position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.5); z-index: 1040;"
  ></div>

  <!-- Modal -->
  <div
    class="modal show d-block"
    style="z-index: 1050;"
    onkeydown={handleKeydown}
    tabindex="-1"
    role="button"
  >
    <div class="modal-dialog modal-dialog-centered">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title">Edit Topic</h5>
          <button
            type="button"
            class="btn-close"
            onclick={handleCancel}
            aria-label="Close"
          ></button>
        </div>

        <div class="modal-body">
          <form>
            <div class="mb-3">
              <label for="topicName" class="form-label">Topic Name</label>
              <input
                type="text"
                class="form-control"
                id="topicName"
                bind:value={editedTopicName}
                placeholder="/topic/name"
                required
              />
              <div class="form-text">
                Enter the ROS topic name (e.g., /cmd_vel, /scan)
              </div>
            </div>

            <div class="mb-3">
              <label for="messageType" class="form-label">Message Type</label>
              <select
                class="form-select"
                id="messageType"
                bind:value={editedMessageType}
                required
              >
                <option value="">Select message type...</option>
                <option value="std_msgs/msg/String">String</option>
                <option value="std_msgs/msg/Int32">Int32</option>
                <option value="std_msgs/msg/Float64">Float64</option>
                <option value="std_msgs/msg/Bool">Bool</option>
                <option value="geometry_msgs/msg/Twist">Twist</option>
                <option value="sensor_msgs/msg/LaserScan">LaserScan</option>
                <option value="nav_msgs/msg/Odometry">Odometry</option>
                <option value="custom">Custom...</option>
              </select>
              <div class="form-text">
                Select the ROS message type for this topic
              </div>
            </div>

            {#if editedMessageType === "custom"}
              <div class="mb-3">
                <label for="customMessageType" class="form-label"
                  >Custom Message Type</label
                >
                <input
                  type="text"
                  class="form-control"
                  id="customMessageType"
                  bind:value={editedMessageType}
                  placeholder="package/msg/MessageType"
                />
                <div class="form-text">
                  Enter custom message type (e.g., my_package/msg/MyMessage)
                </div>
              </div>
            {/if}
          </form>
        </div>

        <div class="modal-footer">
          <button
            type="button"
            class="btn btn-secondary"
            onclick={handleCancel}
          >
            Cancel
          </button>
          <button
            type="button"
            class="btn btn-primary"
            onclick={handleSave}
            disabled={!editedTopicName || !editedMessageType}
          >
            Save Changes
          </button>
        </div>
      </div>
    </div>
  </div>
{/if}
