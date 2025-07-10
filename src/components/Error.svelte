<script lang="ts">
  let { errorMessage = "" } = $props();
</script>

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
