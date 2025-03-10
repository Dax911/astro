# @astrojs/markdoc (experimental) 📝

This **[Astro integration][astro-integration]** enables the usage of [Markdoc](https://markdoc.dev/) to create components, pages, and content collection entries.

- <strong>[Why Markdoc?](#why-markdoc)</strong>
- <strong>[Installation](#installation)</strong>
- <strong>[Usage](#usage)</strong>
- <strong>[Configuration](#configuration)</strong>
- <strong>[Examples](#examples)</strong>
- <strong>[Troubleshooting](#troubleshooting)</strong>
- <strong>[Contributing](#contributing)</strong>
- <strong>[Changelog](#changelog)</strong>

## Why Markdoc?

Markdoc allows you to enhance your Markdown with [Astro components][astro-components]. If you have existing content authored in Markdoc, this integration allows you to bring those files to your Astro project using content collections.

## Installation

### Quick Install

The `astro add` command-line tool automates the installation for you. Run one of the following commands in a new terminal window. (If you aren't sure which package manager you're using, run the first command.) Then, follow the prompts, and type "y" in the terminal (meaning "yes") for each one.

```sh
# Using NPM
npx astro add markdoc
# Using Yarn
yarn astro add markdoc
# Using PNPM
pnpm astro add markdoc
```

If you run into any issues, [feel free to report them to us on GitHub](https://github.com/withastro/astro/issues) and try the manual installation steps below.

### Manual Install

First, install the `@astrojs/markdoc` package using your package manager. If you're using npm or aren't sure, run this in the terminal:

```sh
npm install @astrojs/markdoc
```

Then, apply this integration to your `astro.config.*` file using the `integrations` property:

__`astro.config.mjs`__

```js ins={2} "markdoc()"
import { defineConfig } from 'astro/config';
import markdoc from '@astrojs/markdoc';

export default defineConfig({
  // ...
  integrations: [markdoc()],
});
```


### Editor Integration

[VS Code](https://code.visualstudio.com/) supports Markdown by default. However, for Markdoc editor support, you may wish to add the following setting in your VSCode config. This ensures authoring Markdoc files provides a Markdown-like editor experience.

```json title=".vscode/settings.json"
"files.associations": {
    "*.mdoc": "markdown"
}
```

## Usage

Markdoc files can only be used within content collections. Add entries to any content collection using the `.mdoc` extension:

```sh
src/content/docs/
  why-markdoc.mdoc
  quick-start.mdoc
```

Then, query your collection using the [Content Collection APIs](https://docs.astro.build/en/guides/content-collections/#querying-collections):

```astro
---
import { getEntryBySlug } from 'astro:content';

const entry = await getEntryBySlug('docs', 'why-markdoc');
const { Content } = await entry.render();
---

<!--Access frontmatter properties with `data`-->
<h1>{entry.data.title}</h1>
<!--Render Markdoc contents with the Content component-->
<Content />
```

📚 See the [Astro Content Collection docs][astro-content-collections] for more information.

## Configuration

`@astrojs/markdoc` offers configuration options to use all of Markdoc's features and connect UI components to your content.

### Using components

You can add Astro and UI framework components (React, Vue, Svelte, etc.) to your Markdoc using both [Markdoc tags][markdoc-tags] and HTML element [nodes][markdoc-nodes].

#### Render Markdoc tags as Astro components

You may configure [Markdoc tags][markdoc-tags] that map to components. You can configure a new tag from your `astro.config` using the `tags` attribute.


```js
// astro.config.mjs
import { defineConfig } from 'astro/config';
import markdoc from '@astrojs/markdoc';

// https://astro.build/config
export default defineConfig({
  integrations: [
    markdoc({
      tags: {
        aside: {
          render: 'Aside',
          attributes: {
            // Component props as attribute definitions
            // See Markdoc's documentation on defining attributes
            // https://markdoc.dev/docs/attributes#defining-attributes
            type: { type: String },
          }
        },
      },
    }),
  ],
});
```

Then, you can wire this render name (`'Aside'`) to a component from the `components` prop via the `<Content />` component. Note the object key name (`Aside` in this case) should match the render name:


```astro
---
import { getEntryBySlug } from 'astro:content';
import Aside from '../components/Aside.astro';

const entry = await getEntryBySlug('docs', 'why-markdoc');
const { Content } = await entry.render();
---

<Content
  components={{ Aside }}
/>
```

#### Render Markdoc nodes / HTML elements as Astro components

You may also want to map standard HTML elements like headings and paragraphs to components. For this, you can configure a custom [Markdoc node][markdoc-nodes]. This example overrides Markdoc's `heading` node to render a `Heading` component, passing the built-in `level` attribute as a prop:

```js
// astro.config.mjs
import { defineConfig } from 'astro/config';
import markdoc from '@astrojs/markdoc';

// https://astro.build/config
export default defineConfig({
  integrations: [
    markdoc({
      nodes: {
        heading: {
          render: 'Heading',
          // Markdoc requires type defs for each attribute.
          // These should mirror the `Props` type of the component
          // you are rendering. 
          // See Markdoc's documentation on defining attributes
          // https://markdoc.dev/docs/attributes#defining-attributes
          attributes: {
            level: { type: String },
          }
        },
      },
    }),
  ],
});
```

Now, you can map the string passed to render (`'Heading'` in this example) to a component import. This is configured from the `<Content />` component used to render your Markdoc using the `components` prop:

```astro
---
import { getEntryBySlug } from 'astro:content';
import Heading from '../components/Heading.astro';

const entry = await getEntryBySlug('docs', 'why-markdoc');
const { Content } = await entry.render();
---

<Content
  components={{ Heading }}
/>
```

Now, all Markdown headings will render with the `Heading.astro` component. This example uses a level 3 heading, automatically passing `level: 3` as the component prop:

```md
### I'm a level 3 heading!
```

📚 [Find all of Markdoc's built-in nodes and node attributes on their documentation.](https://markdoc.dev/docs/nodes#built-in-nodes)

#### Use client-side UI components

Today, the `components` prop does not support the `client:` directive for hydrating components. To embed client-side components, create a wrapper `.astro` file to import your component and apply a `client:` directive manually.

This example wraps a `Aside.tsx` component with a `ClientAside.astro` wrapper:

```astro
---
// src/components/ClientAside.astro
import Aside from './Aside';
---

<Aside client:load />
```

This component [can be applied via the `components` prop](#render-markdoc-nodes--html-elements-as-astro-components):

```astro
---
// src/pages/why-markdoc.astro
import { getEntryBySlug } from 'astro:content';
import ClientAside from '../components/ClientAside.astro';

const entry = await getEntryBySlug('docs', 'why-markdoc');
const { Content } = await entry.render();
---

<Content
  components={{
    Aside: ClientAside,
  }}
/>
```

### Access frontmatter and content collection information from your templates

You can access content collection information from your Markdoc templates using the `$entry` variable. This includes the entry `slug`, `collection` name, and frontmatter `data` parsed by your content collection schema (if any). This example renders the `title` frontmatter property as a heading:

```md
---
title: Welcome to Markdoc 👋
---

# {% $entry.data.title %}
```

The `$entry` object matches [the `CollectionEntry` type](https://docs.astro.build/en/reference/api-reference/#collection-entry-type), excluding the `.render()` property.

### Markdoc config

The Markdoc integration accepts [all Markdoc configuration options](https://markdoc.dev/docs/config), including [tags](https://markdoc.dev/docs/tags) and [functions](https://markdoc.dev/docs/functions).

You can pass these options from the `markdoc()` integration in your `astro.config`. This example adds a global `getCountryEmoji` function:

```js
// astro.config.mjs
import { defineConfig } from 'astro/config';
import markdoc from '@astrojs/markdoc';

// https://astro.build/config
export default defineConfig({
  integrations: [
    markdoc({
      functions: {
        getCountryEmoji: {
          transform(parameters) {
            const [country] = Object.values(parameters);
            const countryToEmojiMap = {
              japan: '🇯🇵',
              spain: '🇪🇸',
              france: '🇫🇷',
            }
            return countryToEmojiMap[country] ?? '🏳'
          },
        },
      },
    }),
  ],
});
```

Now, you can call this function from any Markdoc content entry:

```md
¡Hola {% getCountryEmoji("spain") %}!
```

:::note
These options will be applied during [the Markdoc "transform" phase](https://markdoc.dev/docs/render#transform). This is run **at build time** (rather than server request time) both for static and SSR Astro projects. If you need to define configuration at runtime (ex. SSR variables), [see the next section](#define-markdoc-configuration-at-runtime).
:::

📚 [See the Markdoc documentation](https://markdoc.dev/docs/functions#creating-a-custom-function) for more on using variables or functions in your content.

### Define Markdoc configuration at runtime

You may need to define Markdoc configuration at the component level, rather than the `astro.config.mjs` level. This is useful when mapping props and SSR parameters to [Markdoc variables](https://markdoc.dev/docs/variables).

Astro recommends running the Markdoc transform step manually. This allows you to define your configuration and call Markdoc's rendering functions in a `.astro` file directly, ignoring any Markdoc config in your `astro.config.mjs`.

You will need to install the `@markdoc/markdoc` package into your project first:

```sh
# Using NPM
npm install @markdoc/markdoc
# Using Yarn
yarn add @markdoc/markdoc
# Using PNPM
pnpm add @markdoc/markdoc
```

Now, you can define Markdoc configuration options using `Markdock.transform()`.

This example defines an `abTestGroup` Markdoc variable based on an SSR param, transforming the raw entry `body`. The result is rendered using the `Renderer` component provided by `@astrojs/markdoc`:

```astro
---
import Markdoc from '@markdoc/markdoc';
import { Renderer } from '@astrojs/markdoc/components';
import { getEntryBySlug } from 'astro:content';

const { body } = await getEntryBySlug('docs', 'with-ab-test');
const ast = Markdoc.parse(body);
const content = Markdoc.transform({
  variables: { abTestGroup: Astro.params.abTestGroup },
}, ast);
---

<Renderer {content} components={{ /* same `components` prop used by the `Content` component */ }} />
```

## Examples

*   The [Astro Markdoc starter template](https://github.com/withastro/astro/tree/latest/examples/with-markdoc) shows how to use Markdoc files in your Astro project.

## Troubleshooting

For help, check out the `#support` channel on [Discord](https://astro.build/chat). Our friendly Support Squad members are here to help!

You can also check our [Astro Integration Documentation][astro-integration] for more on integrations.

## Contributing

This package is maintained by Astro's Core team. You're welcome to submit an issue or PR!

## Changelog

See [CHANGELOG.md](https://github.com/withastro/astro/tree/main/packages/integrations/markdoc/CHANGELOG.md) for a history of changes to this integration.

[astro-integration]: https://docs.astro.build/en/guides/integrations-guide/

[astro-components]: https://docs.astro.build/en/core-concepts/astro-components/

[astro-content-collections]: https://docs.astro.build/en/guides/content-collections/

[markdoc-tags]: https://markdoc.dev/docs/tags

[markdoc-nodes]: https://markdoc.dev/docs/nodes
