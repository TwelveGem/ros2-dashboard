<script lang="ts">
  import ROSLIB from "roslib";
  import Header from "../components/Header.svelte";
  import TopicCard from "../components/TopicCard.svelte";
  import { onDestroy, onMount } from "svelte";
  import Error from "../components/Error.svelte";

  let ros: ROSLIB.Ros;
  let autoConnect = true;

  let columns = $state(0);

  let isConnected = $state(false);
  let errorMessage = $state("");
  let rosUrl = "ws://localhost:9090";

  let topics = $state([
    { name: "/topic0", type: "std_msgs/msg/String" },
    { name: "/topic1", type: "std_msgs/msg/String" },
    { name: "/topic2", type: "std_msgs/msg/String" },
    { name: "/topic3", type: "std_msgs/msg/String" },
    { name: "/topic4", type: "std_msgs/msg/String" },
    { name: "/topic5", type: "std_msgs/msg/String" },
    { name: "/topic6", type: "std_msgs/msg/String" },
    { name: "/topic7", type: "std_msgs/msg/String" },
    { name: "/topic8", type: "std_msgs/msg/String" },
    { name: "/topic9", type: "std_msgs/msg/String" },
  ]);

  let topicStates = $state<ROSLIB.Topic[]>([]);
  let topicValues = $state<any[]>([]);

  function connectToROS() {
    ros = new ROSLIB.Ros({
      url: rosUrl,
    });

    try {
      ros.on("connection", () => {
        isConnected = true;
        errorMessage = "";
        subscribeToTopics();
      });

      ros.on("error", () => {
        isConnected = false;
        errorMessage = "Coudldn't connect to ROS";
      });

      ros.on("close", () => {
        isConnected = false;
        errorMessage = "Conection Closed";
      });
    } catch (error) {
      errorMessage = "Failed to create ROS connection";
    }
  }

  function subscribeToTopics() {
    if (!ros || !isConnected) return;

    try {
      topicStates = [];
      topicValues = [];

      for (let i = 0; i < topics.length; i++) {
        let state = topics[i];

        const topic = new ROSLIB.Topic({
          ros: ros,
          name: state.name,
          messageType: state.type,
        });

        topicStates.push(topic);

        topic.subscribe((message: any) => {
          topicValues[i] = message;
        });
      }
    } catch (error) {}
  }

  function disconnectFromROS() {
    isConnected = false;

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
  }

  function unsubscribeTopic(index: number) {
    if (topicStates[index]) {
      topicStates[index].unsubscribe();
    }
  }

  function subscribeToTopic(index: number) {
    try {
      let state = topics[index];

      const topic = new ROSLIB.Topic({
        ros: ros,
        name: state.name,
        messageType: state.type,
      });

      topicStates[index] = topic;

      topic.subscribe((message: any) => {
        topicValues[index] = message;
      });
    } catch (error) {}
  }

  function updateTopic(
    index: number,
    newTopicName: string,
    newMessageType: string
  ) {
    // Unsubscribe from the current topic
    if (topicStates[index]) {
      topicStates[index].unsubscribe();
    }

    // Update the topic data
    topics[index] = {
      name: newTopicName,
      type: newMessageType,
    };

    // Clear the current value
    topicValues[index] = null;

    // Subscribe to the new topic if connected
    if (isConnected) {
      subscribeToTopic(index);
    }
  }

  onMount(() => {
    if (autoConnect) {
      connectToROS();
    }
  });

  onDestroy(() => {
    disconnectFromROS();
  });

  $effect(() => {
    if (autoConnect && rosUrl) {
      disconnectFromROS();
      connectToROS();
    }
  });

  function onColumnsChange(up: boolean) {
    if (up && columns < 16) {
      columns++;
    } else if (columns > 0) {
      columns--;
    }
  }
</script>

<Header
  {isConnected}
  {connectToROS}
  {disconnectFromROS}
  {columns}
  {onColumnsChange}
/>

{#if errorMessage}
  <Error {errorMessage} />
{/if}

<div
  class="row"
  style={columns > 0
    ? `display:grid; grid-template-columns: repeat(${columns}, 1fr);`
    : ""}
>
  {#each topics as { name, type }, index}
    <div id={name} class="col">
      <TopicCard
        topicName={name}
        messageType={type}
        topicValue={topicValues[index] || null}
        {smartFormatTopicValue}
        onTopicUpdate={(newTopicName: string, newMessageType: string) =>
          updateTopic(index, newTopicName, newMessageType)}
      />
    </div>
  {/each}
</div>
