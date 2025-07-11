<script lang="ts">
  import ROSLIB from "roslib";
  import Header from "../components/Header.svelte";
  import TopicCard from "../components/TopicCard.svelte";
  import { onDestroy, onMount } from "svelte";
  import Error from "../components/Error.svelte";

  let topicName = $state("/topic");
  let messageType = "std_msgs/msg/String";
  let rosUrl = "ws://localhost:9090";
  let autoConnect = true;
  let isConnected = $state(false);

  let topicValue: string | null = $state(null);
  let errorMessage = $state("");

  let ros: ROSLIB.Ros;
  let topic: ROSLIB.Topic;
  let connectionTimer: number | undefined;

  function connectToROS() {
    try {
      ros = new ROSLIB.Ros({
        url: rosUrl,
      });

      ros.on("connection", () => {
        isConnected = true;
        errorMessage = "";
        subscribeToTopic();
      });

      ros.on("error", (error: any) => {
        isConnected = false;
        errorMessage = "Failed to connect to ROS";
      });

      ros.on("close", () => {
        isConnected = false;
        topicValue = null;
        errorMessage = "Disconnected from ROS";
      });
    } catch (error) {
      errorMessage = "Failed to create ROS connection";
    }
  }

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
    } catch (error) {
      errorMessage = "Failed to subscribe to topic";
    }
  }

  function unsubscribeFromTopic() {
    if (topic) {
      topic.unsubscribe();
    }
  }

  function disconnectFromROS() {
    unsubscribeFromTopic();
    if (ros) {
      ros.close();
    }
  }

  function smartFormatTopicValue(value: any, type: string) {
    switch (type.toLowerCase()) {
      case "std_msgs/string":
      case "std_msgs/msg/string":
        return value.data;
      default:
        return formatTopicValue(value);
    }

    function formatTopicValue(value: any) {
      if (value === null || value === undefined) {
        return "No data";
      }

      if (typeof value === "object") {
        return "\n" + JSON.stringify(value, null, 0);
      }

      return String(value);
    }

  function smartFormatTopicValue(value: any, messageType: string) {
    switch (messageType.toLowerCase()) {
      case "std_msgs/msg/string":
      case "std_msgs/string": {
        return value.data;
      }
      default:
        return formatTopicValue(value);
    }
  }

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

  $effect(() => {
    if (autoConnect && rosUrl) {
      disconnectFromROS();
      connectToROS();
    }
  });

  $effect(() => {
    if (topicName && messageType) {
      unsubscribeFromTopic();
      subscribeToTopic();
    }
  });
</script>

<Header {isConnected} {connectToROS} {disconnectFromROS} />

{#if errorMessage}
  <Error {errorMessage} />
{/if}

<div class="row">
  <span class="col">
    <TopicCard {topicName} {messageType} {topicValue} {smartFormatTopicValue} />
  </span>
  <span class="col">
    <TopicCard {topicName} {messageType} {topicValue} {smartFormatTopicValue} />
  </span>
  <span class="col">
    <TopicCard {topicName} {messageType} {topicValue} {smartFormatTopicValue} />
  </span>
  <span class="col">
    <TopicCard {topicName} {messageType} {topicValue} {smartFormatTopicValue} />
  </span>
  <span class="col">
    <TopicCard {topicName} {messageType} {topicValue} {smartFormatTopicValue} />
  </span>
  <span class="col">
    <TopicCard {topicName} {messageType} {topicValue} {smartFormatTopicValue} />
  </span>
  <span class="col">
    <TopicCard {topicName} {messageType} {topicValue} {smartFormatTopicValue} />
  </span>
  <span class="col">
    <TopicCard {topicName} {messageType} {topicValue} {smartFormatTopicValue} />
  </span>
  <span class="col">
    <TopicCard {topicName} {messageType} {topicValue} {smartFormatTopicValue} />
  </span>
  <span class="col">
    <TopicCard {topicName} {messageType} {topicValue} {smartFormatTopicValue} />
  </span>
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
