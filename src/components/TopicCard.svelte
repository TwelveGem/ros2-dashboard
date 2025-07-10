<script lang="ts">
  import { onMount, onDestroy } from "svelte";
  import ROSLIB from "roslib";

  // Props
  let {
    topicName = "/example_topic",
    messageType = "std_msgs/String",
    rosUrl = "ws://localhost:9090",
    autoConnect = true,
  } = $props();

  // Reactive variables
  let topicValue: string | null = $state(null);
  let isConnected: boolean = $state(false);
  let errorMessage: string = $state("");

  // ROS objects
  let ros: ROSLIB.Ros;
  let topic: ROSLIB.Topic;
  let connectionTimer: number | undefined;

  // Connect to ROS
  function connectToROS() {
    try {
      ros = new ROSLIB.Ros({
        url: rosUrl,
      });

      ros.on("connection", () => {
        console.log("Connected to ROS");
        isConnected = true;
        errorMessage = "";
        subscribeToTopic();
      });

      ros.on("error", (error: any) => {
        console.error("ROS connection error:", error);
        isConnected = false;
        errorMessage = "Failed to connect to ROS";
      });

      ros.on("close", () => {
        console.log("Disconnected from ROS");
        isConnected = false;
        errorMessage = "Disconnected from ROS";
      });
    } catch (error) {
      console.error("Error creating ROS connection:", error);
      errorMessage = "Failed to create ROS connection";
    }
  }

  // Subscribe to topic
  function subscribeToTopic() {
    if (!ros || !isConnected) return;

    try {
      topic = new ROSLIB.Topic({
        ros: ros,
        name: topicName,
        messageType: messageType,
      });

      topic.subscribe((message: any) => {
        topicValue = message;
      });

      console.log(`Subscribed to topic: ${topicName}`);
    } catch (error) {
      console.error("Error subscribing to topic:", error);
      errorMessage = "Failed to subscribe to topic";
    }
  }

  // Unsubscribe from topic
  function unsubscribeFromTopic() {
    if (topic) {
      topic.unsubscribe();
      console.log(`Unsubscribed from topic: ${topicName}`);
    }
  }

  // Disconnect from ROS
  function disconnectFromROS() {
    unsubscribeFromTopic();
    if (ros) {
      ros.close();
    }
  }

  // Format the topic value for display
  function formatTopicValue(value: any): string {
    if (value === null || value === undefined) {
      return "No data";
    }

    if (typeof value === "object") {
      return JSON.stringify(value, null, 2);
    }

    return String(value);
  }

  // Lifecycle
  onMount(() => {
    if (autoConnect) {
      connectToROS();
    }
  });

  onDestroy(() => {
    disconnectFromROS();
    if (connectionTimer) {
      clearInterval(connectionTimer);
    }
  });

  // Reactive statement to reconnect when props change
  $effect(() => {
    if (autoConnect && rosUrl) {
      disconnectFromROS();
      connectToROS();
    }
  });

  $effect(() => {
    if (topicName) {
      unsubscribeFromTopic();
      subscribeToTopic();
    }
  });
</script>

<div class="card shadow-lg border-1 mb-3">
  <!-- HEADER -->
  <div
    class="card-header bg-primary text-white d-flex justify-content-between align-items-center"
  >
    <h5 class="mb-0">
      <i class="bi bi-robot me-2"></i>
      ROS Topic Monitor
    </h5>

    <div class="d-flex align-items-center">
      <div class="d-flex align-items-center me-3">
        <div
          class="badge rounded-pill {isConnected
            ? 'bg-success'
            : 'bg-danger'} me-2
        "
        >
          <i class="bi {isConnected ? 'bi-wifi' : 'bi-wifi-off'}"></i>
        </div>
        <small class="text-light">
          {isConnected ? "Connected" : "Disconnected"}
        </small>
      </div>

      <div class="btn-group">
        <button
          class="btn btn-outline-light btn-sm"
          onclick={connectToROS}
          disabled={isConnected}
        >
          <i class="bi bi-play-fill"></i> Connect</button
        >
        <button
          class="btn btn-outline-light btn-sm"
          onclick={disconnectFromROS}
          disabled={!isConnected}
        >
          <i class="bi bi-stop-fill"></i> Disconnect
        </button>
      </div>
    </div>
  </div>

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

<button
  type="button"
  class="btn btn-primary"
  onclick={() => {
    if (topicName === "/example_topic") {
      topicName = "/topic";
    } else {
      topicName = "/example_topic";
    }
  }}
>
  Toggle souce
</button>
