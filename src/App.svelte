<script>
  import { onMount } from "svelte";
  import * as Ably from "ably";

  let messages = [];
  let presenceData = [];
  let channel = null;

  onMount(() => {
    const ably = new Ably.Realtime.Promise({
      key: "your-api-key-here",
      clientId: "someid",
    });

    channel = ably.channels.get("your-channel-name");

    channel.subscribe((message) => {
      messages = [...messages, message];
    });

    const updatePresence = async () => {
      presenceData = await channel.presence.get();
    };

    (async () => {
      channel.presence.subscribe("enter", updatePresence);
      channel.presence.subscribe("leave", updatePresence);
      channel.presence.subscribe("update", updatePresence);

      await channel.presence.enter("");
      presenceData = await channel.presence.get();
    })();

    return () => {
      channel.presence.leave();
      channel.presence.unsubscribe("enter");
      channel.presence.unsubscribe("leave");
      channel.presence.unsubscribe("update");
      presenceData = [];

      channel.unsubscribe(channel);
      channel.detach();
    };
  });

  const sendMessage = () => {
    channel.publish("test-message", { text: "message text" });
  };

  const update = (text) => {
    channel.presence.update(text);
  };
</script>

<div class="App">
  <h1>Ably Svelte Demo</h1>
  <div>
    <button on:click={sendMessage}>Send Message</button>
    <button on:click={() => update("hello")}>Update status to hello</button>
  </div>

  <h2>Messages</h2>
  <ul>
    {#each messages as msg}
      <li>{msg.data.text}</li>
    {/each}
  </ul>

  <h2>Present Clients</h2>
  <ul>
    {#each presenceData as msg}
      <li>{msg.clientId}: {msg.data}</li>
    {/each}
  </ul>
</div>
