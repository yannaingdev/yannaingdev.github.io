---
title: Service Workers Don’t Break Apps — Misunderstood Control Does
---

There’s a very specific kind of frustration that only shows up when you build your first serious PWA.

Everything works — until you turn the server off.

I had a Next.js app running beautifully online: clean UI, smooth navigation, and proper data loading. I wired up Serwist for PWA support, saw the service worker install, and thought:

> Nice. Offline ready.

Then I stopped the server and refreshed the page.

Immediately, the problems appeared:

- `ERR_CONNECTION_REFUSED`
- Missing CSS
- Broken layout
- The offline page was not showing
- The service worker appeared to be doing nothing

It felt like everything was broken.

It wasn’t.

## The Illusion of “Installed”

The biggest misunderstanding I had was this:

> If the service worker is installed, it is working.

That assumption is incorrect.

A service worker can be installed and activated, yet still not control your page.

If it does not control the page, none of your caching logic matters. There will be no fallbacks, no request interception, and no offline support. The browser simply makes direct network requests, which fail immediately when the server is down.

That was my first real lesson:

> Service workers do nothing unless they control the scope.

## The Silent Killer: Duplicate Precache Entries

The root cause turned out to be small but critical.

I had `/~offline` defined in two places:
- `next.config.mjs` via `additionalPrecacheEntries`
- Manually again in `sw.ts`

That duplication caused the precache step to fail quietly. The worker did not install cleanly, the page was not controlled, and offline navigation bypassed the service worker entirely.

Because this failure happens during installation, it does not always produce obvious errors. It simply fails to work.

That was the first domino.

## When HTML Loads but Everything Looks Broken

After fixing installation, I encountered the next surprise.

Offline pages loaded.

However, there was no styling or layout, only raw HTML rendering.

The reason was simple: I cached the page, but not its dependent assets.

Next.js does not ship fully self-contained HTML. It depends on:

- `/_next/static/chunks/*.js`
- `/_next/static/css/*.css`

Without those files, React cannot boot properly and styles do not apply.

So yes, the page technically loaded offline.

But it was not usable.

This led to the second realization:

> Offline HTML without its JavaScript and CSS dependencies is effectively broken.

Adding runtime caching for scripts and styles resolved the issue immediately. Once those bundles were cached, the app rendered normally offline.

## React Is Not Your Offline Fallback

At one point, I attempted to use a React route (`/offline/page.tsx`) as the offline fallback.

It seemed logical.

It was incorrect.

Service workers operate below React. They do not render components — they return raw `Response` objects.

If React cannot boot, your route does not exist.

The correct structure looks like this:

**Service Worker (network layer)**  
→ Can I serve cached bytes?  
 → Yes: React boots  
 → No: return `offline.html`

**React (UI layer)**  
→ Handles routing and UX only when the app is already running

That separation changed everything.

## The Turning Point: Control, Not Caching

The entire journey ultimately came down to one concept:

**Control.**

Not caching. Not configuration. Not plugins.

Control.

Once I ensured:
- No duplicate precache entries
- Clean installation
- Explicit registration
- Proper scope
- Runtime caching for scripts and styles

Everything stabilized.

Offline navigation worked. CSS loaded correctly. The fallback displayed properly. There were no more `ERR_CONNECTION_REFUSED` errors.

Most importantly:

> The behavior became predictable.

## What This Journey Actually Taught Me

PWAs are not difficult because of the code itself. They are difficult because of mental models.

You are working across multiple layers:

- Browser lifecycle
- Network interception
- Framework rendering
- Asset chunking
- Build-time hashing
- Runtime caching

If those layers are blurred together, debugging feels chaotic.

Once they are clearly separated, the system becomes understandable.

## The Quiet Win

The biggest shift was not that offline finally worked.

It was this:

I stopped guessing.

I could look at a failure and identify it precisely:
- The page was not controlled.
- The asset was not cached.
- The fallback was implemented at the wrong layer.

That was when the system stopped feeling magical and started feeling architectural.

## Final Thought

Service workers do not randomly break applications.

They behave exactly as configured — even when that configuration does not match your assumptions.

Once you understand:
- Control vs installation
- Network fallback vs UI fallback
- Precache vs runtime cache

PWAs stop being mysterious.

They become intentional systems.

And yes, it was a journey.

A slightly painful one.

But absolutely worth it.