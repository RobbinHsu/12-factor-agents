# Workshop 2025-07-16: Python/Jupyter Notebook Implementation
# Workshop 2025-07-16：Python/Jupyter Notebook 實作

• **Main Tool**: `walkthroughgen_py.py` - Converts TypeScript walkthrough to Jupyter notebooks
• **Config**: `walkthrough.yaml` - Defines notebook structure and content
• **Output**: `workshop_final.ipynb` - Generated notebook with Chapters 0-7
• **Testing**: `test_notebook_colab_sim.sh` - Simulates Google Colab environment

• **主工具**：`walkthroughgen_py.py` - 將 TypeScript walkthrough 轉換為 Jupyter notebooks
• **設定**：`walkthrough.yaml` - 定義 notebook 的結構與內容
• **輸出**：`workshop_final.ipynb` - 產生包含 Chapter 0-7 的 notebook
• **測試**：`test_notebook_colab_sim.sh` - 模擬 Google Colab 環境

## Key Implementation Learnings
## 重要實作心得

• **No async/await in notebooks** - All BAML calls must be synchronous, remove all async patterns
• **No sys.argv** - Main functions accept parameters directly: `main("hello")` not command line args
• **Global namespace** - Functions defined in cells persist globally, no module imports between cells
• **BAML setup is optional** - Use `baml_setup: true` step only when introducing BAML (Chapter 1+)
• **get_baml_client() pattern** - Required workaround for Google Colab import cache issues
• **BAML files from GitHub** - Fetch with curl since Colab can't display local BAML files
• **Regenerate BAML** - Use `regenerate_baml: true` in run_main when BAML files change
• **Import removal** - Remove `from baml_client import get_baml_client` imports from Python files
• **IN_COLAB detection** - Use try/except on google.colab import to detect environment
• **Human input handling** - get_human_input() uses real input() in Colab, auto-responses locally

• **notebook 中不可使用 async/await** - 所有 BAML calls 都必須是同步的，移除所有 async patterns
• **不可使用 sys.argv** - main 函式直接接受參數：`main("hello")`，不是命令列參數
• **全域命名空間** - 在 cells 中定義的函式會全域持續存在，不需要在 cells 之間做 module imports
• **BAML setup 是可選的** - 只有在引入 BAML（Chapter 1 之後）時才使用 `baml_setup: true` 步驟
• **get_baml_client() pattern** - 這是處理 Google Colab import cache 問題所需的 workaround
• **BAML files 來自 GitHub** - 由於 Colab 無法顯示本機 BAML files，因此要用 curl 抓取
• **重新產生 BAML** - 當 BAML files 變更時，在 run_main 中使用 `regenerate_baml: true`
• **移除 imports** - 從 Python files 中移除 `from baml_client import get_baml_client` imports
• **IN_COLAB 偵測** - 使用 `try/except` 搭配 `google.colab` import 來偵測環境
• **Human input 處理** - `get_human_input()` 在 Colab 中使用真實的 `input()`，在本機則自動回覆

## Implementation Patterns
## 實作模式

• **walkthroughgen_py.py enhancements** - Added kwargs support for run_main steps
• **Test simulation** - test_notebook_colab_sim.sh creates clean venv with all dependencies
• **Debug artifacts** - Test runs preserved in ./tmp/test_TIMESTAMP/ directories
• **BAML test support** - baml-cli test works fine in notebooks, contrary to initial assumption
• **Tool execution** - All calculator operations (add/subtract/multiply/divide) in agent loop
• **Clarification flow** - ClarificationRequest tool for handling ambiguous inputs
• **Serialization formats** - JSON vs XML for thread history (XML more token-efficient)
• **Progressive complexity** - Start with hello world, gradually add BAML, tools, loops, tests

• **walkthroughgen_py.py 強化** - 為 run_main 步驟新增 kwargs 支援
• **測試模擬** - `test_notebook_colab_sim.sh` 會建立含所有 dependencies 的乾淨 venv
• **除錯產物** - 測試執行結果會保留在 `./tmp/test_TIMESTAMP/` 目錄中
• **BAML test 支援** - `baml-cli test` 在 notebooks 中運作正常，與最初假設相反
• **Tool 執行** - 所有 calculator operations（add/subtract/multiply/divide）都在 agent loop 中完成
• **Clarification flow** - 使用 ClarificationRequest tool 處理模糊輸入
• **Serialization formats** - thread history 可使用 JSON 或 XML（XML 更省 tokens）
• **漸進式複雜度** - 從 hello world 開始，逐步加入 BAML、tools、loops、tests

## Chapter Implementation Status
## Chapter 實作狀態

• **Chapter 0**: Hello World - Simple Python program, no BAML ✅
• **Chapter 1**: CLI and Agent - BAML introduction, basic agent ✅
• **Chapter 2**: Calculator Tools - Tool definitions without execution ✅
• **Chapter 3**: Tool Loop - Full agent loop with tool execution ✅
• **Chapter 4**: BAML Tests - Test cases with assertions ✅
• **Chapter 5**: Human Tools - Clarification requests with input handling ✅
• **Chapter 6**: Improved Prompting - Reasoning steps in prompts ✅
• **Chapter 7**: Context Serialization - JSON/XML thread formats ✅
• **Chapters 8-12**: Skipped - Server-based features not suitable for notebooks ⚠️

• **Chapter 0**：Hello World - 簡單的 Python 程式，不含 BAML ✅
• **Chapter 1**：CLI and Agent - 介紹 BAML 與基本 agent ✅
• **Chapter 2**：Calculator Tools - 僅有 tool 定義，尚未執行 ✅
• **Chapter 3**：Tool Loop - 含 tool 執行的完整 agent loop ✅
• **Chapter 4**：BAML Tests - 帶有 assertions 的測試案例 ✅
• **Chapter 5**：Human Tools - 含輸入處理的 clarification requests ✅
• **Chapter 6**：Improved Prompting - 在 prompts 中加入 reasoning steps ✅
• **Chapter 7**：Context Serialization - JSON/XML thread formats ✅
• **Chapters 8-12**：已跳過 - 以 server 為基礎的功能不適合 notebooks ⚠️

## Common Pitfalls Avoided
## 已避免的常見陷阱

• **Import errors** - baml_client imports fail in notebooks, use global get_baml_client
• **Async patterns** - Notebooks can't handle async/await, everything must be sync
• **File paths** - Use absolute paths from notebook directory, handle ./ prefixes
• **BAML file conflicts** - Each chapter updates same files (agent.baml) not chapter-specific
• **Tool registration** - Ensure all tool types handled in agent loop switch statement
• **Test expectations** - BAML tests may have varying outputs, assertions verify key properties
• **Environment differences** - Code must work in both Colab and local testing environments

• **Import errors** - `baml_client` imports 在 notebooks 中會失敗，請改用全域 `get_baml_client`
• **Async patterns** - notebooks 無法處理 `async/await`，所有內容都必須是同步的
• **File paths** - 使用相對於 notebook 目錄的絕對路徑，並處理 `./` 前綴
• **BAML file conflicts** - 每個 chapter 都會更新相同的 files（`agent.baml`），而不是各 chapter 專屬檔案
• **Tool registration** - 確保 agent loop 的 switch statement 會處理所有 tool types
• **Test expectations** - BAML tests 的輸出可能不同，assertions 應驗證關鍵屬性
• **環境差異** - 程式碼必須同時可在 Colab 與本機測試環境中執行

## Testing Commands
## 測試指令

• Generate notebook: `uv run python walkthroughgen_py.py walkthrough.yaml -o test.ipynb`
• Full Colab sim: `./test_notebook_colab_sim.sh`
• Run BAML tests: `baml-cli test` (from directory with baml_src)

• 產生 notebook：`uv run python walkthroughgen_py.py walkthrough.yaml -o test.ipynb`
• 完整 Colab 模擬：`./test_notebook_colab_sim.sh`
• 執行 BAML tests：`baml-cli test`（在含有 `baml_src` 的目錄中執行）

## File Structure
## 檔案結構

• `walkthrough/*.py` - Python implementations of each chapter's code
• `walkthrough/*.baml` - BAML files fetched from GitHub during notebook execution
• `walkthroughgen_py.py` - Main conversion tool
• `walkthrough.yaml` - Notebook definition with all chapters
• `test_notebook_colab_sim.sh` - Full Colab environment simulation
• `workshop_final.ipynb` - Final generated notebook ready for workshop

• `walkthrough/*.py` - 各 chapter 程式碼的 Python 實作
• `walkthrough/*.baml` - notebook 執行期間從 GitHub 抓取的 BAML files
• `walkthroughgen_py.py` - 主要的轉換工具
• `walkthrough.yaml` - 含所有 chapters 的 notebook 定義
• `test_notebook_colab_sim.sh` - 完整的 Colab 環境模擬
• `workshop_final.ipynb` - 可直接用於 workshop 的最終 notebook
