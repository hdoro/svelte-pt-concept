# PortableText renderer for Svelte

I wrote an [article on how I got to this code](https://hdoro.dev/rendering-portable-text-from-scratch) - you should check that out if you want to help build/test/document a better Svelte + PortableText integration :)

## Usage

Copy and paste all of these files into your Svelte project, import the `<PortableText>` component and use it like so:

```svelte
<script>
  import PortableText from '../PortableText/PortableText.svelte'
  // importing your components

  export let article
</script>

<PortableText
  blocks={article.body}
  serializers={{
    types: {
      // block-level components
      callout: Callout,
      // inline-level components
      userInfo: UserInfo,
    },
    marks: {
      absUrl: AbsoluteURL,
      // Overwrite default mark renderers
      strong: CustomStrong,
    },
    blockStyles: {
      normal: CustomParagraph,
      blockquote: Quote,
      // Re-using the same component across multiple styles
      h1: CustomHeading,
      h2: CustomHeading,
      h3: CustomHeading,
      // Custom user-defined style
      textCenter: CentralizedText
    },
  }}
/>
```

Example components from above:

```svelte
<!-- UserInfo (type) -->
<script lang="ts">
  import { session } from '$app/stores'

  export let node: { bold?: boolean } = { bold: false }

  $: userName = $session?.user?.name || 'person'
</script>

{#if node.bold}
  <strong>{userName}</strong>
{:else}
  {userName}
{/if}
```

```svelte
<!-- AbsoluteURL (mark) -->
<script lang="ts">
  export let mark: { url?: string; newWindow?: boolean } = {}

  $: newWindow = mark.newWindow || false
</script>

{#if mark.url}
  <a href={mark.url} target={newWindow ? "_blank" : undefined}><slot /></a>
{:else}
  <slot />
{/if}
```

```svelte
<!-- CustomHeading (blockStyle) -->
<script>
  export let index
  export let blocks
  export let node

  const HEADING_STYLES = ["h1", "h2", "h3", "h4", "h5"]
  $: style = node.style
  $: precededByHeading = HEADING_STYLES.includes(blocks[index - 1]?.style)

  $: anchorId = `heading-${node._key}`
</script>

<!-- If preceded by heading, have a higher margin top -->
<div class="relative {precededByHeading ? "mt-10": "mt-4"}" id={anchorId}>
  <a href="#{anchorId}">
    <span class="sr-only">Link to this heading</span>
    🔗
  </a>
  {#if style === "h1"}
    <h1 class="text-4xl font-black"><slot/></h1>
  {:else if style === "h2"}
    <h2 class="text-3xl"><slot/></h2>
  {:else}
    <h3 class="text-xl text-gray-600"><slot/></h3>
  {/if}
</div>
```

The component above is also an example of how you can access blocks surounding the current one for rule-based design 😉

## Notes:

1. API is still a bit messy & undocumented
    1. I invite you to go through the source-code and change it to your liking
    1. Don't forget to file an issue on how the API could be better, though!
1. Lists aren't yet rendered.
1. I haven't tested this in multiple projects, expect a few issues.
1. I probably won't turn this into a npm package or maintain this project for long.
    1. This is a reference of a possible implementation for Svelte + PortableText that can inspire a final package.
    1. I'll be in touch with other authors to coordinate these efforts for a final library with 0 dependencies and great performance ([Jacob Stordahl](https://github.com/stordahl/portable-text-to-svelte), [Rune](https://github.com/runeh/svelte-pote) and [MovingBrands](https://github.com/movingbrands/svelte-portable-text))
