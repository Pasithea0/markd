<script lang="ts">
  import { XIcon } from "lucide-svelte";
  export let isSidebarOpen = false;
  export let closeSidebar;
</script>

<div
  class="fixed inset-0 bg-black bg-opacity-50 z-30 transition-opacity duration-200"
  class:opacity-0={!isSidebarOpen}
  class:pointer-events-none={!isSidebarOpen}
  role="button"
  tabindex="0"
  on:click={closeSidebar}
  on:keydown={(e) => e.key === "Enter" && closeSidebar()}>
</div>

<aside
  class="fixed top-0 left-0 w-64 h-full bg-mono-card shadow-lg z-40 transform transition-transform duration-200"
  class:translate-x-0={isSidebarOpen}
  class:-translate-x-full={!isSidebarOpen}>
  <div class="p-4">
    <div class="flex justify-between mt-1 mb-5">
      <p class="text-2xl font-semibold">Navigation</p>
      <button
        title="Close"
        on:click={closeSidebar}
        class="bg-mono-accentLight1 hover:bg-mono-accentLight2 duration-100 px-1 rounded">
        <XIcon />
      </button>
    </div>
    <div class="flex flex-col gap-3">
      <a
        href="/"
        title="Home"
        class="text-left text-type-primary py-2 px-3 bg-mono-accentLight1 hover:bg-mono-accentLight2 hover:no-underline duration-150 rounded-md"
        on:click={closeSidebar}>Home</a>
      <a
        href="/about"
        title="About"
        class="text-left text-type-primary py-2 px-3 bg-mono-accentLight1 hover:bg-mono-accentLight2 hover:no-underline duration-150 rounded-md"
        on:click={closeSidebar}>About</a>
      <a
        href="/usage"
        title="How To"
        class="text-left text-type-primary py-2 px-3 bg-mono-accentLight1 hover:bg-mono-accentLight2 hover:no-underline duration-150 rounded-md"
        on:click={closeSidebar}>How To</a>
    </div>
  </div>
</aside>
