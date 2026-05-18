# Chapter 0 - Hello World
# 第 0 章 - Hello World

Let's start with a basic TypeScript setup and a hello world program.

讓我們從基本的 TypeScript 設定與 hello world 程式開始。

This guide is written in TypeScript (yes, a python version is coming soon)

本指南以 TypeScript 撰寫（沒錯，python 版本即將推出）。

There are many checkpoints between the every file edit in theworkshop steps, 
so even if you aren't super familiar with typescript,
you should be able to keep up and run each example.

在 workshop 步驟中，每次檔案編輯之間都有許多檢查點，
因此即使你對 typescript 不算非常熟悉，
仍然應該能跟上並執行每個範例。

To run this guide, you'll need a relatively recent version of nodejs and npm installed

若要執行本指南，你需要先安裝較新的 nodejs 與 npm 版本。

You can use whatever nodejs version manager you want, [homebrew](https://formulae.brew.sh/formula/node) is fine

你可以使用任何喜歡的 nodejs 版本管理工具，[homebrew](https://formulae.brew.sh/formula/node) 也沒問題。


    brew install node@20

You should see the node version

你應該會看到 node 版本。

    node --version

Copy initial package.json

複製初始的 package.json

    cp ./walkthrough/00-package.json package.json

<details>
<summary>show file / 顯示檔案</summary>

```json
// ./walkthrough/00-package.json
{
    "name": "my-agent",
    "version": "0.1.0",
    "private": true,
    "scripts": {
      "dev": "tsx src/index.ts",
      "build": "tsc"
    },
    "dependencies": {
      "tsx": "^4.15.0",
      "typescript": "^5.0.0"
    },
    "devDependencies": {
      "@types/node": "^20.0.0",
      "@typescript-eslint/eslint-plugin": "^6.0.0",
      "@typescript-eslint/parser": "^6.0.0",
      "eslint": "^8.0.0"
    }
  }
```

</details>

Install dependencies

安裝相依套件

    npm install

Copy tsconfig.json

複製 tsconfig.json

    cp ./walkthrough/00-tsconfig.json tsconfig.json

<details>
<summary>show file / 顯示檔案</summary>

```json
// ./walkthrough/00-tsconfig.json
{
    "compilerOptions": {
      "target": "ES2017",
      "lib": ["esnext"],
      "allowJs": true,
      "skipLibCheck": true,
      "strict": true,
      "noEmit": true,
      "esModuleInterop": true,
      "module": "esnext",
      "moduleResolution": "bundler",
      "resolveJsonModule": true,
      "isolatedModules": true,
      "jsx": "preserve",
      "incremental": true,
      "plugins": [],
      "paths": {
        "@/*": ["./*"]
      }
    },
    "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
    "exclude": ["node_modules", "walkthrough"]
  }
```

</details>

add .gitignore

加入 .gitignore

    cp ./walkthrough/00-.gitignore .gitignore

<details>
<summary>show file / 顯示檔案</summary>

```gitignore
// ./walkthrough/00-.gitignore
baml_client/
node_modules/
```

</details>

Create src folder

建立 src 資料夾

    mkdir -p src

Add a simple hello world index.ts

加入一個簡單的 hello world `index.ts`

    cp ./walkthrough/00-index.ts src/index.ts

<details>
<summary>show file / 顯示檔案</summary>

```ts
// ./walkthrough/00-index.ts
async function hello(): Promise<void> {
    console.log('hello, world!')
}

async function main() {
    await hello()
}

main().catch(console.error)
```

</details>

Run it to verify

執行它以驗證結果

    npx tsx src/index.ts

You should see:

你應該會看到：

    hello, world!

