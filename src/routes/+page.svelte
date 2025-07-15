<script lang="ts">
  // ROS, Svelte state, Svelte component includes
  import ROSLIB from "roslib";

  import { onDestroy, onMount } from "svelte";

  import Header from "../components/Header.svelte";
  import TopicCard from "../components/TopicCard.svelte";
  import Error from "../components/Error.svelte";

  // Backend behavior variables
  let ros: ROSLIB.Ros;
  const autoConnect = true;
  const rosUrl = "ws://localhost:9090";

  // User controlled state
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

  // Backend controlled state
  let isConnected = $state(false);
  let topicStates = $state<ROSLIB.Topic[]>([]);
  let topicValues = $state<any[]>([]);
  let errorMessage = $state("");

  let columns = $state(0);

  // Initializes ROS variable and handles connection states
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

  // Sets state to not connected and destructs ROS variable
  function disconnectFromROS() {
    isConnected = false;

    if (ros) {
      ros.close();
    }
  }

  // Subscribes to everything in `topics` variable
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

  // Subscribes to specific topic in list
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

  // Unsubscribes from topic at index
  function unsubscribeTopic(index: number) {
    if (topicStates[index]) {
      topicStates[index].unsubscribe();
    }
  }

  // Unsubscribes from topic and removes it from all lists
  function deleteTopic(index: number) {
    unsubscribeTopic(index);

    topics.splice(index, 1);
    topicStates.splice(index, 1);
    topicValues.splice(index, 1);
  }

  // Takes index number in and new name/type and sets it
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

  // Formats the data based on what type it is
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

  // Limits the number of columns and updates based on direction
  function onColumnsChange(up: boolean) {
    if (up && columns < 16) {
      columns++;
    } else if (columns > 0) {
      columns--;
    }
  }

  // When page loads, create ROS connection and subscribe
  onMount(() => {
    if (autoConnect) {
      connectToROS();
    }
  });

  // When page is unloading, destroy ROS connection
  onDestroy(() => {
    disconnectFromROS();
  });

  // If autoConnect or rosUrl change, update ROS variable
  $effect(() => {
    if (autoConnect && rosUrl) {
      disconnectFromROS();
      connectToROS();
    }
  });
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
  class="row px-3"
  style={columns > 0
    ? `display:grid; grid-template-columns: repeat(${columns}, 1fr);`
    : ""}
>
  {#each topics as { name, type }, index}
    <div id={name} class="col">
      <TopicCard
        topicName={name}
        messageType={type}
        {index}
        topicValue={topicValues[index] || null}
        {smartFormatTopicValue}
        onTopicUpdate={(newTopicName: string, newMessageType: string) =>
          updateTopic(index, newTopicName, newMessageType)}
        {deleteTopic}
      />
    </div>
  {/each}
</div>
