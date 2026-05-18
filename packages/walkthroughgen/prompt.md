Walkthroughgen is a tool for creating walkthroughs, tutorials, readmes, and documentation.

Walkthroughgen 是一個用於建立 walkthrough、tutorial、readme 與文件的工具。

## Usage
## 用法

You create a walkthrough by writing a simple yaml file that describes the walkthrough. In the file, you reference the incremental files that should exist at each step of the walkthrough

你可以透過撰寫一個描述 walkthrough 的簡單 yaml 檔案來建立 walkthrough。在該檔案中，你會參照每個 walkthrough 步驟中應存在的漸進式檔案。

```
├── walkthrough
│   ├── 00-package-lock.json
│   ├── 00-package.json
│   ├── 01-index.ts
│   ├── 02-cli.ts
│   └── 02-index.ts
└── walkthrough.yaml
```

Your walkthrough.yaml file might look like this (runnable example in [examples/typescript-cli](./examples/typescript))

你的 `walkthrough.yaml` 檔案可能會長這樣（可執行範例位於 [examples/typescript-cli](./examples/typescript)）。

```yaml
title: "setting up a typescript cli"
text: "this is a walkthrough for setting up a typescript cli"
targets:
  - markdown: "./build/walkthrough.md" # generates a walkthrough.md file
    # 產生 walkthrough.md 檔案
    onChange: # default behavior - on changes, show diffs and cp commands
              # 預設行為：發生變更時顯示 diff 與 cp 指令
      diff: true
      cp: true
    newFiles: # when new files are created, just show the copy command
              # 當建立新檔案時，只顯示複製指令
      cat: false
      cp: true
  - final: "./build/final" # outputs the final project to the final folder
    # 將最終專案輸出到 final 資料夾
  - folders: "./build/by-section" # creates a separate working folder for each section
    # 為每個 section 建立獨立的工作資料夾
sections:
  - name: setup
    title: "Copy initial files"
    steps:
      - file: {src: ./walkthrough/00-package.json, dest: package.json}
      - file: {src: ./walkthrough/00-package-lock.json, dest: package-lock.json}
      - file: {src: ./walkthrough/00-tsconfig.json, dest: tsconfig.json}
  - name: initialize
    title: "Initialize the project"
    steps:
      - text: "initialize the project"
        command: |
          npm install
      - text: "then add index.ts"
        file: {src: ./walkthrough/01-index.ts, dest: src/index.ts}
      - text: "run it with tsx"
        command: |
          npx tsx src/index.ts
        results:
          - text: "you should see a hello world message"
            code: |
              hello world
  - name: add-cli
    title: "Add a CLI"
    steps:
      - text: "add a cli"
        file: {src: ./walkthrough/02-cli.ts, dest: src/cli.ts}
      - text: "add a cli"
        file: {src: ./walkthrough/02-index.ts, dest: src/index.ts}
```

Build the project with:

使用以下指令建置專案：

```
npm i -g wtg
wtg build
```

based on your targets, this would create the following files

根據你的 targets，這會建立以下檔案：

```
├── walkthrough
│   ├── 00-package-lock.json
│   ├── 00-package.json
│   ├── 01-index.ts
│   ├── 02-cli.ts
│   └── 02-index.ts
├── build
│   ├── by-section
│   │   ├── 00-initialize # only contains the files in `init`
│   │   │                # 僅包含 `init` 中的檔案
│   │   │   ├── readme.md # contains steps for this section
│   │   │   │            # 包含此 section 的步驟
│   │   │   ├── package.json
│   │   │   ├── package-lock.json
│   │   │   └── tsconfig.json
│   │   └── 01-add-cli # contains the files up to the START of section 1
│   │       # 包含直到第 1 個 section 開始前的所有檔案
│   │       ├── readme.md # contains steps for this section
│   │       │            # 包含此 section 的步驟
│   │       ├── package.json
│   │       ├── package-lock.json
│   │       ├── tsconfig.json
│   │       └── src
│   │           └── index.ts
│   ├── final
│   │   ├── package.json
│   │   ├── package-lock.json
│   │   ├── tsconfig.json
│   │   └── src
│   │       ├── cli.ts
│   │       └── index.ts
│   └── walkthrough.md
```

and your walkthrough.md file will look like:

而你的 `walkthrough.md` 檔案會長得像這樣：

```markdown
# Setting up a typescript cli

this is a walkthrough for setting up a typescript cli

## Copy initial files

  cp walkthrough/00-package.json package.json
  cp walkthrough/00-package-lock.json package-lock.json
  cp walkthrough/00-tsconfig.json tsconfig.json

## Initialize the project

initialize the project

     npm install

then add index.ts


    cp walkthrough/01-index.ts src/index.ts

and run it with tsx

    npx tsx src/index.ts

you should see a hello world message

    hello world

## Add a CLI

add a cli

    ```
    ```
 
    cp walkthrough/02-cli.ts src/cli.ts

update index.ts to use the cli

    ```diff
      const main = async () => {
      +    return cli();
      };
        
      main();
    ```

    or just:

    cp walkthrough/02-index.ts src/index.ts

```

## Features
## 功能特色

### Targets
### 輸出目標

- `file`: generates a single markdown file
- `folder`: creates a set of folders, one for each section
- `final`: outputs the final project to the current directory

- `file`：產生單一 markdown 檔案
- `folder`：建立一組資料夾，每個 section 對應一個
- `final`：將最終專案輸出到目前目錄

### Init
### 初始化

### Sections
### 區段

### Steps
### 步驟

#### Step 
#### 步驟

## Walkthrough.yaml for walkthroughgen
## Walkthroughgen 的 Walkthrough.yaml

## Implementation Plan
## 實作計畫

- [ ] implement core walkthroughgen CLI - `wtg build` # defaults to walkthrough.yaml in current directory
- Scope 1: generating walkthrough.md
  - [ ] create end-to-end test for a simple walkthrough file, just a single yaml file with no sections
  - [ ] create end-to-end test for a walkthrough file with a single section
  - [ ] test generation of diffs and cp commands
- Scope 2: generating final/ project build
  - [ ] create end-to-end test for a walkthrough file with a final target
- Scope 3: generating by-section project builds with readmes
  - [ ] create end-to-end test for a walkthrough file with a by-section target

- [ ] 實作核心 walkthroughgen CLI - `wtg build` # 預設讀取目前目錄中的 walkthrough.yaml
- 範圍 1：產生 walkthrough.md
  - [ ] 為簡單 walkthrough 檔案建立 end-to-end 測試，只包含單一 yaml 檔且沒有 sections
  - [ ] 為包含單一 section 的 walkthrough 檔案建立 end-to-end 測試
  - [ ] 測試 diff 與 cp 指令的產生
- 範圍 2：產生 final/ 專案建置結果
  - [ ] 為包含 final target 的 walkthrough 檔案建立 end-to-end 測試
- 範圍 3：產生含 readme 的 by-section 專案建置結果
  - [ ] 為包含 by-section target 的 walkthrough 檔案建立 end-to-end 測試
