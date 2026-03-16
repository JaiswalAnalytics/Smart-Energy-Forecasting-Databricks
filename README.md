# Smart-Energy-Forecasting-Databricks
An end-to-end Databricks AI pipeline predicting commercial building energy consumption using a Medallion Architecture, PySpark, and MLflow.


# ⚡ Smart Energy Forecasting: End-to-End Databricks AI Pipeline

**Author:** Shubham Jaiswal  
**Challenge:** Build With Databricks - 2 (Advanced AI Challenge)  
**Presentation Video:** [Insert YouTube/Drive Link to your 10-Min Video]  
**LinkedIn Post:** [Insert Link to your LinkedIn Post]

---

## 📖 1. Problem Definition & Business Impact
Commercial buildings consume massive amounts of energy. Facility managers often rely on static schedules for HVAC (Heating, Ventilation, and Air Conditioning) and lighting, leading to massive energy waste and high operational costs during off-peak hours or mild weather. 

**The AI Solution:** This project builds a time-series forecasting system to predict the hourly electricity consumption of commercial buildings. By forecasting exact energy needs based on building characteristics and dynamic weather patterns, facility managers can optimize HVAC schedules, reduce peak-load grid costs, and lower their carbon footprint.

---

## 🏗️ 2. Architecture & Tech Stack
graph TD
    %% Data Sources
    subgraph Sources [Data Sources]
        A[Kaggle: train.csv] --> V[(Unity Catalog Volume)]
        B[Kaggle: building_metadata.csv] --> V
        C[Kaggle: weather_train.csv] --> V
    end

    %% Bronze Layer
    subgraph Bronze [🥉 Bronze Layer: Raw Data]
        V --> B1[(bronze_meter_readings)]
        V --> B2[(bronze_building_metadata)]
        V --> B3[(bronze_weather)]
    end

    %% Silver Layer
    subgraph Silver [🥈 Silver Layer: Cleaned & Joined]
        B1 --> S1[(silver_energy_data)]
        B2 --> S1
        B3 --> S1
        S1 -.->|OPTIMIZE & ZORDER| S1
    end

    %% Gold Layer
    subgraph Gold [🥇 Gold Layer: ML Ready & Features]
        S1 --> G1[(gold_energy_features)]
        G1 -.->|Feature Engineering: Hour, Month, log1p| G1
    end

    %% ML & Inference Layer
    subgraph AI [🤖 Machine Learning & MLflow]
        G1 --> ML1((Random Forest Regressor))
        ML1 -.->|Logged Artifacts & Metrics| MLflow[MLflow Registry]
        ML1 --> G2[(gold_energy_predictions)]
    end

---

## 🌊 3. Data Pipeline Design (Medallion Architecture)
The pipeline processes the **ASHRAE - Great Energy Predictor III** dataset (millions of IoT sensor readings, weather logs, and building metadata).

### 🥉 Bronze Layer (Raw Ingestion)
* Ingested raw `train.csv`, `building_metadata.csv`, and `weather_train.csv` directly from Unity Catalog Volumes.
* Saved as immutable Delta tables to preserve the raw state.

### 🥈 Silver Layer (Cleaning & Integration)
* **Data Quality:** Removed missing weather data rows and filtered out impossible negative energy readings.
* **Integration:** Joined 3 disparate tables into a single unified dataset using PySpark `join()`.
* **Delta Optimization:** Applied `OPTIMIZE` and `ZORDER BY (building_id, timestamp)` to physically co-locate time-series data on disk, drastically reducing query times for downstream ML training.

### 🥇 Gold Layer (Feature Engineering)
* **Feature Extraction:** Engineered temporal features (`hour`, `day_of_week`, `month`) from raw timestamps.
* **Target Transformation:** Applied a logarithmic transformation (`log1p`) to the highly skewed `meter_reading` target variable to normalize the distribution for the ML model.

---

## 🤖 4. Machine Learning & MLOps
To predict energy consumption, the pipeline trains a distributed Machine Learning model.

* **Model Selection:** `RandomForestRegressor`. Selected for its ability to capture non-linear relationships in tabular and time-series data without requiring heavy scaling.
* **Data Preparation:** Utilized `StringIndexer` for categorical variables (e.g., building primary use) and `VectorAssembler` to create the feature vectors.
* **Validation:** 80/20 Train/Test split using `randomSplit()`.
* **Experiment Tracking:** Utilized **MLflow** (`mlflow.start_run()`) to log hyperparameters (`num_trees=50`, `max_depth=5`) and evaluation metrics.
* **Performance:** Achieved a baseline **Validation RMSE of 1.25** on the test set. Model artifacts were securely staged and logged to Unity Catalog.

---

## 🔮 5. Inference Pipeline
To bridge the gap between AI and business utility, the pipeline includes a batch inference step:
1. Loads the MLflow-tracked model.
2. Generates predictions on unseen data.
3. Reverses the log transformation (`expm1`) to convert predictions back into readable electricity units.
4. Saves the final `predicted_energy_usage` vs `actual_energy_usage` into a `gold_energy_predictions` Delta table for dashboarding and facility management review.

---

## 🚀 6. Reproducibility & Setup
To run this project in your own Databricks environment:
1. Clone this repository to your Databricks Workspace.
2. Download the [ASHRAE dataset from Kaggle](https://www.kaggle.com/c/ashrae-energy-prediction/data).
3. Upload the 3 CSV files to a Unity Catalog Volume (e.g., `/Volumes/workspace/default/raw_data/`).
4. Update the `volume_path` and `target_db` variables in the notebooks to match your environment.
5. Run the notebooks sequentially (`01` through `04`).
