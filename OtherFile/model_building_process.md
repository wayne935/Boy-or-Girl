# 模型建立流程（Model Building Process）

本檔說明模型建立的標準流程，並且整個流程已實作在同一個 Jupyter Notebook：`model_building_process.ipynb`。

流程步驟：

1. 資料分析（Data Analysis）
	- 檢查資料型態（dtype）、缺值（missing values）、數值分佈、類別欄位的值域等。
	- 產生簡單的統計報表與可視化（例如缺值表、直方圖、箱型圖）。

2. 資料處理（Data Processing）
	- 對遺失值進行處理（例如使用 `SimpleImputer`、`IterativeImputer` 或模型本身處理）。
	- 類別特徵轉換（One-Hot Encoding、Ordinal Encoding 或 Target Encoding），數值欄位的縮放/轉換（視需要）。
	- 建立資料前處理 pipeline，保證在 Cross-Validation 迴圈內執行所有轉換以避免資料洩漏。

3. 模型選擇與建立（Model Selection & Training）
	- 使用一個或多個候選模型（例如 RandomForest、Gradient Boosting 等）。
	- 可加入超參數搜尋（GridSearch / RandomizedSearch / Optuna），並在交叉驗證內評估。

4. 模型評估（Evaluation）
	- 使用適合不平衡資料的評估指標（例如 ROC-AUC、F1-macro、PR-AUC）。
	- 使用分層 K-Fold（StratifiedKFold）或專案所需的驗證策略。

5. 產生 Kaggle submission 檔案（Submission）
	- 產生 CSV 格式：第一欄 `id`，第二欄 `gender`，範例：

```
id,gender
1,1
2,2
3,2
4,1
5,1
...
```

注意：使用者要求「以上流程需全部寫在同一個 `.ipynb` 檔案」，請打開或執行 `model_building_process.ipynb`（已建立於此專案）來查看可執行的範本與範例程式碼。

若要我把 notebook 的模型改為特定套件（例如 LightGBM / XGBoost）或加入更多展示圖表，請告訴我你的偏好。
