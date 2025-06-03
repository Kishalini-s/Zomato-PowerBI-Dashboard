# Data Cleaning and Transformation Process for Zomato Dashboard

This document details the step-by-step data cleaning and transformation processes applied to the raw Zomato dataset within Power Query Editor in Power BI. These steps were crucial to ensure data quality, consistency, and suitability for analytical modeling.

## Tools Used

* **Power Query Editor (M Language):** All transformations were performed using Power Query's graphical interface and custom M functions.
* **Power BI Desktop:** For data loading and modeling.

## Process Overview

The raw Zomato dataset underwent a series of transformations, organized as follows:

### 1. Initial Data Loading and Column Selection

* **Purpose:** To load the raw data and immediately remove columns that were not relevant for the planned analysis, reducing dataset size and complexity.
* **Action:** Removed the following columns:
    * `url`: (Reason: Not required for analysis, unique identifier)
    * `address`: (Reason: Detailed address not needed; `City` and `Locality` suffice)
    * `phone`: (Reason: Not relevant for analytical insights)
    * `menu_item`: (Reason: Contained nested or complex data not used in this scope)
    * `dish_liked`: (Reason: Highly varied text, not suitable for direct aggregate analysis without NLP)
    * `reviews_list`: (Reason: Highly varied text, not suitable for direct aggregate analysis without NLP)

### 2. Column Renaming for Clarity

* **Purpose:** To improve readability and consistency of column headers for easier understanding and DAX formula creation.
* **Action:** Renamed columns from their original format to more descriptive names:
    * `rate` -> `Rating_out_of_5`
    * `approx_cost(for two people)` -> `Cost_for_two`
    * `rest_type` -> `Restaurant_Type`

### 3. Removing Duplicate Rows

* **Purpose:** To ensure each record represents a unique restaurant entry, preventing overcounting or skewed statistics.
* **Action:** Identified and removed duplicate rows based on a combination of identifying columns to ensure uniqueness.
    * "Removed duplicates based on `Restaurant_Name`

### 4. Splitting 'Rating' Column

* **Purpose:** The original `Rating_out_of_5` column (e.g., "3.5/5") contained both the rating value and the number of votes within a single text string. This step separates these into distinct numerical columns.
* **Action:**
    * The `Rating` column was first transformed to handle inconsistent values (e.g., `-` or `NEW` ratings converted to `null`).
    * It was then split using the delimiter `/` to extract the numerical rating value.
    * **Resulting Columns:**
        * `Rating_out_of_5` (Decimal Number)
        * `Number_of_Votes` (Whole Number)

### 5. Cleaning 'Restaurant_Type' Column

* **Purpose:** The `Restaurant_Type` column often contained multiple types separated by commas, or inconsistent spellings. This step standardized and simplified these entries.
* **Action:**
    * Standardized common variations (e.g., converting "QSR" to "Quick Bites").
    * Handled multiple types by (e.g., taking the first type or categorizing into broader `Service_Category` as seen below).
    * *(Detail any specific `Text.Replace` or conditional logic you used here)*

### 6. Handling Missing Values (Imputation)

* **Purpose:** To fill `null` values in critical numerical and categorical columns using statistically robust methods, thereby maintaining data integrity and enabling comprehensive analysis.
* **Methodology:** Leveraged Power Query's `Table.Group` and custom functions to perform contextual imputation, similar to `groupby().transform()` in Python Pandas.
* **Custom Functions Used:**
    * `fGetMedian(list as list)`: Calculates the median of a list, handling `null`s and empty lists robustly.
    * `fGetMode(list as list)`: Calculates the mode (most frequent item) of a list, handling `null`s, empty lists, and ties robustly.

* **Detailed Imputation Steps:**

    * #### `Cost_for_two` Column Imputation
        1.  **Initial Cleaning:** Removed commas (`,`) from string values and converted the column to `Decimal Number` type, coercing errors to `null`.
        2.  **Level 1 Imputation (City, Restaurant_Type):** `null` values in `Cost_for_two` were filled with the median `Cost_for_two` of restaurants belonging to the **same `City` AND `Restaurant_Type`** group.
        3.  **Level 2 Imputation (City, Service_Category):** Any remaining `null`s were then filled using the median `Cost_for_two` of restaurants belonging to the **same `City` AND `Service_Category`** group (where `Service_Category` was derived based on `online_order`, `book_table`, `Restaurant_Type`).

    * #### `cuisines` Column Imputation
        1.  **Initial Cleaning:** Ensured all missing values were explicitly `null` (not empty strings).
        2.  **Level 1 Imputation (City, Restaurant_Type):** `null` values in `cuisines` were filled with the mode (most frequent cuisine) of restaurants within the **same `City` AND `Restaurant_Type`** group.
        3.  **Level 2 Imputation (City, Service_Category):** Any remaining `null`s were then filled using the mode of the `cuisines` within the **same `City` AND `Service_Category`** group.

    * #### `Rating_out_of_5` Column Imputation
        1.  **Initial Cleaning:** Handled inconsistent `Rating` values (e.g., "-", "NEW") by converting them to `null` before numerical conversion.
        2.  **Level 1 Imputation (City, cuisines, Restaurant_Type):** `null` values in `Rating_out_of_5` were filled with the median `Rating_out_of_5` of restaurants belonging to the **same `City`, `cuisines`, AND `Restaurant_Type`** group.
        3.  **Level 2 Imputation (City):** Any remaining `null`s were then filled using the overall median `Rating_out_of_5` from the broader **`City`** group alone.

### 7. Data Type Correction

* **Purpose:** To ensure all columns have the appropriate data types for accurate calculations and visualizations in Power BI.
* **Action:** Final type conversions were applied to all columns (e.g., `votes` to Whole Number, `Cost_for_two` to Decimal Number, text columns to Text type).

---