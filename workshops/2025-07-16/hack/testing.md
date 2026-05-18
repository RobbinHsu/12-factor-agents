# Jupyter Notebook Testing Framework
# Jupyter Notebook 測試框架

This document describes the general testing framework for validating any functionality in Jupyter notebooks, with a specific example of testing BAML log capture.

本文說明用於驗證 Jupyter notebooks 中各種功能的一般測試框架，並以測試 BAML log capture 作為具體範例。

## General Framework
## 一般框架

### Overview
### 概述

The testing framework provides a complete iteration loop for testing notebook implementations:

此測試框架提供一個完整的迭代循環，用來測試 notebook 實作：

1. **Generate** test notebooks with specific functionality 
2. **Execute** notebooks in a simulated Google Colab environment  
3. **Analyze** executed notebooks for expected outputs and behaviors
4. **Report** clear pass/fail results

1. **Generate** 具有特定功能的測試 notebooks
2. **Execute** 在模擬的 Google Colab 環境中執行 notebooks
3. **Analyze** 已執行 notebooks 是否具有預期輸出與行為
4. **Report** 清楚的通過／失敗結果

### Core Components
### 核心組件

#### Notebook Simulator (`test_notebook_colab_sim.sh`)
#### Notebook Simulator（`test_notebook_colab_sim.sh`）

The simulation script creates a realistic Google Colab environment for any notebook:

此模擬腳本會為任何 notebook 建立貼近真實的 Google Colab 環境：

**Environment Setup:**

**環境設定：**

- Creates timestamped test directory: `./tmp/test_YYYYMMDD_HHMMSS/`
- Sets up fresh Python virtual environment
- Installs Jupyter dependencies (`notebook`, `nbconvert`, `ipykernel`)

- 建立帶有時間戳記的測試目錄：`./tmp/test_YYYYMMDD_HHMMSS/`
- 建立全新的 Python virtual environment
- 安裝 Jupyter dependencies（`notebook`、`nbconvert`、`ipykernel`）

**Notebook Execution:**

**Notebook 執行：**

- Copies test notebook to clean environment
- Uses `ExecutePreprocessor` to run all cells (simulates Colab execution)
- **Critical:** Activates virtual environment before execution
- **Critical:** Saves executed notebook with cell outputs back to disk

- 將測試 notebook 複製到乾淨的環境中
- 使用 `ExecutePreprocessor` 執行所有 cells（模擬 Colab 執行）
- **Critical：** 在執行前啟用 virtual environment
- **Critical：** 將包含 cell outputs 的已執行 notebook 寫回磁碟

**Usage:**

**使用方式：**

```bash
./test_notebook_colab_sim.sh your_notebook.ipynb
```

The simulator will:

模擬器會：

- Execute all cells in the notebook
- Preserve the test directory for inspection
- Show final directory structure
- Report success/failure

- 執行 notebook 中的所有 cells
- 保留測試目錄供檢查使用
- 顯示最終目錄結構
- 回報成功／失敗

#### Output Inspector (`inspect_notebook.py`)
#### Output Inspector（`inspect_notebook.py`）

Debug utility for examining notebook cell outputs in detail:

這是一個用來詳細檢查 notebook cell outputs 的除錯工具：

**Features:**

**功能：**

- Shows cell source code and execution counts  
- Displays all output types (stream, execute_result, error)
- Highlights patterns in output text
- Shows execution errors with tracebacks
- Filters cells by keywords for focused debugging

- 顯示 cell 原始碼與 execution counts  
- 顯示所有 output types（stream、execute_result、error）
- 標示 output text 中的模式
- 顯示含 traceback 的執行錯誤
- 依關鍵字篩選 cells，以便聚焦除錯

**Usage:**

**使用方式：**

```bash
# Inspect all cells
# 檢查所有 cells
python3 inspect_notebook.py path/to/notebook.ipynb

# Filter for specific content
# 依特定內容篩選
python3 inspect_notebook.py path/to/notebook.ipynb "keyword"

# Look for errors
# 尋找錯誤
python3 inspect_notebook.py path/to/notebook.ipynb "error"
```

**Sample Output:**

**範例輸出：**

```
🔍 CELL 0 (code)
📝 SOURCE:
import sys
print("Hello!")
print("Error!", file=sys.stderr)

📤 OUTPUTS (2 outputs):
  Output 0: type=stream
    Text length: 7 chars
    > Hello!...
  Output 1: type=stream  
    Text length: 7 chars
    > Error!...
    🎯 Found patterns: ['Error']
```

### Key Insights for Notebook Testing
### Notebook 測試的重要觀察

#### Execution Environment
#### 執行環境

1. **Virtual environment activation is critical** - Without it, execution fails silently
2. **Output persistence must be explicit** - `ExecutePreprocessor` only modifies notebook in memory
3. **Check execution counts** - `execution_count=None` means cell never executed
4. **Handle different output types** - stream, execute_result, error, display_data

1. **Virtual environment activation 非常重要** - 少了它，執行可能會靜默失敗
2. **Output persistence 必須明確處理** - `ExecutePreprocessor` 只會修改記憶體中的 notebook
3. **檢查 execution counts** - `execution_count=None` 代表 cell 從未執行
4. **處理不同 output types** - stream、execute_result、error、display_data

#### Common Debugging Steps
#### 常見除錯步驟

1. **Verify basic execution:**

1. **驗證基本執行：**

   ```bash
   python3 -c "
   import json
   nb = json.load(open('path/to/notebook.ipynb'))
   print('Execution counts:', [cell.get('execution_count') for cell in nb['cells'] if cell['cell_type']=='code'])
   "
   ```

2. **Check for execution errors:**

2. **檢查執行錯誤：**

   ```bash
   python3 inspect_notebook.py path/to/notebook.ipynb "error"
   ```

3. **Look for specific output patterns:**

3. **尋找特定輸出模式：**

   ```bash
   python3 inspect_notebook.py path/to/notebook.ipynb "your_pattern"
   ```

### Creating Custom Tests
### 建立自訂測試

#### 1. Minimal Test Template
#### 1. 最小測試範本

Create a simple notebook that tests basic functionality:

建立一個簡單的 notebook 來測試基本功能：

```json
{
  "cells": [
    {
      "cell_type": "code",
      "execution_count": null,
      "metadata": {},
      "outputs": [],
      "source": [
        "# Test basic execution\\n",
        "# 測試基本執行\\n",
        "print('Hello from notebook!')\\n",
        "\\n",
        "# Test file creation\\n",
        "# 測試檔案建立\\n",
        "with open('test.txt', 'w') as f:\\n",
        "    f.write('Test successful\\\\n')\\n",
        "\\n",
        "# Test error handling\\n",
        "# 測試錯誤處理\\n",
        "try:\\n",
        "    result = your_function_to_test()\\n",
        "    print(f'Result: {result}')\\n",
        "except Exception as e:\\n",
        "    print(f'Error: {e}')"
      ]
    }
  ],
  "metadata": {
    "kernelspec": {
      "display_name": "Python 3",
      "language": "python", 
      "name": "python3"
    }
  },
  "nbformat": 4,
  "nbformat_minor": 4
}
```

#### 2. Test Script Template
#### 2. 測試腳本範本

```bash
#!/bin/bash
set -e

echo "🧪 Testing [Your Feature]..."

# Clean up any previous test
# 清除任何先前的測試
rm -f test_notebook.ipynb

# Generate or copy your test notebook
# 產生或複製你的測試 notebook
cp your_test_notebook.ipynb test_notebook.ipynb

# Run in simulator
# 在模擬器中執行
echo "🚀 Running test in sim..."
./test_notebook_colab_sim.sh test_notebook.ipynb

# Find the executed notebook
# 找出已執行的 notebook
NOTEBOOK_DIR=$(ls -1dt tmp/test_* | head -1)
NOTEBOOK_PATH="$NOTEBOOK_DIR/test_notebook.ipynb"

# Analyze results
# 分析結果
echo "📋 Analyzing results..."
python3 inspect_notebook.py "$NOTEBOOK_PATH" "your_search_term"

# Add your custom analysis
# 加入你的自訂分析
python3 -c "
import json
with open('$NOTEBOOK_PATH') as f:
    nb = json.load(f)

# Your custom analysis logic here
# 在這裡加入你的自訂分析邏輯
success = check_for_expected_outputs(nb)

if success:
    print('✅ PASS: Test succeeded!')
else:
    print('❌ FAIL: Test failed!')
    exit(1)
"

echo "🧹 Cleaning up..."
rm -f test_notebook.ipynb
```

---

## Use Case: BAML Log Capture Testing
## 使用案例：BAML Log Capture 測試

This section demonstrates how to use the general framework for a specific use case: testing BAML log capture in notebooks.

本節示範如何將一般框架用於特定使用案例：測試 notebooks 中的 BAML log capture。

### Problem Statement
### 問題說明

BAML (a language model framework) uses FFI bindings to a Rust binary and outputs logs to stderr. We need to test whether different log capture methods can successfully capture these logs in Jupyter notebook cells.

BAML（language model framework）會透過 FFI bindings 呼叫 Rust binary，並將 logs 輸出到 stderr。我們需要測試不同的 log capture 方法，是否能在 Jupyter notebook cells 中成功擷取這些 logs。

### Test Implementation
### 測試實作

#### Test Configuration (`simple_log_test.yaml`)
#### 測試設定（`simple_log_test.yaml`）

```yaml
title: "BAML Log Capture Test"
text: "Simple test for log capture"

sections:
  - title: "Log Capture Test"
    steps:
      - baml_setup: true
      - fetch_file:
          src: "walkthrough/01-agent.baml"
          dest: "baml_src/agent.baml"
      - file:
          src: "./simple_main.py"
      - text: "Testing log capture with show_logs=true:"
      - run_main:
          args: "What is 2+2?"
          show_logs: true
```

#### Test Function (`simple_main.py`)
#### 測試函式（`simple_main.py`）

```python
def main(message="What is 2+2?"):
    """Simple main function that calls BAML directly"""
    client = get_baml_client()
    
    # Call the BAML function - this should generate logs
    # 呼叫 BAML 函式 - 這應該會產生 logs
    result = client.DetermineNextStep(f"User asked: {message}")
    
    print(f"Input: {message}")
    print(f"Result: {result}")
    return result
```

#### Log Capture Implementation
#### Log Capture 實作

The current working implementation in `walkthroughgen_py.py`:

目前 `walkthroughgen_py.py` 中可運作的實作如下：

```python
def run_with_baml_logs(func, *args, **kwargs):
    """Test log capture using IPython capture_output"""
    # Ensure BAML_LOG is set
    # 確保已設定 BAML_LOG
    if 'BAML_LOG' not in os.environ:
        os.environ['BAML_LOG'] = 'info'
    
    print(f"[LOG CAPTURE TEST] Running with BAML_LOG={os.environ.get('BAML_LOG')}...")
    
    # Capture both stdout and stderr
    # 同時擷取 stdout 與 stderr
    with capture_output() as captured:
        result = func(*args, **kwargs)
    
    # Display captured outputs
    # 顯示擷取到的 outputs
    if captured.stdout:
        print("=== Captured Stdout ===")
        print(captured.stdout)
    
    if captured.stderr:
        print("=== Captured BAML Logs ===")
        print(captured.stderr)
    else:
        print("=== No BAML Logs Captured ===")
    
    print("=== Function Result ===")
    print(result)
    
    return result
```

### Test Execution
### 測試執行

#### Main Test Script (`test_log_capture.sh`)
#### 主要測試腳本（`test_log_capture.sh`）

```bash
#!/bin/bash
set -e

echo "🧪 Testing BAML Log Capture..."

# Generate test notebook from YAML config
# 從 YAML 設定產生測試 notebook
echo "📝 Generating test notebook..."
uv run python walkthroughgen_py.py simple_log_test.yaml -o test_capture.ipynb

# Run in simulator  
# 在模擬器中執行
echo "🚀 Running test in sim..."
./test_notebook_colab_sim.sh test_capture.ipynb

# Find the executed notebook
# 找出已執行的 notebook
NOTEBOOK_DIR=$(ls -1dt tmp/test_* | head -1)
NOTEBOOK_PATH="$NOTEBOOK_DIR/test_notebook.ipynb"

echo "📋 Analyzing results from $NOTEBOOK_PATH..."

# Debug output
# 除錯輸出
echo "🔍 Dumping debug info..."
python3 inspect_notebook.py "$NOTEBOOK_PATH" "run_with_baml_logs"

# Analyze for BAML log patterns
# 分析 BAML log patterns
echo "📊 Running log capture analysis..."
python3 analyze_log_capture.py "$NOTEBOOK_PATH"

echo "🧹 Cleaning up..."
rm -f test_capture.ipynb
```

#### Analysis Script (`analyze_log_capture.py`)
#### 分析腳本（`analyze_log_capture.py`）

```python
#!/usr/bin/env python3
import json
import sys
import os

def check_logs(notebook_path):
    """Check if BAML logs were captured in the notebook"""
    
    with open(notebook_path) as f:
        nb = json.load(f)
    
    found_log_pattern = False
    found_capture_test = False
    
    for i, cell in enumerate(nb['cells']):
        if cell['cell_type'] == 'code' and 'outputs' in cell:
            source = ''.join(cell.get('source', []))
            if 'run_with_baml_logs' in source:
                found_capture_test = True
                print(f'Found log capture test in cell {i}')
                
                # Check outputs for BAML logs
                # 檢查 outputs 中是否含有 BAML logs
                for output in cell['outputs']:
                    if output.get('output_type') == 'stream' and 'text' in output:
                        text = ''.join(output['text'])
                        # Look for the specific BAML log pattern
                        # 尋找特定的 BAML log pattern
                        if '---Parsed Response (class DoneForNow)---' in text:
                            found_log_pattern = True
                            print(f'✅ FOUND BAML LOG PATTERN in cell {i} output!')
    
    return found_capture_test, found_log_pattern

# Run analysis and return pass/fail
# 執行分析並回傳通過／失敗結果
capture_test_found, log_pattern_found = check_logs(sys.argv[1])

if not capture_test_found:
    print('❌ FAIL: No log capture test found in notebook')
    sys.exit(1)

if log_pattern_found:
    print('✅ PASS: BAML logs successfully captured in notebook output!')
    sys.exit(0)
else:
    print('❌ FAIL: BAML log pattern not found in captured output')
    sys.exit(1)
```

### Expected Output Flow
### 預期輸出流程

#### Successful Test Run:
#### 成功的測試執行：

```bash
$ ./test_log_capture.sh

🧪 Testing BAML Log Capture...
📝 Generating test notebook...
Generated notebook: test_capture.ipynb
🚀 Running test in sim...
🧪 Creating clean test environment in: ./tmp/test_20250716_191106
📁 Test directory will be preserved for inspection
🐍 Creating fresh Python virtual environment...
📦 Installing Jupyter dependencies...
🏃 Running notebook in clean environment...
✅ Notebook executed successfully!
💾 Executed notebook saved with outputs

📋 Analyzing results from tmp/test_20250716_191106/test_notebook.ipynb...
🔍 Dumping debug info...
Found log capture test in cell 11

📤 OUTPUTS (3 outputs):
  Output 0: type=stream
    Text length: 49 chars
    > [LOG CAPTURE TEST] Running with BAML_LOG=info......
  Output 1: type=stream
    Text length: 1272 chars
    > 2025-07-16T19:11:22.445 [BAML [92mINFO[0m] [35mFunction DetermineNextStep[0m...
    🎯 Found patterns: ['BAML', 'Parsed', 'Response']

📊 Running log capture analysis...
Found log capture test in cell 11
✅ FOUND BAML LOG PATTERN in cell 11 output!
✅ PASS: BAML logs successfully captured in notebook output!
🧹 Cleaning up...
```

### Key BAML-Specific Insights
### BAML 專屬的重要觀察

1. **BAML logs go to stderr** - Due to FFI bindings to Rust binary
2. **Requires `BAML_LOG=info`** - Environment variable controls verbosity  
3. **Logs include ANSI color codes** - Need to handle terminal formatting
4. **Pattern matching** - Look for `---Parsed Response (class DoneForNow)---` to confirm successful execution
5. **IPython capture_output() works** - Successfully captures stderr in notebook context

1. **BAML logs 會輸出到 stderr** - 因為它透過 FFI bindings 呼叫 Rust binary
2. **需要 `BAML_LOG=info`** - 由環境變數控制詳細程度  
3. **Logs 包含 ANSI color codes** - 需要處理終端格式化
4. **Pattern matching** - 尋找 `---Parsed Response (class DoneForNow)---` 以確認執行成功
5. **`IPython capture_output()` 可正常運作** - 能在 notebook 情境中成功擷取 stderr

### Iteration Loop Benefits
### 迭代循環的好處

This framework enables rapid testing of different log capture approaches:

這個框架可以讓你快速測試不同的 log capture 方法：

1. **Modify** the `run_with_baml_logs` function in `walkthroughgen_py.py`
2. **Run** `./test_log_capture.sh`  
3. **Get** immediate pass/fail feedback
4. **Debug** with `inspect_notebook.py` if needed
5. **Repeat** until working implementation found

1. **Modify** `walkthroughgen_py.py` 中的 `run_with_baml_logs` 函式
2. **Run** `./test_log_capture.sh`  
3. **Get** 立即的通過／失敗回饋
4. **Debug** 必要時使用 `inspect_notebook.py`
5. **Repeat** 直到找到可運作的實作為止

This same pattern can be applied to test any notebook functionality: library integrations, environment setup, output formatting, error handling, etc.

相同的模式也可以套用到任何 notebook 功能的測試：library integrations、environment setup、output formatting、error handling 等。
