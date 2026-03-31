# 第六週練習：降雨空間內插與不確定性分析
## 遙感與地理資訊分析應用 - Week 6 Exercise

## 📋 專案概述

本練習展示先進的降雨資料空間內插技術，比較傳統統計地質方法（克利金法）與機器學習方法（隨機森林）。重點在於理解預測不確定性並為防災決策提供依據。

### 🎯 學習目標
- 掌握常態克利金法進行空間內插
- 理解變異圖分析與金塊效應
- 比較統計地質與機器學習方法
- 分析預測不確定性與信賴度等級
- 執行分區統計以支援行政決策
- 生成專業地理空間輸出（GeoTIFF）

## 📁 資料結構

```
0331-week6-Exercise/
├── Week6-Student.ipynb          # 主要練習筆記本
├── README.md                     # 本檔案
├── data/
│   └── District Boundaries/
│       └── TOWN_MOI_1140318.shp  # 台灣鄉鎮邊界（TGOS）
├── kriging_rainfall.tif          # 輸出：克利金降雨估計
├── kriging_variance.tif          # 輸出：克利金變異數圖
└── rf_rainfall.tif               # 輸出：隨機森林降雨估計
```

## 🔧 需求套件

### Python 套件
```python
# 核心科學計算
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

# 地理空間分析
from pykrige.ok import OrdinaryKriging
import geopandas as gpd
from rasterstats import zonal_stats
import rasterio
from rasterio.transform import from_bounds

# 機器學習
from sklearn.ensemble import RandomForestRegressor
from sklearn.model_selection import train_test_split

# 視覺化
import seaborn as sns
```

### 外部資料
- **台灣鄉鎮邊界**：`TOWN_MOI_1140318.shp`（TGOS圖檔）
- 座標參考系統：EPSG:3826（TWD97 / TM2 zone 121）

## 📊 練習章節

### 1. 資料載入與探索
- 載入降雨測站資料（座標與量測值）
- 視覺化降雨測站空間分布
- 降雨觀測值之基礎統計分析

### 2. 變異圖分析
```python
# 涵蓋關鍵概念：
- 實驗變異圖計算
- 理論變異圖模型（球形、指數、高斯）
- 金塊效應及其物理意義
- 變程與基台參數
```

### 3. 常態克利金法實作
```python
# 核心實作：
ok = OrdinaryKriging(
    x, y, z_log, 
    variogram_model='spherical',
    variogram_parameters={'sill': sill_val, 'range': range_val, 'nugget': nugget_val}
)
```

### 4. 金塊效應分析
- **高金塊（10%）**：假設量測誤差/雜訊
- **低金塊（1%）**：高度信任量測值
- 比較極端值周圍的預測行為（蘇澳測站）

### 5. 機器學習比較
```python
# 空間內插的隨機森林
rf = RandomForestRegressor(n_estimators=100, random_state=42)
rf.fit(X_train, y_train)
```

### 6. 不確定性量化
- 克利金變異數圖
- 信賴度分類（高/中/低）
- 空間不確定性視覺化

### 7. 分區統計
- 計算行政邊界內的降雨統計
- 生成決策支援表格供緊急管理使用
- 基於信賴度的風險評估

## 🗂️ 主要輸出

### 生成的 GeoTIFF 檔案
1. **`kriging_rainfall.tif`**：克利金降雨估計（毫米/小時）
2. **`kriging_variance.tif`**：預測不確定性圖
3. **`rf_rainfall.tif`**：隨機森林降雨估計

### 視覺化產品
- 含測站位置的降雨估計圖
- 不確定性/信賴度圖
- 金塊效應比較圖
- 行政邊界統計表格

## 📈 關鍵發現與見解

### 克利金法 vs. 機器學習
| 層面 | 克利金法 | 隨機森林 |
|--------|---------|----------|
| **不確定性量化** | ✅ 提供變異數圖 | ❌ 無內建不確定性 |
| **空間連續性** | ✅ 平滑、物理真實 | ❌ 方塊狀偽影 |
| **外推能力** | ✅ 理論健全 | ❌ 訓練範圍外差 |
| **可解釋性** | ✅ 統計參數 | ❌ 黑箱模型 |
| **計算效率** | ✅ 中等資料集快速 | ❌ 大網格較慢 |

### 金塊效應影響
- **高金塊（10%）**：平滑預測，降低極端值
- **低金塊（1%）**：保留極端值，較高空間變異性
- **決策影響**：低金塊更適合偵測局部極端事件

### 行政決策支援
分析提供鄉鎮級別降雨統計與信賴度評等：
- **高信賴度**：變異數 < 33百分位數
- **中信賴度**：33-66百分位數  
- **低信賴度**：變異數 > 66百分位數

## 🚨 實務應用

### 防災管理
- **早期預警**：識別高降雨預測的高風險鄉鎮
- **資源分配**：基於信賴度等級優先處理區域
- **疏散規劃**：使用不確定性圖進行保守規劃

### 氣象服務
- **資料缺口分析**：變異數圖揭示觀測盲點
- **網路優化**：識別需要增設測站的區域
- **模式驗證**：比較不同內插方法

## 🔍 技術說明

### 座標系統
- **輸入資料**：假設經緯度座標
- **處理**：EPSG:3826（TWD97 / TM2 zone 121）
- **輸出**：具備地理參考的 GeoTIFF

### 資料處理
- **對數轉換**：應用於降雨資料以達常態化
- **反轉換**：對預測值應用指數函數
- **無資料處理**：-9999 值代表缺失資料

### 記憶體考量
- 網格解析度顯著影響記憶體使用量
- 大範圍區域考慮分塊處理
- 為計算效率優化變異圖參數

## 🛠️ 疑難排解

### 常見問題
1. **圖檔路徑錯誤**：在 Windows 上使用原始字串 `r"路徑\到\檔案.shp"`
2. **記憶體問題**：降低網格解析度或範圍
3. **座標不符**：確保所有資料使用相同 CRS
4. **缺少相依套件**：透過 `pip install` 安裝所需套件

### 效能技巧
- 使用適當的網格解析度（平衡細節與效能）
- 重複分析時快取變異圖計算
- 大型資料集考慮平行處理

## 📚 參考資料

### 學術資源
- Cressie, N. A. C. (1993). *Statistics for Spatial Data*. Wiley.
- Isaaks, E. H., & Srivastava, R. M. (1989). *An Introduction to Applied Geostatistics*. Oxford University Press.
- Oliver, M. A., & Webster, R. (2014). A tutorial guide to geostatistics: Computing and modelling variograms and kriging. *Catena*, 113, 56-69.

### 技術文件
- [PyKrige 文件](https://github.com/bsmurphy/PyKrige)
- [GeoPandas 使用指南](https://geopandas.org/en/stable/docs.html)
- [Rasterstats 文件](https://pythonhosted.org/rasterstats/)

## 🤝 貢獻

本練習為「遙感與地理資訊分析應用」課程的一部分。如有問題、建議或改進，請聯繫課程指導老師。

## 📄 授權

本教育材料僅供學術用途使用。在研究或出版物中使用時，請確保適當引用。

---

**課程**：遙感與地理資訊分析應用  
**週次**：6 - 空間內插與不確定性分析  
**指導老師**：[指導老師姓名]  
**日期**：2026年3月  
**版本**：1.0
