<script>
  import { Router, Route } from "svelte-routing";

  import QuickStart from "./QuickStart.svelte";

  export let url = "";
</script>

<main class="container">
  <Router {url}>
    <div>
      <Route path="/"><QuickStart /></Route>
    </div>
  </Router>
</main>

<style>
</style>
