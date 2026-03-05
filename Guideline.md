# 🏆 Kaggle 專案最高開發準則 (Supreme Guidelines)

## 核心原則 (Core Philosophy)

- **泛化能力至上 (Generalization over Memorization)**：本專案唯一的成功指標是模型在未見過資料上的泛化能力。嚴禁為了提升單一訓練集或公開測試集（Public Leaderboard）的分數而過度調整超參數（overfitting）。

- **防範資料洩漏 (Zero Data Leakage)**：所有資料轉換、特徵工程、遺失值填補與採樣技術，必須在交叉驗證（cross-validation）的迴圈內完成；絕對禁止先對全體資料處理後再切分成訓練/驗證集。

- **資料視覺化**: 在進行任何數據處理前，應先對資料進行充分的視覺化分析，以了解數據分布、異常值、遺失值模式以及類別不平衡情況。這將有助於制定更合理的處理策略。

---

## 一、異常值與極端值處理策略 (Outlier & Anomaly Handling)

- **禁止武斷刪除 (No Arbitrary Deletion)**：面對明顯不合理的數值（例如體重 3000 公斤），不應直接刪除。尤其在問卷資料中，惡意填答可能含有群體性特徵（例如某一性別比例較高），刪除可能造成偏差。

- **特徵保留與轉換**：
	- 建立標記性布林特徵（例如 `is_troll_weight` 或 `anomaly_score`）來標示疑似惡意填答。
	- 對原始數值可採用縮尾處理（Winsorization）或將極端值視為特殊類別（例如標記為 `NaN`）由模型另行處理。

---

## 二、遺失值填補策略 (Missing Value Imputation)

- **禁止使用單純的全域平均/中位數填補**：這類方法會破壞原始分布與特徵關係。

- **推薦作法**：
	- 優先使用能自動處理遺失值的樹狀模型（如 XGBoost、LightGBM、CatBoost）。
	- 若需顯式填補，考慮多重插補（MICE / IterativeImputer）或 KNN Imputer，並同時新增 `*_is_missing` 指示欄位（indicator feature）。

---

## 三、類別不平衡處理策略 (Class Imbalance Management)

- **評估指標**：避免只使用 Accuracy；建議使用能反映不平衡情況的指標，例如 ROC-AUC、F1-macro、或 PR-AUC (Precision-Recall AUC)。

- **演算法層級優先**：優先在模型或損失函數層處理不平衡（例如 `class_weight='balanced'`、Focal Loss）。

- **資料層級採樣**：若使用 SMOTE / ADASYN / 欠採樣，僅在交叉驗證的訓練折上執行；驗證折需保留原始分布以確保真實評估。

---

## 四、模型建構與驗證框架 (Modeling & Validation Framework)

- **嚴格的交叉驗證 (Strict CV)**：使用分層 K 折（Stratified K-Fold）以維持每一折的標籤分布一致性。

- **架構選擇與防呆機制**：
	- 單一模型：可選擇對表格資料穩定且容錯性高的 Gradient Boosting 類模型。
	- 多階段 / 分層模型（Multi-stage / Cascade）：鼓勵採用分階段策略，例如：
		1. 第一階段模型：偵測「正常填答」與「亂填/惡意填答」的樣本。
		2. 第二階段模型：對不同群體（正常 vs 惡意填答）訓練專屬的預測模型。

- **Trust Your Local CV**：團隊決策應以本地的 Stratified CV 分數為最高依據。若 Local CV 下降但 Public LB 上升，極可能為過擬合，應拒絕該次更新並回頭檢查資料洩漏或評估流程。

---
## 五、團隊協作與版本控制 (Team Collaboration & Version Control)

- **使用 Git 進行版本控制**：所有的程式碼變更必須透過 Git 進行版本控制，並遵循 Git Flow 或其他團隊約定的工作流程。

- **定期合併與重構**：鼓勵團隊成員定期將自己的分支合併到主分支，並進行必要的重構，以保持代碼庫的整潔與可維護性。

- **撰寫清晰的提交訊息**：每次提交都應包含清晰且具體的訊息，描述所做的變更與原因，以便未來追蹤與回溯。

- **參數**: random_seed 一律設為 42

## 附註

本指南為高階原則與建議，實作時可依資料型態與專案需求做合理調整；但任何破壞交叉驗證流程或導致資料洩漏的做法均屬禁忌。
