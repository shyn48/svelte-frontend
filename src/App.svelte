<script>
  import { onMount } from "svelte";

  let averagePrice = "Loading...";
  let isConnected = false;
  let errorMessage = "";

  onMount(() => {
    try {
      const ws = new WebSocket("ws://localhost:8081/average-price");

      ws.onopen = () => {
        isConnected = true;
        console.log("WebSocket connection opened");
      };

      ws.onmessage = (event) => {
        // Split the received data on newline characters
        const messages = event.data.split("\n");
        // Process each message individually
        for (let msg of messages) {
          try {
            const data = JSON.parse(msg);
            if (data.type === "averagePrice") {
              averagePrice = parseFloat(data.data).toFixed(2);
            }
          } catch (error) {
            console.error("Error parsing message:", error, "Message:", msg);
          }
        }
      };

      ws.onerror = (error) => {
        console.error("WebSocket error:", error);
        errorMessage = "WebSocket error occurred. Please try again later.";
      };

      ws.onclose = () => {
        isConnected = false;
        console.log("WebSocket connection closed");
      };
    } catch (error) {
      console.error("WebSocket initialization error:", error);
      errorMessage = "Failed to initialize WebSocket connection.";
    }
  });
</script>

<main>
  <h1>Average Price: ${averagePrice}</h1>
  {#if !isConnected}
    <p class="status">Connecting to server...</p>
  {/if}
  {#if errorMessage}
    <p class="status error">{errorMessage}</p>
  {/if}
</main>

<style>
  main {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    height: 100vh;
    font-family: Arial, sans-serif;
    background: linear-gradient(to right, #141e30, #243b55);
    color: #fff;
    text-align: center;
  }

  h1 {
    font-size: 3rem;
    margin: 0;
  }

  .status {
    margin-top: 1rem;
    font-size: 1rem;
  }

  .error {
    color: #ff4d4d;
  }
</style>
