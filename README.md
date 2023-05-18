# :warning: THIS IS A PITCH. EVERYTHING IN HERE IS SPECULATION. THE ALL-RC TOOLS AND SOFTWARE ARE NOT YET WRITTEN :warning:

All-RC - Standardized shared configuration file for everyone

## Why All-RC?

As web developers, we love to use every tool we can to make our lives easier and our code better.
We use linters, formatters, build tools, extended syntaxes, and more. All of these need a
configuration file.

Let's look at a project which makes good use of some much-loved tools:

- vite, for build & bundle
- Svelte & SvelteKit, for UI components, routing, and SSR
- TypeScript, for type safety
- Prettier, for formatting
- ESLint, for code quality
- PostCSS, for CSS processing

And let's look at the config files needed for all of this;

- vite.config.ts
- svelte.config.js
- tsconfig.json
- .prettierrc
- .prettierignore
- .eslintrc.js
- .eslintignore
- postcss.config.js

Even factoring in the other files and folders we'll have for this project, config files account for
more than half of the nodes in our project root.

### Enter All-RC

<table align="right">
  <tr>
    <th>
      <img
          height="200"
          src="https://imgs.xkcd.com/comics/standards_2x.png"
          alt="[Title]: How Standards Proliferate (See AC chargers, character encodings, instant messaging, etc.) [Text]: Situation: there are 14 competing standards. [Person A]: '14?! Ridiculous! We need to develop one universal standard that covers everyone's use cases.' [Person B]: 'yeah!' [Text]: Situation: there are 15 competing standards."
        >
    </th>
  </tr>
  <tr>
    <th>Yes, we've all seen <a href="https://xkcd.com/927/">XKCD's 'Standards'</a></th>
  </tr>
</table>

All-RC stands between our tools and their configuration files. Instead of each tool reading their
own config file, they can ask All-RC for the config they need.

For you, the developer, this means you add all your configs to an `.allrc` file, and All-RC will do
the rest:

- JSON5: `.allrc`
- YAML: `.allrc.yaml`
- TOML: `.allrc.toml`
- JavaScript:
  - Always ESM: `.allrc.mjs`
  - Always CommonJS: `.allrc.cjs`
  - `package.json` type: `.allrc.js`
- TypeScript: `.allrc.ts`

All-RC advises that tool authors use their tool's NPM package name as the key in `.allrc`, this
means when adding configurations for a tool, you should use that tool's NPM package name as the key
in `.allrc`. For example, if you're configuring the `foo` CLI, and it comes from `@example/foo`, you
should use `@example/foo` as the key in `.allrc`.

For tools which haven't yet implemented All-RC support, you can still use All-RC! See the
[support guide](#using-tools-which-dont-support-all-rc) for more details.

## Using Tools Which Don't Support All-RC

All-RC comes with a Command line tool which can fit support for All-RC to almost any tool.

As it's standard for tools to use their NPM package name as the name for their key in `.allrc`,
you should also do so for tools which don't yet support All-RC. This helps All-RC know which
configuration options should be handed to which tools.

### Direct Interoperability

For some popular tools which haven't implemented All-RC support, the `allrc` CLI has a hand-crafted
compatibility layer.

To see if your favorite tool has direct interoperability support, check `allrc direct list`.
If a tool is supported, all you need to do is add `allrc direct` to the start of your command.

`allrc direct <tool-name> [...args]`

<details>
  <summary>
  Tools with Direct Interoperability
  </summary>
    <ul>
      <li>(tool name)</li>
      <li>(tool name)</li>
      <li>(tool name)</li>
    </ul>
</details>

By the way, all the dependencies for our direct interoperability are listed as peer dependencies,
meaning you'll never have to install anything you don't need.

### General Interoperability

For most tools, general compatibility is all that's needed. As long as the tool has a command-line
flag which accepts a config file path, for example `--config=./foo/bar/config`, general
compatibility will work.

The `allrc` CLI can process the configuration for a given tool, write it to a hidden folder in
`node_modules`, and point the tool to the configuration file.

To utilize general compatibility, all you need to do is add `allrc compat` to the start of your
command, and change your `--config` argument to be `@allrc`.

`allrc compat <tool-name> --config=@allrc [...args]`

> Note: `<tool-name>` here refers to the tool's CLI name. All-RC can determine the tool's NPM
> package name and therefore it's key in `.allrc`.
> This is different to `allrc link` (which takes the tool's NPM package name)

### Symlink Interoperability

Some tools aren't popular enough for All-RC to support directly, and also don't have a command-line
argument for a config file.

For these tools, All-RC can create a symlink to the hidden config file mentioned in
[General Interoperability](#general-interoperability).

To use this, run `allrc link <tool-name> <config-file-path>` before running your tool.

> Note: `<tool-name>` here refers to the key in `.allrc`, which **should** be the name of the tool's
> NPM package. You **should not** pass in the tool's CLI name, as this could be different.
> This is different to `allrc compat` (which _does_ take the tool's CLI name)

## Supporting All-RC in Your Tool

To support All-RC in your tool, you need to do two things:

1. Install `@all-rc/read-config`
2. Import and call `readConfig` with your tool's NPM package name to retrieve your config

If you want to provide type definitions for you config (and it's _highly_ suggested that you do),
make sure your package exports a type named `AllRC`.

```ts
import { readConfig } from "@all-rc/read-config";

export interface AllRC {
  /* ... */
}

function myToolsCLI() {
  const config = readConfig<AllRC>("<your-package-name>");
}
```

All-RC advises that tool authors use their tool's NPM package name as the key in `.allrc`.
While it may become possible via a quirk of All-RC's implementation to use a different key, All-RC
reserves the right to break compatibility with tools which don't adhere to this standard.
