<img alt="icon" src=".alixea/icon.svg" width="32">

# Node.js component for Alixea

A generic Node.js component that can be used to run any Node.js app. Runs on Alixea's Node.js base image
([component-bases](https://github.com/alixea-erp/component-bases)). The version tag is the Node.js major of the image.

## Operation

### Development

Installs the dependencies when the component starts, then runs `npm run dev`, which is `node --watch src/index.mjs`
and reloads on every change. This can be changed with the `containerCommands.developmentStart` field in `alixea.yaml`.

The dependencies are installed with the project's package manager: the one `packageManager` in `package.json` names,
or else the one its lockfile belongs to (npm, pnpm, Yarn or Bun).

### Staging and production

Staging and production build the deployment's commit on the deployment itself, with the deployment's environment
variables: the dependencies are installed, then the `build` script runs. The template's build only prints a line, so
put a build step there if the app needs one. A commit is built once; a restart of the component reuses the build. A
failed build leaves the previous version running, and its log is shown on the deployment page, where the commit can
also be built again.

The component then runs `npm start`, which can be changed with the `containerCommands.productionStart` field in
`alixea.yaml`.

The project folder is read-only while the app runs, so it cannot write files next to its code. Keep data in a database
or another component, and use `/tmp` for a file needed only during one request.

### System packages

The image is `node:26-slim` with pnpm, Yarn and Bun. A package with native code may need libraries the image does not
have. Install them with `initScript` in `alixea.yaml`, which runs as root when the component starts, or
`preBuildScript`, which runs only before the build:

```yaml
- name: Node.js
  identifier: node
  package: https://github.com/alixea-erp/component-nodejs#v26
  initScript: |
    apt-get update && apt-get install -y --no-install-recommends libcairo2
```

### Ports

The component serves on port **3000**. To use another one, change the `port` of its `app` host in `alixea.yaml`.
Alixea passes that port to the app as `PORT`, and `HOSTNAME` is `0.0.0.0`. A start command of your own should listen
on `$PORT` too: staging and production count a new version as ready once it accepts connections on that port.

## Links

- [Node.js documentation](https://nodejs.org/docs/latest-v26.x/api/)
