# CH12 -- comparison
## objectives 
You will know how to

    + compare two number, string etc.

## CH12-1 -- comparison of number
Like `C#`,

| operator | description |
| :-- | :-- |
| `==` | equal to |
| `!=` | not equal to |
| `<` | less than |
| `>` | greater than |
| `<=` | less than or equal to |
| `>=` | greater than or equal to |

### Examples
#### Example 1

```
name: Comparison Operator Showcase

on: [push] # 為了演示，我們使用 push 事件觸發

env:
  # 設置我們要比較的數值
  LINES_CHANGED: 150
  # 設置我們要比較的布林值 (字串形式)
  IS_CRITICAL: 'true' 

jobs:
  compare_and_evaluate:
    runs-on: ubuntu-latest
    
    steps:
      - name: 1. 檢查完全相等 (==) 與不相等 (!=)
        if: ${{ env.LINES_CHANGED == 150 && env.IS_CRITICAL != 'false' }}
        run: echo "結果 1: 變更行數恰好是 150 **且** 它是關鍵性變更。 (== 和 != 測試通過)"

      - name: 2. 檢查大於 (>) 與大於等於 (>=)
        if: ${{ env.LINES_CHANGED > 100 && env.LINES_CHANGED >= 150 }}
        run: echo "結果 2: 變更行數大於 100 **且** 大於等於 150。 (> 和 >= 測試通過)"

      - name: 3. 檢查小於 (<) 與小於等於 (<=)
        if: ${{ env.LINES_CHANGED < 200 || env.LINES_CHANGED <= 150 }}
        run: echo "結果 3: 變更行數小於 200 **或** 小於等於 150。 (< 和 <= 測試通過)"

      - name: 4. 複合邏輯判斷 (混用所有運算符)
        id: final_check
        # 複雜判斷邏輯：
        # (行數在 [100, 200] 區間 且 是關鍵性變更) OR (行數剛好是 150)
        if: ${{ (env.LINES_CHANGED >= 100 && env.LINES_CHANGED <= 200 && env.IS_CRITICAL == 'true') || env.LINES_CHANGED == 150 }}
        run: echo "結果 4: 最終複合檢查通過，該變更滿足高標準審查條件。"
        
      # 使用 steps 輸出和 job 輸出，展示如何將結果傳遞給下一個 Job (高階函式應用)
      - name: 5. 輸出測試結果
        id: set_output
        run: echo "COMPLEX_RESULT=Success" >> $GITHUB_OUTPUT
        if: ${{ always() }} # 確保無論步驟 4 是否執行都設置輸出

  next_job_using_output:
    runs-on: ubuntu-latest
    needs: compare_and_evaluate
    steps:
      - name: 呼叫並檢查前一個 Job 的輸出
        run: |
          echo "前一個 Job 的最終檢查結果是："
          # 注意：這裡的輸出是步驟 5 設置的，我們通過 if 條件間接判斷了步驟 4 的結果
          echo "${{ needs.compare_and_evaluate.outputs.final_result }}"
    outputs:
      # 將步驟 5 的輸出設置為 Job 輸出
      final_result: ${{ steps.set_output.outputs.COMPLEX_RESULT }}
```
