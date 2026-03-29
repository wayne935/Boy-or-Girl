# Model Building v2 (雙方法並行 + 防過擬合 + 防洩漏)

本流程保留兩種可並行方法，最後以同一套 Stratified 10-Fold 指標比較，選擇最佳組合。

## 0. 核心原則（必遵守）

1. 特徵選擇不得在全資料先做。
2. 任何特徵選擇（RFE/SelectFromModel）必須放進 Pipeline，讓每一折獨立挑特徵。
3. 不用 SMOTE 當主方案，優先使用成本敏感學習（class weight / scale_pos_weight）。
4. 不做硬規則後處理（例如 height > 180 就改成 male），避免資料分布改變時失效。
5. 以 Local CV（mean + std）作為主決策，不只看單次 split 或 Public LB。

## 1. 資料分析與清理

1. 缺失值掃描：各欄位 missing ratio，>70% 欄位再評估是否保留。
2. 目標分布：檢查 gender 不平衡比例，供後續權重計算。
3. 異常值處理：將極端值轉為 NaN，再由 imputer 補值。
	- height: <140 或 >200 -> NaN
	- weight: <35 或 >120 -> NaN
	- iq: <50 或 >180 -> NaN
	- fb_friends: >5000 -> NaN

## 2. 共同特徵工程（兩方法都用）

1. 生理特徵
	- bmi = weight / (height/100)^2
	- height percentile（連續）
	- height bin（離散化）：low / mid / high
	- is_tall（二元）：height >= 170

2. 類別特徵
	- star_sign -> 四象（fire/water/air/earth）
	- phone_os 缺失補 Unknown

3. 文本基礎特徵
	- has_intro, intro_len
	- uses_emoji, tilde_count（~ 或 ～）, exclamation_count（!）

## 3. 方法 A（可解釋特徵優先）

適合穩定、可解釋、較低過擬合風險。

1. Keyword Score
	- male_words = ['帥', '打球', '魯蛇', '遊戲', '電競']
	- female_words = ['美', '可愛', '妝', '保養', '香水']
	- keyword_score = count(male_words) - count(female_words)

2. Stylometric 特徵
	- emoji_ratio, punctuation_ratio
	- tilde_count, exclamation_count

3. 模型
	- XGBoost（正則化強化）
	- RandomForest（class_weight='balanced'）

4. XGBoost 建議範圍
	- max_depth: 4-6
	- min_child_weight: 5-10
	- reg_alpha: 0.5-5.0
	- reg_lambda: 1.0-10.0
	- subsample: 0.7-0.9
	- colsample_bytree: 0.7-0.9


## 5. 防洩漏 Pipeline 設計（重點）

每個候選模型都必須使用 Pipeline。

1. 前處理（imputer / encoder / scaler）
2. 特徵選擇（RFE 或 SelectFromModel）
3. 模型（XGB/LGB/RF）

注意：
1. 不能先用全資料做 RFE 再 CV。
2. 所有步驟都要在 cross-validation 的每個 fold 內 fit。

## 6. 不平衡處理策略

主策略：成本敏感學習（Cost-Sensitive Learning）

1. XGBoost
	- scale_pos_weight = n_negative / n_positive

2. RandomForest / LightGBM
	- class_weight='balanced'

備選策略（非主方案）：
1. IHT（Instance Hardness Threshold）可少量實驗。
2. SMOTE 不建議作主線，僅作對照組。

## 7. 評估與模型選擇（兩方法都保留）

統一評估 protocol：

1. Stratified 10-Fold CV
2. 主要指標：ROC-AUC、F1-macro
3. 次要指標：Accuracy

最後決策規則：

1. 先比 ROC-AUC mean（越高越好）
2. 若接近（差距 <= 0.003），選 std 較低者（越穩定越好）
3. 若仍接近，選較簡單可解釋的方法（優先方法 A）

## 8. 建議實驗矩陣（最少保留 4 組）

1. A + XGBoost（Pipeline + RFE in-fold）
2. A + RandomForest（Pipeline + SelectFromModel）
3. B + LightGBM（Pipeline + 特徵選擇 in-fold）
4. B + XGBoost（Pipeline + 強正則）

最後可再做：
1. Soft Voting（最佳 A 模型 + 最佳 B 模型）
2. 若 Voting 在 10-Fold mean 與 std 都更好，才採用。

## 9. Submission 規則

1. 使用最終選定模型在完整訓練集 fit。
2. 輸出 submission：id,gender。
3. 僅保留模型輸出，不加人工硬規則修正。

---

本檔已更新為雙方法並行流程：
1. 方法 A（關鍵字/語氣/離散化，可解釋）
2. 方法 B（語義 embedding，語義捕捉力高）

兩者都保留，最終依同一套 10-Fold 結果選最佳組合。