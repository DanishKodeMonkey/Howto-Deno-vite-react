# How to Vite + Deno 2.0 + React + TypeScript in vscode

In order to use Deno along with the create-vite-react template some initial
configurations will have to be made in order for vscode to play nice with things
like the npm imports, node_modules folder, jsx typescript compiling and type
checking.

The process, once figured out, is pretty straight forward. I have gotten this
approach to work on the create-vite-extra template. It may work on other
templates, but I haven't tested this.

## Lets get started.

Before anything else,
[ensure you have deno installed and configured on your system](https://docs.deno.com/runtime/)

Start by creating a directory for the project

```bash
mkdir denovite && cd denovite
```

Inside our new folder directory, initialize the deno environment

```bash
deno init
```

Then, go ahead and use deno to install the create-vite-extra template.

```bash
deno run -A npm:create-vite-extra@latest .
```

This will prompt deno to run the npm command create-vite-extra at the latest
version, in the current folder, denoted by the punctuation '.' with the all
permissions flag '-A',
[this is purely to allow the deno process access to the internet to get the files and install them in the folder.](https://docs.deno.com/runtime/fundamentals/security/)

The process will prompt you if the existing files in the folder should be
deleted, go ahead and do so. They will be replaced with the relevant vite-react
equivalent.

## Congrats, we have a somewhat borked template now!

Before continuing go ahead and
[initialize the IDE workspace](https://docs.deno.com/runtime/getting_started/setup_your_environment/)

For vscode, press `ctrl+shift+p`, search for
`Deno: Initialize Workspace Configuration` and use it. A .vscode folder will be
generated with the relevant files, enabling deno in the workspace.

Now, lets have a look at `./src/main.tsx` and `./src/App.tsx`.

If you find alot of squiggly lines all over the place, it is likely that deno
need some additional configuration to play nice with the node based npm
environment that vite-react-extra is built on.

# The solution

[What brought me on to this solution was a clue give n in the repo for the template](https://github.com/bluwy/create-vite-extra/blob/master/template-deno-react-ts/README.md)

    Papercuts

    Currently there's a "papercut" for Deno users:

    peer dependencies need to be referenced in vite.config.js - in this example it is react and react-dom packages that need to be referenced

looking in `./vite.config.mts` we find the version of react and react-dom that
the template is built on, we need to tell deno about these.

So, in `./deno.json` add a imports section with the references we need, just
like any deno import.

While you are at it, for compatibility reasons, add
`json nodeModulesDir: "auto"` as well to force deno to leverage the node_modules
directory that vite react uses.

```json
{
  "imports": {
    "react": "npm:react@^18.2.0",
    "react-dom": "npm:react-dom@^18.2.0"
  }
}
```

Next, in order for denos typescript to be able to understand react we will have
to specify the default libraries for it to refer to the types of

Furthermore, in order for typescript to not lose it whenever it sees jsx elements, we will also specify how typescript handles jsx syntax.

react-jsx tells TypeScript to use the new JSX transform introduced in React 17. It optimizes imports by automatically including React only where necessary. You don't need to explicitly import React when using JSX.

jsxImportSource specifies the module to import when using JSX in conjunction with the react-jsx runtime. I chose preact for this due to its lightweight alternative to React, and its easy implementation using the esm cdn. And it just worked out of the box.

so, back to `./deno.json` add

```json
{
  "compilerOptions": {
    "lib": ["dom", "dom.iterable", "esnext"],
    "jsx": "react-jsx",
    "jsxImportSource": "https://esm.sh/preact"
  }
}
```

With this our `deno.json` should look something like this:

```json
{
  "tasks": {
    "dev": "deno run -A --node-modules-dir npm:vite",
    "build": "deno run -A --node-modules-dir npm:vite build",
    "preview": "deno run -A --node-modules-dir npm:vite preview",
    "serve": "deno run --allow-net --allow-read https://deno.land/std@0.157.0/http/file_server.ts dist/"
  },
  "imports": {
    "react": "npm:react@^18.2.0",
    "react-dom": "npm:react-dom@^18.2.0"
  },
  "compilerOptions": {
    "lib": ["dom", "dom.iterable", "esnext"],
    "jsx": "react-jsx",
    "jsxImportSource": "https://esm.sh/preact"
  }
}

```

And if we look back to main.tsx and App.tsx all the squigglies pertaining to react elements and jsx should be fixed! (If not maybe try a restart of your ide)

Go ahead and give it a shot with `deno run dev` and check it out on your localhost:5173 adress!

## Conclusion

Now were all set to start playing around with our deno-vite-react project! Surely no other obstructions or frustrations will occur.

note: I realize that there is still a error pertaining to importing the default react.svg logo as reactLogo. I haven't found the workaround to this quirk yet.