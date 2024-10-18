# Learning Notes

## Svelte & SvelteKit

## Routing
- Routes are defined using directory structure: `routes/page-name`
- `+page.svelte` defines the page component
- `+page.ts` contains JS/TS code for data fetching
  - Contains default exported `load` function available to use on the Svelte page with type safety
- Pages can have server-side actions to update with zero client-side JavaScript (more below)
- Layouts (`+layout.svelte`) are useful for developing headers & footers
- SvelteKit tooling makes route creation easier
- `lib` folder is used for shared components

## Page Files
### page.ts vs page.server.ts
- `page.ts`: Normal client JavaScript code
- `page.server.ts`: 
  - Code runs only on server (useful for database access which shouldn't happen client-side)
  - Contains server-side actions (API call functions)
  - Controls prerender & SSR settings

## Svelte vs React

- Reactive declarations (Starts with $:) makes reactive state management easier compared to React (which uses useState, useEffect). Its less in code and also we don't need to manage dependency array.
- When it comes to computations, useEffect runs everytime, so we have to use useMemo to cache it but Svelte does this automatically and only runs when other variable changes.
- We can't send Components as props in Svelte unlike React, we have to use slot for the same
- In React we put HTML in JS but in Svelte we put JS in HTML. So it provides better way to write conditional, loops etc.
- Svelte provides inbuilt store management for shared state with less boilerplate
