<script lang="ts">
  import "@/styles/styles.css"; // HTML renderer styles
  import { setContext } from "svelte";
  import { Carta, Markdown, MarkdownEditor } from "carta-md";
  import { Carta as CartaType } from "carta-md";
  import localStorageStore from "$lib/stores/localStorage";
  import DOMPurify from "isomorphic-dompurify";
  import { markdownTheme } from "$lib/stores/themeStore";
  import NavBar from "$lib/parts/NavBar.svelte";
  import Stats from "$lib/parts/Stats.svelte";
  import { placeholder } from "$lib/functions/placeholder";
  import { onMount } from "svelte";

  // Plugins:

  // Slash
  import "@/styles/slash.css";
  import { slash } from "@cartamd/plugin-slash";
  import { additionalSnippets } from "$lib/functions/slash";
  // Emoji
  import "@/styles/emoji.css";
  import { emoji } from "@cartamd/plugin-emoji";
  // External links forced
  import type { Plugin, UnifiedTransformer } from "carta-md";
  import rehypeExternalLinks from "rehype-external-links";

  const externalLinksTransformer: UnifiedTransformer<"sync"> = {
    execution: "sync",
    type: "rehype",
    transform: ({ processor }) => {
      processor.use(rehypeExternalLinks, {
        target: "_blank",
        rel: ["noopener", "noreferrer"],
      });
    },
  };

  const externalLinks = (): Plugin => ({
    transformers: [externalLinksTransformer],
  });
  // Admonitions
  import plugin from "remark-github-beta-blockquote-admonitions";
  import { defaultConfig } from "$lib/functions/admonitionsConfig";
  import "@/styles/admonitions.css";

  const admonitionsTransformer: UnifiedTransformer<"sync"> = {
    execution: "sync",
    type: "remark",
    transform: ({ processor }) => {
      processor.use(plugin, defaultConfig);
    },
  };

  const admonitions = (): Plugin => ({
    transformers: [admonitionsTransformer],
  });
  // Raw HTML
  import rehypeRaw from "rehype-raw";

  const rawhtml: Plugin = {
    transformers: [
      {
        execution: "sync",
        type: "rehype",
        transform({ processor }) {
          processor.use(rehypeRaw);
        },
      },
    ],
  };
  // Math
  import { math } from "@cartamd/plugin-math";
  import "katex/dist/katex.css";
  // Code
  import { code } from "@cartamd/plugin-code";
  // Subscript + Superscript
  import { subscript } from "carta-plugin-subscript";
  // End Plugins

  let leftWidth = 50;
  let isResizing = false;
  let selectedTab: "write" | "preview" = "write";
  let editorScrollTop = 0;
  let rendererScrollTop = 0;
  let isScrollSyncEnabled = true;
  let carta: CartaType;
  let source = localStorageStore.get("markdown") || "";
  setContext("source", source);
  $: localStorageStore.set("markdown", source);

  $: carta = new Carta({
    rehypeOptions: { allowDangerousHtml: true },
    sanitizer: DOMPurify.sanitize,
    theme: $markdownTheme === "light" ? "light-plus" : "dark-plus",
    extensions: [
      slash({
        snippets: additionalSnippets,
      }),
      emoji(),
      externalLinks(),
      admonitions(),
      math(),
      code({ theme: $markdownTheme === "light" ? "light-plus" : "dark-plus" }),
      rawhtml,
      subscript(),
    ],
    gfmOptions: {
      // remark-gfm that Carta uses convert single tilde to strikethrough, disable that to use single tilde for subscript.
      // see https://stackoverflow.com/a/78076200/7884074
      singleTilde: false,
    },
  });

  function handleEditorScroll(event: Event) {
    if (!isScrollSyncEnabled) return;
    const editor = event.target as HTMLDivElement;
    editorScrollTop = editor.scrollTop;
    rendererScrollTop = editorScrollTop;

    const renderer = document.querySelector(".renderer") as HTMLDivElement;
    if (renderer) {
      renderer.scrollTop = rendererScrollTop;
    }
  }

  function handleRendererScroll(event: Event) {
    if (!isScrollSyncEnabled) return;
    const renderer = event.target as HTMLDivElement;
    rendererScrollTop = renderer.scrollTop;
    editorScrollTop = rendererScrollTop;

    const editor = document.querySelector(".editor .w-full") as HTMLDivElement;
    if (editor) {
      editor.scrollTop = editorScrollTop;
    }
  }

  function handleMouseDown() {
    isResizing = true;
    document.addEventListener("mousemove", handleMouseMove);
    document.addEventListener("mouseup", handleMouseUp);
  }

  function handleMouseMove(event: MouseEvent) {
    if (isResizing) {
      const totalWidth = document.body.clientWidth;
      leftWidth = (event.clientX / totalWidth) * 100;
    }
  }

  function handleMouseUp() {
    isResizing = false;
    document.removeEventListener("mousemove", handleMouseMove);
    document.removeEventListener("mouseup", handleMouseUp);
  }

  let pwa = false;

  function isPWA() {
    if (typeof window !== "undefined") {
      return window.matchMedia("(display-mode: standalone)").matches;
    }
    return false;
  }

  function managePWAStyles() {
    pwa = isPWA();
    const existingStyleTag = document.getElementById("pwa-style");

    if (pwa) {
      if (!existingStyleTag) {
        const style = document.createElement("style");
        style.id = "pwa-style";
        style.innerHTML = `
          @media (max-width: 768px) {
            .carta-toolbar {
              bottom: 18px !important;
              padding-right: 3px !important;
            }
            .stats {
              bottom: 68px !important;
              border-block: 1px solid #333333 !important;
            }
            .carta-icons-menu {
              margin-bottom: 36px !important;
            }
            .sidebar-icon-buttons {
              margin-bottom: 40px !important;
            }
          }
        `;
        document.head.appendChild(style);
      }
    } else if (existingStyleTag) {
      existingStyleTag.remove();
    }
  }

  onMount(() => {
    managePWAStyles();
  });
</script>

<NavBar />

{#key source}
  <Stats />
{/key}

<div class="flex h-[calc(100dvh-72px)] bg-mono-background">
  <div
    class="editor"
    style="width: {leftWidth}%;">
    <div class="flex h-full overflow-hidden">
      <div
        class="line-count hidden p-2 text-gray-600 text-right border-r border-[#252525] w-12 {(
          selectedTab === 'write'
        ) ?
          ''
        : 'hidden'}">
      </div>
      <div
        class="w-full pb-[env(safe-area-inset-bottom)] p-2 border-none outline-none resize-none bg-mono-background font-mono overflow-y-auto"
        on:scroll="{handleEditorScroll}">
        {#key $markdownTheme}
          <MarkdownEditor
            {carta}
            bind:value="{source}"
            bind:selectedTab
            theme="{$markdownTheme}"
            mode="tabs"
            {placeholder} />
        {/key}
      </div>
    </div>
  </div>

  <!-- svelte-ignore a11y-no-static-element-interactions -->
  <div
    class="divider w-1 h-full cursor-ew-resize bg-[#202020]"
    style="left: calc({leftWidth}% - 5px);"
    on:mousedown="{handleMouseDown}">
  </div>

  <div
    class="renderer pb-[15px] p-2 overflow-auto markdown-content"
    style="width: {100 - leftWidth}%"
    on:scroll="{handleRendererScroll}">
    <div class="renderer-toolbar">
      <span class="mr-2 text-type-primary">Sync Scroll</span>
      <label class="inline-flex items-center">
        <input
          type="checkbox"
          bind:checked="{isScrollSyncEnabled}"
          class="hidden" />
        <span
          class="{`relative inline-block w-8 h-5 transition duration-200 ease-in-out ${isScrollSyncEnabled ? 'bg-[#2E6AAC]' : 'bg-[#3a3a3a]'} rounded-full cursor-pointer`}">
          <span
            class="{`absolute top-1 left-1 inline-block w-3 h-3 bg-white rounded-full transition-transform duration-200 ease-in-out ${isScrollSyncEnabled ? 'translate-x-3' : ''}`}"
          ></span>
          <span
            class="{`absolute top-1 left-1 flex items-center justify-center w-3 h-3 transition-transform duration-200 ${isScrollSyncEnabled ? 'translate-x-3' : ''}`}">
            <i
              class="{`fas ${isScrollSyncEnabled ? 'fa-check' : 'fa-times'} ${isScrollSyncEnabled ? 'text-[#2E6AAC]' : 'text-type-footer'} text-[10px]`}"
            ></i>
          </span>
        </span>
      </label>
    </div>
    {#key source + $markdownTheme}
      <Markdown
        {carta}
        value="{source}"
        theme="{$markdownTheme}" />
    {/key}
  </div>
</div>
