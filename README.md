# :warning: THIS IS A PITCH. EVERYTHING IN HERE IS SPECULATION. THE ALL-RC TOOLS AND SOFTWARE ARE NOT YET WRITTEN :warning:

All-RC - Standardized shared configuration file for everyone

## What is All-RC?

The absolute simplest description of All-RC is; "why many config files when one config file good?"

All-RC is a standard for shared configuration files.

Instead of each tool having it's own configuration file, which can lead to a cluttered file tree
filled with config files you're hardly ever going to touch again, All-RC not only builds the bridges
needed for you to put all your configurations in one unified file, but supplies an effective API for
tool authors to provide their users with builtin support for All-RC.

## Why All-RC?

As web developers, we love to use every tool we can to make our lives easier and our code better.
We use linters, formatters, build tools, extended syntaxes, and more. All of these need a
configuration file.

The unfortunate problem is, this can lead to a lot of mess in our projects, as every tool has a
different standard for not just the name of it's file, but also the format - even being different
between the same language, such as only supporting ESM or only supporting CJS.

<ul>
  <li>
    <details>
      <summary>
        Let's look at a project which makes good use of some much-loved tools
      </summary>
      <ul>
        <li>Vite, for build & bundle</li>
        <li>Svelte & SvelteKit, for UI components, routing, and SSR</li>
        <li>TypeScript, for type safety</li>
        <li>Prettier, for formatting</li>
        <li>ESLint, for code quality</li>
        <li>PostCSS, for CSS pre-processing</li>
      </ul>
    </details>
  </li>
  <li>
    <details>
      <summary>
        Before we've even written any code, we have 8 config files
      </summary>
      <ul>
        <li>vite.config.ts</li>
        <li>svelte.config.js</li>
        <li>tsconfig.json</li>
        <li>.prettierrc</li>
        <li>.prettierignore</li>
        <li>.eslintrc.js</li>
        <li>.eslintignore</li>
        <li>postcss.config.js</li>
      </ul>
    </details>
  </li>
  <li>
    <details>
      <summary>
        Even factoring in the other files and folders we'll have for this project, config files account
        for more than half of the nodes in our project root.
      </summary>
      <ul>
        <li>dist</li>
        <li>node_modules</li>
        <li>src</li>
        <li>package.json</li>
        <li>README.md</li>
        <li>.gitignore</li>
        <li>.npmrc</li>
      </ul>
    </details>
  </li>
</ul>

Now let's imagine we've got a monorepo. It has a two or three apps, a few dozen packages, and of
course the workspace root. Depending on the tools you're using, you could be looking at anywhere
from 50 to 150 config files.

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
the rest. One project, one config file.

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

For example, a CLI tool named `foo` would be run with `allrc direct foo [...args]`.

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
command, and change your `--config` argument to be `@allrc.`, followed by the file format of config
to use, such as `@allrc.cjs`, or `@allrc.json`.

`allrc compat <tool-name> --config=@allrc [...args]`

For example, a CLI tool named `foo` which needs a `cjs` config file would be run with `allrc compat foo --config=@allrc.cjs [...args]`.

This will run the tool, with all the command line arguments you've passed, and with the
configuration you've provided in `.allrc`.

> Note: `<tool-name>` here refers to the tool's CLI name. All-RC can determine the tool's NPM
> package name and therefore it's key in `.allrc`.

### Symlink Interoperability

Some tools aren't popular enough for All-RC to provide direct interoperability, and also don't have
a command-line argument for specifying a config file.

For these tools, All-RC can create a symlink to the hidden config file mentioned in
[General Interoperability](#general-interoperability).

To use this, run `allrc link <package-name> <config-file-path>` before running your tool.

`allrc link <package-name> <config-file-path> && <tool-name> [...args]`

For example, a CLI tool named `foo` from the package `@example/foo` would be run with
`allrc link @example/foo ./foo.config.json && foo [...args]`.

This will parse the tool's config from your `.allrc`, write it to a hidden folder in `node_modules`,
and create a symlink to it at the path you've provided.

> Note: `<package-name>` here refers to the key in `.allrc`, which **should** be the name of the
> tool's NPM package. You **should not** pass in the tool's CLI name, as this could be different.

## Supporting All-RC in Your Tool

To support All-RC in your tool, you need to do two things:

1. Install `@all-rc/read-config`
2. Import and call `readConfig` with your tool's NPM package name to retrieve your config

If you want to provide type definitions for you config (and it's _highly_ suggested that you do),
make sure your package exports a type named `AllRC`.

```ts
import { readConfig, MissingKeyError, MissingFileError } from "@all-rc/read-config";

export interface AllRC {
  /* ... */
}

function getMyToolsConfig() {
  try {
    return readConfig<AllRC>("<your-package-name>");
  } catch (e) {
    if (e instanceof MissingFileError) {
      // no `.allrc` file found
    }
    if (e instanceof MissingKeyError) {
      // no config for your tool found
    }
  }
}
```

All-RC advises that tool authors use their tool's NPM package name as the key in `.allrc`.
While it may be possible to use a different key, All-RC reserves the right to break compatibility
with tools which don't adhere to this standard at any time (even in a patch).
