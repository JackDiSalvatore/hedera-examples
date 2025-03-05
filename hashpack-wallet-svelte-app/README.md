#  HEM-183: Installing Hashconnect using Svelte

Instructions on how to connect a Sveltekit project with Haspack Wallet using hashconnect module.

First start a new sveltekit project using Typescript

```bash
npm init svlete my-app
cd my-app
npm i
```

Then install the needed dependencies

```bash
npm install hashconnect # hash connect module
npm install buffer # buffer polyfill module used for node global builtins
```

And then you can run the app like this:

Terminal 1
```bash
npm run dev
```

Terminal 2
```bash
ngrok http 3000
```

To view the app navigate to the https link spit out off ngrok in Terminal 2.

### Issues

1. The first issue I ran into was displayed on the node server in Terminal 1.

```
hashconnect doesn't appear to be written in CJS, but also doesn't appear to be a valid ES module (i.e. it doesn't have "type": "module" or an .mjs extension for the entry point). Please contact the package author to fix.
```

Because hashconnect is not currently ES module compliant (although this PR could change that https://github.com/Hashpack/hashconnect/pull/96) it can not be imported into svelte as you normally would.  Instead you have to do it as described here https://github.com/sveltejs/kit/issues/712
like this:

In the browser the error was `ReferenceError: exports is not defined`

```js
import type { HashConnect, HashConnectTypes } from "hashconnect";
import { onMount } from "svelte";

let hashconnect: HashConnect;

onMount(async () => {
    let hc = await import('hashconnect');
    const { HashConnect } = hc;
    hashconnect = new HashConnect();
});
```

2. The second issue I ran into was `ReferenceError: global is not defined`

I used this article to resolve it https://dev.to/richardbray/how-to-fix-the-referenceerror-global-is-not-defined-error-in-sveltekitvite-2i49

and added the following to the `svelte.config.js` file

```js
/** @type {import('@sveltejs/kit').Config} */
const config = {
	// Consult https://github.com/sveltejs/svelte-preprocess
	// for more information about preprocessors
	preprocess: preprocess(),

	kit: {
		adapter: adapter(),

		// Override http methods in the Todo forms
		methodOverride: {
			allowed: ['PATCH', 'DELETE']
		},
        // =======================================
        // ADD EVERYTHING BELOW HERE
        vite: {
            define: {
                global: {},
            }
        }
        // =======================================
	}
};
```

3. Next we get this error. `ReferenceError: Buffer is not defined`

After googling around a bit, I found someone say this "if you need to use node global builtins for browser you need to polyfill them".  So we need to do some polyfilling with VITE.  Instead of figuring that out, I found another workaround suggested online here https://github.com/vitejs/vite/discussions/2785

This uses the `buffer` module to solve our problem.

At the entry point of the app (the very top of `index.svelte`) add the following

```svelte
<script context="module" lang="ts">
	export const prerender = true;
	import { Buffer as BufferPolyfill } from 'buffer'
	declare var Buffer: typeof BufferPolyfill;
	globalThis.Buffer = BufferPolyfill

	console.log('buffer', Buffer.from('foo', 'hex'))
</script>
```


# create-svelte

Everything you need to build a Svelte project, powered by [`create-svelte`](https://github.com/sveltejs/kit/tree/master/packages/create-svelte).

## Creating a project

If you're seeing this, you've probably already done this step. Congrats!

```bash
# create a new project in the current directory
npm init svelte

# create a new project in my-app
npm init svelte my-app
```

## Developing

Once you've created a project and installed dependencies with `npm install` (or `pnpm install` or `yarn`), start a development server:

```bash
npm run dev

# or start the server and open the app in a new browser tab
npm run dev -- --open
```

## Building

To create a production version of your app:

```bash
npm run build
```

You can preview the production build with `npm run preview`.

> To deploy your app, you may need to install an [adapter](https://kit.svelte.dev/docs/adapters) for your target environment.
