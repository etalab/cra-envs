<p align="center">
    <img src="https://user-images.githubusercontent.com/6702424/111203919-de538880-85c5-11eb-9b99-3fae5c5b7580.png">  
</p>
<p align="center">
    <i>Safely bundle server's environment variable into react apps</i>
    <br>
    <br>
    <img src="https://github.com/garronej/react-envs/workflows/ci/badge.svg?branch=main">
    <img src="https://img.shields.io/bundlephobia/minzip/react-envs">
    <img src="https://img.shields.io/npm/dw/react-envs">
    <img src="https://img.shields.io/npm/l/react-envs">
</p>

# Motivation

Create react app provides no official way to inject environment variable from the server into the page.  
When you run `yarn build` create react app does bundle all the variables prefixed by `REACT_APP_`
and expose them under `process.env` ([see here](https://create-react-app.dev/docs/adding-custom-environment-variables/)).  
The problem, however, is that you likely don't want to build your app on the server.  
The CRA team also suggests to [introduce placeholders](https://create-react-app.dev/docs/title-and-meta-tags/#injecting-data-from-the-server-into-the-page) in the `public/index.html` 
and do the substitution on the server before serving the app. This solution involves a lot of hard to maintain scripting.

This module abstract away the burden of managing environment variable injection as well as providing a type-safe way
to retrieve them in your code.

# Step by step guide

Start by installing the tool: 

```bash
yarn add react-envs 
```

Then declare all the allowed environment variables into the `.env` file of your project

Example
```ini
REACT_APP_FOO="Default value of foo"
REACT_APP_BAR=
REACT_APP_BAZ=
REACT_APP_FIZZ=
```

Once it's done run the script `npx generate-typed-env-getter` ( Use `npx generate-typed-env-getter js` if you your project don't use TypeScript)

It will generate `src/env.ts` ( or `src/env.js`) looking like:
```typescript
/* 
 * Automatically generated by react-envs.
 * If you wish to declare a new environment variable add it in the .env file
 * then run 'npx generate-typed-env-getter'. This file will be updated.
 */
import { getEnvVarName } from "react-envs";

export function getEnv() {
    return {
        "FOO": getEnvVarName("FOO"),
        "BAR": getEnvVarName("BAR"),
        "BAZ": getEnvVarName("BAZ"),
        "FIZZ": getEnvVarName("FIZZ")
    } as const;
}
```

Now let's test it by creating a `.env.local` file like:  
```ini
REACT_APP_BAR="Value of bar defined in .env.local"
```

And let's do this somewhere in our code: 

```typescript
import { getEnv } from "./path/to/env.ts"

console.log(getEnv());
```
Now if we run `REACT_APP_BAZ="Value of baz passed inline" yarn start` we get this
in the console: 

```json
{
    "FOO": "Default value of foo",
    "BAR": "Value of bar defined in .env.local",
    "BAZ": "Value of baz passed inline",
    "FIZZ": ""
}
```

Now if you run `yarn build` then `BAZ="Value of baz on the server" npx embed-environnement-variables`
the value of `BAZ` will be injected in `build/index.html` so that if you start statically serving
the `build/` dir, for example with `npx serve` you will get this in the console:  

```json
{
    "FOO": "Default value of foo",
    "BAR": "Value of baz on the server",
    "BAZ": "",
    "FIZZ": ""
}
```

Note that on the server the environment variable names don't need to be prefixed with `REACT_APP_` (they can though).

# Setup

`package.json`
```json
"scripts": {
    "postinstall": "generate-typed-env-getter",
    "prestart": "generate-typed-env-getter",
    "pretest": "generate-typed-env-getter"
}
```
NOTE: Those scripts are optional, as long as you remember to rerun `npx generate-typed-env-getter`
each time you update `.env` you are good.

`.gitignore`
```ini
/src/env.ts
```

`Dockerfile`
```dockerfile
# build environment
ENTRYPOINT sh -c "npx embed-environment-variables && mv build /usr/share/nginx/html && nginx -g 'daemon off;'"
```

See [complete `Dockerfile`](TODO) example. TODO: Link example


