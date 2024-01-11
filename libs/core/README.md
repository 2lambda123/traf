# traf 🚀

Avoid unnecessary builds, tests, lint and more in your monorepo CI. Optimize your development process by intelligently finding affected dependencies and selectively triggering builds and tests only when needed.

## Monorepo support

This lib is designed to work with any monorepo, supported by the following packages:

- nx - [`@traf/nx`](https://github.com/lemonade-hq/traf#trafnx)

If you want to add support for another monorepo tool, please open an issue.

## `@traf/core`

A library that finds affected projects in a monorepo, based on the changed **lines** in the current branch.

### Installation

```bash
npm install @traf/core
```

### **Usage**

```ts
import { trueAffected } from '@traf/core';

const affected = await trueAffected({
  rootTsConfig: 'tsconfig.base.json',
  projects: [
    {
      name: 'proj1',
      sourceRoot: '<project source>',
      tsConfig: '<project source>/tsconfig.json',
    },
    // ...
  ],
})
```

### **Options**

| Option         | Type        | Description                                                  | Default       |
| -------------- | ----------- | ------------------------------------------------------------ | ------------- |
| `rootTsConfig` | `string`    | The path to the root tsconfig file                           |               |
| `projects`     | `Project[]` | An array of projects to check                                |               |
| `cwd`          | `string`    | The current working directory                                |               |
| `base`         | `string`    | The base branch to compare against                           | `origin/main` |
| `includeFiles` | `string[]`  | Glob patterns to include (relative to projects' source root) |               |

> `rootTsConfig` - The path to the root tsconfig file. This file should contain the configuration for the entire monorepo, including the `paths` property that maps all projects, allowing `ts-morph` to find the references.

### **Project**

| Option                 | Type       | Description                                                       |
| ---------------------- | ---------- | ----------------------------------------------------------------- |
| `name`                 | `string`   | The unique name of the project within the monorepo                                                  |
| `sourceRoot`           | `string`   | The root directory of the project's source files                                           |
| `tsConfig`             | `string`   | The path to the project's tsconfig file, which should only include the files relevant to the project|
| `implicitDependencies` | `string[]` | An array of implicit dependencies                                 |

### How it works?

The algorithm is based on the following steps:

1. Using git to find all changed lines in the current branch.
2. Using [ts-morph](https://ts-morph.com/) to find the changed element (function, class, const etc..) per line.
3. Using ts-morph [findReferences](https://ts-morph.com/navigation/finding-references#finding-referencing-nodes) to find all references to the changed element recursively.
4. For each reference, find the project that contains the reference and add it to the affected projects list.
