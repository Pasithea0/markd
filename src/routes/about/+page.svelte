<script>
  import NavBar from "$lib/parts/NavBar.svelte";
  import IconButton from "$lib/components/IconButton.svelte";
</script>

<div class="sticky top-0">
  <NavBar />
</div>

<div class="flex pt-8 mx-2 bg-mono-background justify-center items-center">
  <div class="p-5 bg-mono-accent h-1/3 max-w-2xl rounded-md">
    <p class="text-3xl font-semibold">About Us</p>
    <p class="text-type-dimmed mt-2">
      MarkD is an advanced live markdown editor directly in your browser. It is built using
      SvelteKit and supports HTML as well as an extended Markdown syntax, all in the same file.
      MarkD is an open source project and a very opinionated one at that. All of the styles are
      inline tailwind css classes, and the Markdown editor uses Carta. All hosted on Cloudflare
      Pages!
    </p>
    <div class="flex justify-center my-3">
      <a
        href="https://github.com/itzcozi/markd"
        title="Source code"
        class="text-sm">
        <img
          src="https://github-readme-stats.vercel.app/api/pin/?username=itzcozi&repo=markd&theme=dark"
          alt="Github repository" />
      </a>
    </div>
    <div class="flex flex-row gap-3 items-center my-2">
      <IconButton
        title="Email"
        onClick="{() => window.open('mailto:dev@wyzie,ru')}"
        className="w-full py-2 rounded-md">
        Email Us
      </IconButton>
    </div>
    <p class="text-xs mt-4">
      Created by <a
        href="https://github.com/itzcozi"
        title="BadDeveloper's github">BadDeveloper</a>
      and
      <a
        href="https://github.com/Pasithea0"
        title="Pas's github">Pasithea</a> with 💙
    </p>
  </div>
</div>
