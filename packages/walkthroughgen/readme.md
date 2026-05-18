# Walkthroughgen
# Walkthroughgen

Walkthroughgen is a tool for creating walkthroughs, tutorials, readmes, and documentation. It helps you maintain step-by-step guides by generating markdown and working directories from a simple YAML configuration.

Walkthroughgen 是一個用於建立 walkthrough、tutorial、readme 與文件的工具。它能根據簡單的 YAML 設定產生 markdown 與工作目錄，幫助你維護逐步教學指南。

## Features
## 功能特色

- 📝 **Markdown Generation**: Create beautiful markdown files with diffs, code blocks, and collapsible sections
- 📁 **Working Directories**: Generate separate directories for each section of your walkthrough
- 🔄 **Incremental Changes**: Track and display changes between steps
- 🎯 **Multiple Targets**: Output to markdown, section folders, and final project state
- 📦 **File Management**: Copy files, create directories, and run commands
- 🔍 **Rich Diffs**: Show meaningful diffs between file versions
- 📚 **Section READMEs**: Generate per-section documentation

- 📝 **Markdown Generation**：建立包含 diff、code block 與可摺疊區段的精美 markdown 檔案
- 📁 **Working Directories**：為 walkthrough 的每個 section 產生獨立目錄
- 🔄 **Incremental Changes**：追蹤並顯示步驟之間的變更
- 🎯 **Multiple Targets**：輸出為 markdown、section 資料夾與最終專案狀態
- 📦 **File Management**：複製檔案、建立目錄並執行指令
- 🔍 **Rich Diffs**：顯示有意義的檔案版本差異
- 📚 **Section READMEs**：為每個 section 產生文件

## Installation
## 安裝

```bash
npm install -g walkthroughgen
```

## Quick Start
## 快速開始

1. Create a `walkthrough.yaml` file:

1. 建立 `walkthrough.yaml` 檔案：

```yaml
title: "My Tutorial"
text: "A step-by-step guide"
targets:
  - markdown: "./walkthrough.md"
    onChange:
      diff: true
      cp: true
  - folders:
      path: "./by-section"
      final:
        dirName: "final"
sections:
  - name: setup
    title: "Initial Setup"
    steps:
      - file: {src: ./files/package.json, dest: package.json}
      - command: "npm install"
```

2. Run the generator:

2. 執行產生器：

```bash
walkthroughgen generate walkthrough.yaml
```

## Directory Structure
## 目錄結構

A typical walkthrough project looks like this:

典型的 walkthrough 專案會長這樣：

```
my-tutorial/
├── walkthrough/          # Source files for each step
│                         # 每個步驟的來源檔案
│   ├── 00-package.json
│   ├── 01-index.ts
│   └── 02-config.ts
├── walkthrough.yaml     # Walkthrough configuration
│                        # Walkthrough 設定
└── build/              # Generated output
                         # 產生的輸出
    ├── by-section/    # Section-by-section working directories
    │                  # 各 section 的工作目錄
    │   ├── 00-setup/
    │   └── 01-config/
    ├── final/         # Final project state
    │                  # 最終專案狀態
    └── walkthrough.md # Generated markdown
                       # 產生的 markdown
```

## Walkthrough.yaml Configuration
## Walkthrough.yaml 設定

### Top-Level Fields
### 最上層欄位

- `title`: Title of the walkthrough
- `text`: Introduction text
- `targets`: Output configuration
- `sections`: Tutorial sections

- `title`：walkthrough 的標題
- `text`：介紹文字
- `targets`：輸出設定
- `sections`：tutorial 的各個 section

### Targets
### 輸出目標

#### Markdown Target
#### Markdown 目標

```yaml
targets:
  - markdown: "./output.md"
    onChange:
      diff: true  # Show diffs for changed files
                  # 顯示已變更檔案的 diff
      cp: true    # Show cp commands
                  # 顯示 cp 指令
    newFiles:
      cat: false  # Don't show file contents
                  # 不顯示檔案內容
      cp: true    # Show cp commands
                  # 顯示 cp 指令
```

#### Folders Target
#### Folders 目標

```yaml
targets:
  - folders:
      path: "./by-section"        # Base path for section folders
                                   # section 資料夾的基底路徑
      skip: ["cleanup"]          # Sections to skip
                                   # 要略過的 sections
      final:
        dirName: "final"        # Name for final state directory
                                 # 最終狀態目錄的名稱
```

### Sections
### 區段

Each section represents a logical step in your tutorial:

每個 section 代表 tutorial 中的一個邏輯步驟：

```yaml
sections:
  - name: setup              # Used for folder naming and skip array
                             # 用於資料夾命名與 skip 陣列
    title: "Initial Setup"   # Display title
                             # 顯示標題
    text: "Setup steps..."   # Section description
                             # section 描述
    steps:
      # ... steps ...
      # ... 步驟 ...
```

### Steps
### 步驟

Steps define the actions to take:

Steps 會定義要執行的動作：

#### File Copy
#### 檔案複製
```yaml
steps:
  - text: "Copy package.json"
    file:
      src: ./files/package.json
      dest: package.json
```

#### Directory Creation
#### 建立目錄
```yaml
steps:
  - text: "Create src directory"
    dir:
      create: true
      path: src
```

#### Command Execution
#### 指令執行
```yaml
steps:
  - text: "Install dependencies"
    command: "npm install"
    incremental: true  # run when building up folders target
                       # 在建立 folders target 的過程中執行
```

#### Command Results
#### 指令結果
```yaml
steps:
  - command: "npm run test"
    results:
      - text: "You should see:"
        code: |
          All tests passed!
```

## Generated Output
## 產生的輸出

### Markdown Features
### Markdown 功能

- **File Diffs**: Shows changes between versions
- **Copy Commands**: Easy-to-follow file copy instructions
- **Collapsible Sections**: Hide/show file contents
- **Code Highlighting**: Syntax highlighting for various languages

- **File Diffs**：顯示版本之間的變更
- **Copy Commands**：容易跟隨的檔案複製指示
- **Collapsible Sections**：可隱藏／顯示檔案內容
- **Code Highlighting**：支援多種語言的語法高亮

Example markdown output:

markdown 輸出範例：

~~~markdown
# Initial Setup

Copy the package.json:

    cp ./files/package.json package.json

<details>
<summary>show file</summary>

```json
{
  "name": "my-project",
  "version": "1.0.0"
}
```
</details>

Install dependencies:

    npm install

You should see:

    added 123 packages
~~~

### Section Folders
### Section 資料夾

The `folders` target creates:

`folders` target 會建立：

1. A directory for each section
2. Section-specific README.md files
3. Working project state
4. Optional final state directory

1. 每個 section 一個目錄
2. section 專用的 README.md 檔案
3. 工作中的專案狀態
4. 可選的最終狀態目錄

## Examples
## 範例

See the [examples](./examples) directory for complete examples:

完整範例請參考 [examples](./examples) 目錄：

- [TypeScript CLI](./examples/typescript): Basic TypeScript project setup
- [Walkthroughgen](./examples/walkthroughgen): Self-documenting example

- [TypeScript CLI](./examples/typescript)：基本的 TypeScript 專案設定
- [Walkthroughgen](./examples/walkthroughgen)：自我說明的範例

## Tips
## 提示

1. Use meaningful section names - they become folder names
2. Include context in step text
3. Use `incremental: true` for commands that modify state
4. Leverage diffs to highlight important changes
5. Use the `skip` array to exclude setup/cleanup sections from output

1. 使用有意義的 section 名稱——它們會成為資料夾名稱
2. 在 step 文字中加入必要的 context
3. 對會修改狀態的指令使用 `incremental: true`
4. 善用 diff 來凸顯重要變更
5. 使用 `skip` 陣列將 setup/cleanup sections 排除在輸出之外

## Contributing
## 貢獻

Contributions welcome! Please read [CONTRIBUTING.md](./CONTRIBUTING.md) for details.

歡迎貢獻！詳細資訊請閱讀 [CONTRIBUTING.md](./CONTRIBUTING.md)。

## License
## 授權

This project is licensed under the MIT License - see the [LICENSE](./LICENSE) file for details.

本專案採用 MIT License 授權——詳細內容請參閱 [LICENSE](./LICENSE) 檔案。
