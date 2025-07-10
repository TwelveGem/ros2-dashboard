<script lang="ts">
  // Props
  let { isConnected = false } = $props();
</script>

<div>
  <!-- BACKGROUND COLOR -->
  <div class="bg-primary">
    <!-- SEPARATE ITEMS -->
    <div class="d-flex justify-content-between px-2 align-items-center">
      <!-- FAVICON AND TITLE -->
      <h5 class="mt-2 text-white">
        <img
          src="./favicon.png"
          alt=""
          style="height: 40px; width: 40px; object-fit: contain;"
          class="me-2"
        />
        ROS Topic Monitor
      </h5>

      <div class="bg-info">
        <div class="d-flex align-items-center">
          <div class="d-flex align-items-center me-3">
            <div
              class="badge rounded-pill {isConnected
                ? 'bg-success'
                : 'bg-danger'} me-2"
            >
              Test
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</div>
