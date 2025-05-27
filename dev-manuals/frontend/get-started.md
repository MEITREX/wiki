# Get Started

This document will contain a short overview of the technologies used in this project as well as a short description of how to setup your environment to start coding.
When only interested in the latter, directly jump to the [Setup Section](#setup)

## Technologies used

For this project we use the following technologies:

- [Next.js](#nextjs)
- [Tailwind CSS](#tailwind-css)
- [TypeScript](#typescript)
- [pnpm](#pnpm)

### Next.js

Next.js is a framework that is based on React. It allows you to create full-stack Web applications by extending the latest React features and integrating powerful Rust-based JavaScript tooling for fast builds.

To learn more about Next.js, please read either the [documentation](https://nextjs.org/docs) or take a look at their [interactive learning course](https://nextjs.org/learn/basics/create-nextjs-app).

### Tailwind CSS

Tailwind CSS functions as a CSS framework and makes working with CSS easier and faster than with plain CSS and dialects.
It is fast, flexible, and reliable — with zero-runtime cost.
If you use VSCode please consider installing the [Tailwind CSS IntelliSense Extension](https://marketplace.visualstudio.com/items?itemName=bradlc.vscode-tailwindcss) to avoid triggering warnings or errors where the Tailwind rules aren't recognized.

To learn more about Tailwind CSS or if you want to look up code examples for your coding please look at the [documentation](https://tailwindcss.com/docs/utility-first).

### TypeScript

TypeScript is a strongly typed programming language that builds on JavaScript, giving you better tooling. It yields a better experience during development due to autocompletion & advanced type checking to catch errors before app deployment.

Further reference to how TypeScript works can be found in their [documentation](https://www.typescriptlang.org/docs/).

### `pnpm`

`pnpm` is - relative to `npm` - a faster, more space-efficient package manager with a slightly different CLI.

If you seek more information on `pnpm`, check out their [documentation](https://pnpm.io/motivation).

# Setup

To start working on the frontend part of the project you need to setup the backend part of the project, clone the repository and install all dependencies. To do so, please follow the steps below:

1. Clone the repository

```bash
git clone git@github.com:MEITREX/frontend.git
# or
git clone https://github.com/MEITREX/frontend.git
```

2. Install all dependencies with `pnpm install`
3. Fetch the GraphQL schema file from the gateway with `pnpm update-schema`
4. Run the frontend developer build with `pnpm dev`
    - This runs `pnpm relay` and `next dev` concurrently
    - To get relay to update your GraphQL fragments and queries, you probably need the [VSCode extension](#ide-setup)
5. Open <http://localhost:3005> with your browser

## IDE Setup

We recommend using [VSCode](https://code.visualstudio.com/) as your IDE, because it offers proper support for TypeScript right of the bat.
For all the other IDEs you need to install a plugin to get the same support.

Besides this project being a typical frontend project, hence requiring the default extensions for typical frontend development, you also just need support for working with GraphQL and Tailwind CSS.
Therefore following extensions are recommend:

- [Tailwind CSS IntelliSense](https://marketplace.visualstudio.com/items?itemName=bradlc.vscode-tailwindcss)
- [Relay GraphQL](https://marketplace.visualstudio.com/items?itemName=meta.relay)
- [GraphQL](https://marketplace.visualstudio.com/items?itemName=graphql.vscode-graphql)
- [GraphQL Syntax](https://marketplace.visualstudio.com/items?itemName=graphql.vscode-graphql-syntax)
- [ESLint](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint)
- [Prettier](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)

> Optional, but nice to decrease the pain:

- [Pretty TS Errors](https://marketplace.visualstudio.com/items?itemName=yoavbls.pretty-ts-errors)

> Additionally to ESLint with more rules, but sometimes a bit over the top:

- [Pretty TS Errors](https://marketplace.visualstudio.com/items?itemName=sonarsource.sonarlint-vscode)

Those extensions and some default settings are already recommended and set when opening the `frontend` repository.

