# Simple fullstack setup workshop

- `npm install --global yarn`
- `yarn init -y`
- `git init`
- `mkdir backend frontend shared`
- Create `.gitignore`

```
node_modules/**/*
*/dist/**/*
```

- `yarn add -D typescript`
- create `tsconfig.main.json`

```
{
  "compilerOptions": {
    "rootDir": ".",
    "baseUrl": ".",
    "target": "ES2020",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "useDefineForClassFields": true,
    "module": "ESNext",
    "skipLibCheck": true,
    "sourceMap": true,
    "declaration": false,
    "importHelpers": true,
    "skipDefaultLibCheck": true,
    "noErrorTruncation": true,
    "strict": true,
    "noImplicitAny": true,
    "jsx": "react-jsx",
    "paths": {
      "@shared/*": ["./shared/*"]
    }
  },
  "include": ["./shared/**/*", "node_modules/@types/*"]
}

```

- `yarn add react react-dom`
- `yarn add -D @types/react @types/react-dom`
- `yarn add -D vite @vitejs/plugin-react`
- In `frontend` create `tsconfig.json`

```
{
  "extends": "../tsconfig.base.json",
  "compilerOptions": {
    "module": "ESNext",
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx"
  },
  "include": ["src", "../node_modules/@types"]
}

```

- In `frontend` create `tsconfig.node.ts`

```
{
  "extends": "../tsconfig.base.json",
  "compilerOptions": {
    "composite": true,
    "skipLibCheck": true,
    "module": "ESNext",
    "moduleResolution": "bundler",
    "allowSyntheticDefaultImports": true
  },
  "include": ["vite.config.ts"]
}

```

- In `frontend` create `vite.config.ts`

```
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
});
```

- `mkdir frontend/src`
- In frontend create `index.html`

```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Vite + React + TS</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

- In `frontend/src` create `App.tsx` and `main.tsx`
- main.tsx:

```
import React from 'react';
import ReactDOM from 'react-dom/client';
import { App } from './App.tsx';

ReactDOM.createRoot(document.getElementById('root') as HTMLElement).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
);
```

- App.tsx:

```
export function App() {
  return <p>Hello!</p>;
}

```

- In `package.json` add

```
"scripts": {
    "start:frontend": "vite ./frontend"
}
```

- `yarn start:frontend`
- `"build:frontend": "cd frontend && tsc && vite build"`
- `yarn build:frontend`
- `yarn add express cors`
- `yarn add -D @types/express @types/node @types/cors`
- `mkdir backend/src`
- In `backend/src` create `app.ts`

```
import * as express from 'express';
import * as cors from 'cors';

const app = express();
app.use(express.json());
app.use(cors());

app.get('/api', (req, res) => {
  res.json({data: 'hello!'});
});

app.listen(4321, () => console.log('App is listening on: ', 4321));

```

- In `backend` create `tsconfig.json`

```
{
  "extends": "../tsconfig.base.json",
  "compilerOptions": {
    "outDir": "./dist",
    "module": "CommonJS",
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020"],
    "skipLibCheck": true
  },
  "include": ["src"],
  "types": ["node", "express"]
}

```

- `yarn add -D nodemon concurrently`
- `    "start:backend": "cd ./backend && concurrently 'tsc --watch' 'nodemon ./dist/backend/src/app.js'"`
- `yarn start:backend`
- `"build:backend": "cd ./backend && tsc && cp ../package.json ./dist && cd ./dist && npm install --production"`
- `yarn build:backend`
- `"start:all": "concurrently 'npm run start:frontend' 'npm run start:backend'"`
- In `shared` create `config.ts`

```
const PORT = 1532;
const API = `http://localhost:${PORT}/api/`;
export { API, PORT };

```

- `yarn add -D vite-tsconfig-paths`
- In `frontend/vite.config.ts`

```
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import tsconfigPaths from 'vite-tsconfig-paths';

export default defineConfig({
  plugins: [react(), tsconfigPaths()],
});

```

- In `frontend/src/App.tsx`

```
import { useEffect, useState } from 'react';
import { API } from '@shared/config';

export function App() {
  const [state, setState] = useState();

  useEffect(() => {
    const controller = new AbortController();
    const signal = controller.signal;

    fetch(API, {
      signal: signal,
    })
      .then((response) => response.json())
      .then((response) => {
        setState(response);
      });
    return () => {
      controller.abort();
    };
  });
  return <p>{JSON.stringify(state, null, 3)}</p>;
}

```

- In `backend/src/app.ts`

```
import * as express from 'express';
import * as cors from 'cors';
import { PORT } from '@shared/config';

const app = express();
app.use(express.json());
app.use(cors());

app.get('/api', (req, res) => {
  res.json({ data: 'hello!' });
});

app.listen(PORT, () => console.log('App is listening on: ', PORT));

```

- Should throw error - need tsc-alias
- `yarn add -D tsc-alias`
- ```
    "start:backend": "cd ./backend && concurrently 'tsc --watch' 'tsc-alias --watch' 'nodemon ./dist/backend/src/app.js'",

    "build:backend": "cd ./backend && tsc && tsc-alias -p tsconfig.json && cp ../package.json ./dist && cd ./dist && npm install --production",
  ```

- `yarn add -D eslint @typescript-eslint/eslint-plugin @typescript-eslint/parser eslint-plugin-react`
- create `eslintrc.json` in base folder, `frontend` and `backend`
- In base:

```
{
  "root": true,
  "extends": ["eslint:recommended", "plugin:@typescript-eslint/recommended"],
  "parser": "@typescript-eslint/parser",
  "parserOptions": {
    "ecmaVersion": "latest",
    "sourceType": "module"
  },
  "plugins": ["@typescript-eslint"]
}

```

- In `frontend`

```
{
  "extends": ["../.eslintrc.json", "plugin:react/recommended"],
  "plugins": ["@typescript-eslint", "react"],
  "rules": {
    "react/react-in-jsx-scope": "off"
  }
}

```

- In `backend`

```
{ "extends": ["../.eslintrc.json"] }
```

- `"lint": "eslint . --ext .ts,.tsx"`
- `yarn lint`
- `yarn add -D prettier`
- create `.prettierrc`

```
{
  "trailingComma": "all",
  "tabWidth": 2,
  "semi": true,
  "singleQuote": true,
  "bracketSpacing": true,
  "endOfLine": "auto",
  "printWidth": 100
}
```

- `"format": "prettier --write ."`
- `yarn format`
- To connect with eslint
- `yarn add -D eslint-plugin-prettier`
- In `eslintrc.json`

```
{
  "root": true,
  "extends": ["eslint:recommended", "plugin:@typescript-eslint/recommended"],
  "parser": "@typescript-eslint/parser",
  "parserOptions": {
    "ecmaVersion": "latest",
    "sourceType": "module"
  },
  "plugins": ["@typescript-eslint", "prettier"]
}

```

- In `frontend/eslintrc.json`

```
{
  "extends": ["../.eslintrc.json", "plugin:react/recommended"],
  "plugins": ["@typescript-eslint", "react", "prettier"],
  "rules": {
    "react/react-in-jsx-scope": "off"
  }
}

```

- `npx mrm@2 lint-staged`
- In `package.json`

```
 "lint-staged": {
    "*.{js,ts,jsx,tsx}": [
      "prettier --write",
      "eslint --fix"
    ]
  }
```

- Commit changes
