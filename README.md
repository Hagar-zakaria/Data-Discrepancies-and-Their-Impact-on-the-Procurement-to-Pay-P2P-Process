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

# Step 3: Analyze Correlations Among Outliers

In this section, we explore the correlations between various financial components such as purchase order amounts and invoice totals (GROSS, NET, VAT). Understanding these relationships is crucial for identifying potential discrepancies and ensuring that what is ordered aligns accurately with what is billed. A weak correlation in these areas can highlight areas where overbilling or misalignment in procurement processes may be occurring, warranting further investigation.

### Cross-Dataset Correlation Analysis

```python
# Merge the datasets on a common key or align them (for this example, we'll align them directly)
merged_df = pd.concat([
    purchase_orders_df[['PO_Amount']].rename(columns={'PO_Amount': 'Purchase Order Amount'}),
    invoices_received_df[['GROSS', 'NET', 'VAT']]
], axis=1).dropna()

# Calculate correlation matrix
corr_cross = merged_df.corr()

# Visualize correlation matrix
plt.figure(figsize=(12, 10))
sns.heatmap(corr_cross, annot=True, cmap='coolwarm', vmin=-1, vmax=1)
plt.title('Cross-Dataset Correlation Matrix: PO_Amount vs Invoice Amounts')
plt.show()

# Display correlation matrix
print(corr_cross)
```
![download - 2024-08-19T122400 903](https://github.com/user-attachments/assets/f15751c6-abca-45bd-b27c-86702f7ffec2)


## Cross-Dataset Correlation Results 

### Purchase Order Amount and GROSS

- **Correlation Coefficient:** 0.010
- **Interpretation:** The link between the purchase order amounts and the gross amounts on the invoices is very weak. This suggests that what’s billed doesn’t closely match what was ordered, indicating possible discrepancies between the two.

### Purchase Order Amount and NET

- **Correlation Coefficient:** 0.010
- **Interpretation:** Similarly, the weak link between purchase order amounts and net amounts on invoices indicates potential inconsistencies in pricing or quantities. What was ordered and what was invoiced might not be aligning well.

### Purchase Order Amount and VAT

- **Correlation Coefficient:** 0.010
- **Interpretation:** The weak correlation here suggests that VAT on invoices doesn’t align well with the amounts ordered. This could point to issues like overbilling or gaps between procurement and accounting processes.

### GROSS, NET, and VAT

- **Correlation Coefficients:**
  - **GROSS and NET:** 0.999995
  - **GROSS and VAT:** 0.999832
  - **NET and VAT:** 0.999768
- **Interpretation:** These three components (GROSS, NET, VAT) are almost perfectly aligned with each other on the invoices, which is expected and good from an accounting perspective. However, this strong alignment contrasts with their weak link to the purchase order amounts.

### Summary and Recommendations

- **Discrepancies Between Purchase Orders and Invoices:** The weak correlations between what’s ordered and what’s billed (GROSS, NET, VAT) suggest potential problems. These might include:
  - **Overbilling:** Invoices might be higher than the corresponding purchase orders.
  - **Procurement and Billing Misalignment:** Differences in what was ordered versus what was billed could be due to errors or inefficiencies in the procurement process.

**In simple terms, the data shows that there might be significant issues between what was supposed to be purchased and what was actually billed. These discrepancies should be investigated to ensure accuracy and prevent overcharges.**

# Step 4: Analysis of Discrepancies between Vendor Bank Details and Payments

```python
# Compare Vendor Bank Details with Payments
merged_vendor_payments = pd.merge(vendor_df[['SUPPLIER_NAME', 'PAYEE_NAME', 'BANK_ACCT']],
                                  payments_df[['SUPPLIER_NAME', 'PAID AMOUNT', 'Bank Details']],
                                  on='SUPPLIER_NAME', how='inner')

bank_discrepancies = merged_vendor_payments[merged_vendor_payments['BANK_ACCT'] != merged_vendor_payments['Bank Details']]
bank_discrepancies.to_csv('vendor_vs_payments_bank_discrepancies.csv', index=False)
print(f"Discrepancies in vendor bank details saved to 'vendor_vs_payments_bank_discrepancies.csv'")
```

### Vendor vs Payments Discrepancies Visualization
# Vendor vs Payments Discrepancies

```python
plt.figure(figsize=(10, 6))
sns.countplot(y='SUPPLIER_NAME', data=bank_discrepancies)
plt.title('Count of Bank Account Discrepancies by Supplier')
plt.xlabel('Number of Discrepancies')
plt.ylabel('Supplier Name')
plt.show()
```
![download - 2024-08-19T123747 527](https://github.com/user-attachments/assets/f9447eba-ac9c-4711-8553-2b61b6b2f8e5)

### Visualizing Amounts with Bank Discrepancies

```python
plt.figure(figsize=(12, 8))
sns.barplot(x='PAID AMOUNT', y='SUPPLIER_NAME', data=bank_discrepancies, ci=None)
plt.title('Total Paid Amount with Bank Account Discrepancies by Supplier')
plt.xlabel('Total Paid Amount')
plt.ylabel('Supplier Name')
plt.show()
```
![download - 2024-08-19T123832 591](https://github.com/user-attachments/assets/5e340d99-b1e8-42e0-ab22-85c995f6c4df)

## Results of Discrepancies between Vendor Bank Details and Payments

Here’s a straightforward explanation of the payment discrepancies we found, including the key details:

### BRIGGS EQUIPMENT UK LTD

- **Number of Issues:** 3
- **Recorded Bank Account:** 08-67-30/73313715
- **Bank Account Used for Payment:** 33-69-90/78912345
- **Total Amount Paid:**
  - £16,972.67
  - £82,194.32
  - £5,921.18

**Why It’s Concerning:** A total of £105,088.17 was sent to a different bank account than what we have on record for this supplier. This means the supplier might not get paid, or the money could end up in the wrong account altogether.

### Other Suppliers with Issues

#### DAVLYN PROPERTIES (WIGAN) LTD
- **Recorded Bank Account:** 17-67-88/73276308
- **Bank Account Used for Payment:** 33-69-90/78912345
- **Total Amount Paid:** £3,730.56

#### JB TRANSPORT
- **Recorded Bank Account:** 37-85-27/81781216
- **Bank Account Used for Payment:** 33-69-90/78912345
- **Total Amount Paid:** £2,185.00

#### JIS (EUROPE) LIMITED
- **Recorded Bank Account:** 36-75-66/13731168
- **Bank Account Used for Payment:** 33-69-90/78912345
- **Total Amount Paid:** £2,549.61

#### MACE INDUSTRIES LTD
- **Recorded Bank Account:** 47-12-09/19703100
- **Bank Account Used for Payment:** 33-69-90/78912345
- **Total Amount Paid:** £127,565.52

#### PACKEXE LTD
- **Recorded Bank Account:** 66-73-61/19897612
- **Bank Account Used for Payment:** 33-69-90/78912345
- **Total Amount Paid:** £13,428.36

#### PRAMAC (UK) LTD
- **Recorded Bank Account:** 60-73-63/79275624
- **Bank Account Used for Payment:** 33-69-90/78912345
- **Total Amount Paid:** £13,079.52

#### PROGRESSIVE PRODUCTS LTD
- **Recorded Bank Account:** 58-88-41/84511844
- **Bank Account Used for Payment:** 33-69-90/78912345
- **Total Amount Paid:** £43,961.28

#### T W JONES
- **Recorded Bank Account:** 35-29-94/64071985
- **Bank Account Used for Payment:** 33-69-90/78912345
- **Total Amount Paid:** £27,703.20

### Why These Issues Matter

- **Mismatched Accounts:** We found 12 cases where payments were made to the wrong bank account (33-69-90/78912345) instead of the correct one listed for each supplier.
- **Total Money at Risk:** The total amount paid using incorrect bank details is significant, which could lead to a major financial loss if the payments don’t reach the right accounts.

### What We Need to Do Now

- **Verify Payments:** We should immediately check with the suppliers to confirm whether they received the payments and update their bank details if needed.
- **Correct Errors:** Fix any mistakes in our system to make sure future payments go to the correct accounts.

# Step 5: Analyzing Discrepancies between Purchase Orders and Goods Receipt Notes

It's crucial to ensure that the quantity of goods ordered matches the quantity received. Discrepancies between purchase orders and goods receipt notes can lead to inventory inaccuracies, operational delays, and financial discrepancies. In this step, we analyze the differences between the quantities ordered and the quantities received by comparing the Purchase Orders (POs) with the corresponding Goods Receipt Notes (GRNs). Through this analysis, we aim to identify patterns of over-delivery, under-delivery, and potential recording errors that could impact the efficiency and accuracy of the P2P process.

### Custom Function to Compare Columns

```python
# Custom function to compare two columns from different datasets
def compare_columns(df1, df2, key, col1, col2):
    # Merge the two dataframes on the specified key
    merged_df = pd.merge(df1[[key, col1]], df2[[key, col2]], on=key, how='inner')

    # Identify where the columns do not match
    discrepancies = merged_df[merged_df[col1] != merged_df[col2]]

    return discrepancies

# Compare Purchase Orders with Goods Receipt Notes
po_vs_grn_discrepancies = compare_columns(purchase_orders_df, goods_receipt_notes_df, key='PO_Number', col1='QTY', col2='QTY Received')
po_vs_grn_discrepancies.to_csv('po_vs_grn_discrepancies.csv', index=False)
print(f"Discrepancies between Purchase Orders and Goods Receipt Notes saved to 'po_vs_grn_discrepancies.csv'")
```


### Visualizing Discrepancies for Purchase Orders vs. Goods Receipt Notes

```python
plt.figure(figsize=(12, 8))
sns.scatterplot(x='QTY', y='QTY Received', data=po_vs_grn_discrepancies, hue='PO_Number', palette='viridis')
```

### Highlighting the discrepancies

```python
plt.title('Discrepancies between Ordered Quantity and Received Quantity')
plt.xlabel('Quantity Ordered')
plt.ylabel('Quantity Received')
plt.axline((0, 0), slope=1, color='red', linestyle='--', label='Expected (QTY = QTY Received)')
plt.legend()
plt.show()
```

![download - 2024-08-19T131054 916](https://github.com/user-attachments/assets/8e440ab7-7d52-4bf8-bad4-d7add8164c3c)


### Analytics of Discrepancies between Purchase Orders vs. Goods Receipt Notes

#### What the Scatter Plot Shows:
The scatter plot compares the number of items that were ordered to the number of items that were actually received for various purchase orders. Each dot represents a specific purchase order, and the position of the dot in relation to the red dashed line helps us understand whether more or fewer items were received than ordered.

#### Key Observations:

**Over-Delivery (Dots Above the Red Line):**

- **PO Number HH0839093:** Ordered 15 units, but received 19 units.
- **PO Number AA1239093:** Ordered 20 units, but received 25 units.
- **PO Number GG0539100:** Ordered 23 units, but received 29 units.
- **PO Number EE0339086:** Ordered 25 units, but received 31 units.

**What This Means:** These orders received more items than were ordered. This could be due to errors in ordering, suppliers delivering more than requested, or mistakes in recording the received quantities.

**Under-Delivery (Dots Below the Red Line):**

- **PO Number EE0539100:** Ordered 18 units, but received only 9 units.
- **PO Number FF0139114:** Ordered 21 units, but received only 11 units.
- **PO Number BB0139100:** Ordered 21 units, but received only 16 units.
- **PO Number AA1039107:** Ordered 15 units, but received only 8 units.

**What This Means:** These orders received fewer items than were ordered. This could be because of shortages during delivery, damage to items during transit, or mistakes in recording what was received.

**Exact Matches (Dots on the Red Line):**
The red line represents perfect matches where the number of items ordered equals the number of items received. However, in this analysis, there are no dots on the line, meaning that every order had some level of discrepancy.

#### Specific Cases to Highlight:

- **PO Number GG1239107:** This purchase order appears twice with conflicting information:
  - One record shows 17 units ordered and 11 received.
  - Another record shows 11 units ordered and 17 received.
  
**What This Suggests:** There might be an issue where quantities were swapped or recorded incorrectly.

**PO Numbers with High Variability:**

- **EE0339086 (25 ordered, 31 received)** and **HH0339100 (25 ordered, 31 received)** show significant over-deliveries.
- **FF0139114** and **BB0139100** consistently show under-deliveries, which could point to a recurring problem in the ordering or receiving process.

#### Summary of Observations:
- **Over-Deliveries:** Some orders received more items than were ordered, which might lead to waste or confusion.
- **Under-Deliveries:** Some orders received fewer items than ordered, potentially causing shortages or delays in operations.
- **Inconsistent Recording:** The case of **GG1239107** highlights possible mistakes in how data is recorded, which could lead to inaccuracies in inventory.

Overall, this analysis shows that there are several issues with how quantities are ordered and received, and these discrepancies could lead to inefficiencies or errors in inventory management.


