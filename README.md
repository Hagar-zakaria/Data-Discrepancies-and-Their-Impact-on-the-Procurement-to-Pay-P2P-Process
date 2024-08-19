# Data Discrepancies and Their Impact on the Procurement-to-Pay (P2P) Process

## Introduction

The procurement-to-pay (P2P) process is a critical business operation that ensures goods and services are ordered, received, and paid for correctly. However, discrepancies in data can disrupt this process, leading to financial losses, operational delays, and strained supplier relationships. In this article, we will explore various data discrepancies found in a P2P dataset, analyze their impact, and suggest improvements to prevent future issues.

## Data Overview

We began by examining multiple datasets that are part of the P2P process:

- **Vendor Information** (`Vendor-File.xlsx`)
- **Payments** (`Payments.xlsx`)
- **Purchase Requisitions** (`List-of-Purchase-Requisition-1.xlsx`)
- **Purchase Orders** (`List-of-Purchase-Orders.xlsx`)
- **Invoices Received** (`List-of-Invoices-Received.xlsx`)
- **Goods Receipt Notes** (`List-of-Goods-Receipt-Notes.xlsx`)

The first step in our analysis was to clean these datasets to ensure accuracy and consistency.

```python
import pandas as pd

# Load the datasets
vendor_df = pd.read_excel('Vendor-File.xlsx')
payments_df = pd.read_excel('Payments.xlsx')
purchase_requisition_df = pd.read_excel('List-of-Purchase-Requisition-1.xlsx')
purchase_orders_df = pd.read_excel('List-of-Purchase-Orders.xlsx')
invoices_received_df = pd.read_excel('List-of-Invoices-Received.xlsx')
goods_receipt_notes_df = pd.read_excel('List-of-Goods-Receipt-Notes.xlsx')


## Step 1: Data Cleaning

Data cleaning is essential to handle missing values and standardize the format across different datasets. This step ensures that subsequent analyses are based on accurate and consistent data.

```python
# Data Cleaning: Handling missing values
vendor_df.fillna({
    'SUPPLIER_NAME': 'Unknown',
    'PAYEE_NAME': 'Unknown',
    'Address': 'Not Provided',
    'POST_CODE': '0000',
    'EMAIL': 'noemail@domain.com',
    'BANK_ACCT': '00000000',
    'Date of Creation': '1900-01-01',
    'Date Last Updated': '1900-01-01',
    'Created/Updated_By': 'Unknown',
    'Reviewed_By': 'Unknown'
}, inplace=True)


# Outlier Detection and Analysis in the Procurement-to-Pay Process

## Step 2: Detecting and Analyzing Outliers

Outliers can distort the analysis and lead to incorrect conclusions. Therefore, it is essential to identify them in various financial metrics such as unit costs, purchase order amounts, and payment amounts.

### Python Code for Outlier Detection

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

# Function to clean non-numeric data
def clean_non_numeric(df, numerical_columns):
    for col in numerical_columns:
        df[col] = pd.to_numeric(df[col], errors='coerce')  # Convert to numeric, forcing errors to NaN
    return df

# Function to flag outliers based on IQR
def flag_outliers(df, numerical_columns):
    outliers_summary = {}
    for col in numerical_columns:
        if pd.api.types.is_numeric_dtype(df[col]):
            Q1 = df[col].quantile(0.25)
            Q3 = df[col].quantile(0.75)
            IQR = Q3 - Q1
            lower_bound = Q1 - 1.5 * IQR
            upper_bound = Q3 + 1.5 * IQR
            outliers = df[(df[col] < lower_bound) | (df[col] > upper_bound)]
            outliers_summary[col] = len(outliers)
            df[col + '_Outlier'] = 0
            df.loc[(df[col] < lower_bound) | (df[col] > upper_bound), col + '_Outlier'] = 1
        else:
            df[col + '_Outlier'] = 0
    return df, outliers_summary

# Function to display outliers summary
def display_outliers_summary(outliers_summary, title):
    print(f'{title}')
    for col, count in outliers_summary.items():
        print(f'Column: {col}, Number of Outliers: {count}')

# Function to save outliers to CSV
def save_outliers_to_csv(df, numerical_columns, df_name):
    outliers_df = df[df[[col + '_Outlier' for col in numerical_columns]].sum(axis=1) > 0]
    outliers_df.to_csv(f'{df_name}_outliers.csv', index=False)

# Function to visualize outliers with box plots
def visualize_outliers(df, numerical_columns, title):
    plt.figure(figsize=(12, 8))
    for col in numerical_columns:
        if pd.api.types.is_numeric_dtype(df[col]):
            plt.figure()
            sns.boxplot(x=df[col])
            plt.title(f'{title}: {col}')
            plt.show()

# Example for Purchase Orders
purchase_orders_df, outliers_summary = flag_outliers(purchase_orders_df, ['QTY', 'Unit Cost', 'PO_Amount'])


### Summary of Outliers Detected

#### Quantity Ordered (QTY)
- **Number of Outliers:** No significant outliers were detected.
- **Interpretation:** The quantities ordered were generally consistent, indicating that most orders followed typical patterns.

#### Unit Cost
- **Number of Outliers:** 79 outliers detected.
- **Range of Outliers:** Unit costs went up to approximately 160,000.
- **Interpretation:** There were several purchase orders with unusually high unit costs, suggesting a need for further review to ensure no overpricing or data entry errors occurred.

#### Purchase Order Amount (PO_Amount)
- **Number of Outliers:** 77 outliers detected.
- **Range of Outliers:** Purchase order amounts ranged up to approximately 16 million.
- **Interpretation:** These high amounts could indicate large or bulk orders, or potential errors that require investigation.


## Visualization

Below is a box plot that visualizes the distribution of unit costs, highlighting the outliers identified.

```python
import matplotlib.pyplot as plt
import seaborn as sns

# Visualize outliers with box plots
plt.figure(figsize=(12, 8))
sns.boxplot(x=purchase_orders_df['Unit Cost'])
plt.title('Box Plot of Unit Costs with Outliers')
plt.show()

### Interpretation

The box plot above shows the spread of unit costs across all purchase orders. The whiskers represent the range of typical values, while the dots beyond the whiskers represent outliers. These outliers might indicate errors or special cases that need further investigation.

### Impact on P2P Process

Identifying these outliers is crucial because they can significantly impact the accuracy of financial reports and operational efficiency. For instance, overpayment due to inflated unit costs or errors in purchase order amounts can lead to unnecessary financial losses, while underreporting might affect inventory management and supply chain operations.

### Recommendations

- **Review High-Cost Outliers:** The outliers identified with significantly higher unit costs should be reviewed for potential overpricing or data entry errors.
- **Investigate Large Purchase Orders:** Orders with unusually large amounts should be investigated to ensure they are legitimate and properly authorized.


### Additional Visualizations

#### Quantity Received (Goods Receipt Notes)

```python
# Visualize outliers in Goods Receipt Notes
plt.figure(figsize=(12, 8))
sns.boxplot(x=goods_receipt_notes_df['QTY Received'])
plt.title('Goods Receipt Notes - Outliers Visualization: QTY Received')
plt.show()


**Interpretation:** The quantity received is consistent and expected, with no significant discrepancies.

