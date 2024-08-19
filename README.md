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
```

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
```
# Step 2: Detecting and Analyzing Outliers

Outliers can distort the analysis and lead to incorrect conclusions. Therefore, it is essential to identify them in various financial metrics such as unit costs, purchase order amounts, and payment amounts.

```python
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

# Now you can run your main loop again
dataframes_and_columns = {
    'purchase_orders_df': ['QTY', 'Unit Cost', 'PO_Amount'],
    'payments_df': ['PAID AMOUNT'],
    'purchase_requisition_df': ['QTY'],
    'invoices_received_df': ['GROSS', 'NET', 'VAT'],
    'goods_receipt_notes_df': ['QTY Received']
}

for df_name, numerical_columns in dataframes_and_columns.items():
    df = eval(df_name)

    # Clean non-numeric data
    df = clean_non_numeric(df, numerical_columns)

    # Run outlier detection
    df, outliers_summary = flag_outliers(df, numerical_columns)

    # Display Outliers Summary Table
    display_outliers_summary(outliers_summary, f'{df_name} - Outliers Summary')

    # Visualize Outliers with Box Plots
    visualize_outliers(df, numerical_columns, f'{df_name} - Outliers Visualization')

    # Save outliers to CSV
    save_outliers_to_csv(df, numerical_columns, df_name)
```


![download - 2024-08-19T101621 644](https://github.com/user-attachments/assets/b385f075-93b5-4926-817e-c263c4549e76)
![download - 2024-08-19T101627 680](https://github.com/user-attachments/assets/da812309-9824-4715-83db-e583524b2d9b)
![download - 2024-08-19T101632 243](https://github.com/user-attachments/assets/f4629f9a-30e0-48f8-b8dd-5f56f3e68c67)
![download - 2024-08-19T101637 716](https://github.com/user-attachments/assets/cb9f0cda-15fb-4909-bd02-d26dcbc74c3f)
![download - 2024-08-19T101642 693](https://github.com/user-attachments/assets/7e5000c9-2765-41cb-a368-949f65adbf18)
![download - 2024-08-19T101647 882](https://github.com/user-attachments/assets/13e89165-eedc-4066-9de1-2e4e108e18c8)
![download - 2024-08-19T101657 497](https://github.com/user-attachments/assets/42057a16-f11d-46f4-a421-8e0477e6fba9)
![download - 2024-08-19T101704 075](https://github.com/user-attachments/assets/84da1b71-8571-4d59-af08-dd38a0d17005)
![download - 2024-08-19T101710 943](https://github.com/user-attachments/assets/4ac85c21-a747-4937-be6f-ba20b23974e7)

### Outlier's Analytics

#### Purchase Orders Analysis

**Total Records Analyzed:**
We analyzed 1,000 purchase orders.

**QTY (Quantity Ordered):**
- **Number of Outliers:** 0 outliers detected.
- **Interpretation:** The quantity ordered across all purchase orders is consistent, with no extreme values that stand out as unusual.

**Unit Cost:**
- **Number of Outliers:** 79 outliers detected.
- **Range of Outliers:** These outliers represent unit costs significantly higher than most, ranging up to approximately 160,000.
- **Interpretation:** Several purchase orders have unusually high unit costs, which may need further review to ensure there’s no overpricing or data entry errors.

**PO_Amount:**
- **Number of Outliers:** 77 outliers detected.
- **Range of Outliers:** These outliers range up to approximately 16 million.
- **Interpretation:** The amounts are significantly higher than typical PO amounts, suggesting large or bulk orders, or potential errors that need further investigation.

#### Payments Analysis

**Total Records Analyzed:**
We reviewed 500 payment records.

**PAID AMOUNT:**
- **Number of Outliers:** 36 outliers detected.
- **Range of Outliers:** These payments range up to approximately 160,000.
- **Interpretation:** Some payments are considerably larger than others, possibly indicating bulk payments, large contracts, or potential data entry errors.

#### Purchase Requisition Analysis

**Total Records Analyzed:**
We looked at 800 purchase requisitions.

**QTY (Quantity Requested):**
- **Number of Outliers:** 0 outliers detected.
- **Interpretation:** The quantities requested are consistent with no significant discrepancies, indicating typical and expected requisition amounts.

#### Invoices Received Analysis

**Total Records Analyzed:**
We examined 400 invoices.

**GROSS (Total Amount):**
- **Number of Outliers:** 65 outliers detected.
- **Range of Outliers:** These invoices have gross amounts ranging up to approximately 400,000.
- **Interpretation:** Several invoices show very high gross amounts, which could correspond to significant purchases or potential errors.

**NET (Net Amount):**
- **Number of Outliers:** 65 outliers detected.
- **Range of Outliers:** The net amounts for these invoices are similarly high, ranging up to approximately 350,000.
- **Interpretation:** These net amounts align with the gross amounts, but their high values warrant further review.

**VAT (Tax Amount):**
- **Number of Outliers:** 64 outliers detected.
- **Range of Outliers:** VAT amounts range up to approximately 60,000.
- **Interpretation:** High VAT amounts might indicate large transactions or potential errors in tax calculations.

#### Goods Receipt Notes Analysis

**Total Records Analyzed:**
We reviewed 600 records.

**QTY Received:**
- **Number of Outliers:** 0 outliers detected.
- **Interpretation:** The quantity received is consistent and expected, with no significant discrepancies.

### Summary of Findings

**Key Observations:**
The majority of outliers were identified in:
- Unit Cost
- PO_Amount
- PAID AMOUNT
- Invoice amounts (GROSS, NET, VAT)

**Potential Issues:**
These significant outliers could indicate potential issues such as:
- Overpricing or unusually high costs that may need renegotiation or review.
- Large or bulk orders that, while potentially legitimate, should be verified.
- Possible data entry errors or anomalies that require correction.


