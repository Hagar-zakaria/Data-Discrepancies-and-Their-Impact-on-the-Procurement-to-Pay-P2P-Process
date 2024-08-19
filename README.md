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


# Step 5;  Compare Purchase Requisitions with Purchase Orders

```python
def compare_columns(df1, df2, key, col1, col2):
    # Merge the two dataframes on the key column to align the data
    merged_df = pd.merge(df1[[key, col1]], df2[[key, col2]], on=key, how='inner', suffixes=('_df1', '_df2'))

    # Find rows where the values in the specified columns are not equal
    discrepancies = merged_df[merged_df[f'{col1}_df1'] != merged_df[f'{col2}_df2']]

    return discrepancies

# Remove any leading or trailing spaces from column names
purchase_requisition_df.columns = purchase_requisition_df.columns.str.strip()
purchase_orders_df.columns = purchase_orders_df.columns.str.strip()

print("Columns in purchase_requisition_df:", purchase_requisition_df.columns)
print("Columns in purchase_orders_df:", purchase_orders_df.columns)

if 'QTY' in purchase_requisition_df.columns and 'QTY' in purchase_orders_df.columns:
    print("Column 'QTY' exists in both dataframes.")
else:
    print("Column 'QTY' does not exist in one or both dataframes.")

# Compare Purchase Requisitions with Purchase Orders
pr_vs_po_discrepancies = compare_columns(purchase_requisition_df, purchase_orders_df, key='PR_Number', col1='QTY', col2='QTY')
pr_vs_po_discrepancies.to_csv('pr_vs_po_discrepancies.csv', index=False)
print(f"Discrepancies between Purchase Requisitions and Purchase Orders saved to 'pr_vs_po_discrepancies.csv'")

# Merge PR with PO and check for discrepancies
pr_po_merge = pd.merge(purchase_requisition_df, purchase_orders_df, on='PR_Number', how='left', suffixes=('_PR', '_PO'))

# Column to compare
column_to_compare = 'QTY'

# Ensure both columns are numeric
pr_po_merge[f'{column_to_compare}_PR'] = pd.to_numeric(pr_po_merge[f'{column_to_compare}_PR'], errors='coerce')
pr_po_merge[f'{column_to_compare}_PO'] = pd.to_numeric(pr_po_merge[f'{column_to_compare}_PO'], errors='coerce')

# Calculate the difference
pr_po_merge[f'{column_to_compare}_Difference'] = pr_po_merge[f'{column_to_compare}_PR'] - pr_po_merge[f'{column_to_compare}_PO']

# Filter rows where there is a discrepancy in the QTY column
pr_po_discrepancies = pr_po_merge[
    pr_po_merge[f'{column_to_compare}_Difference'].notna() &
    pr_po_merge[f'{column_to_compare}_Difference'].ne(0)
]

# Handle 'Unknown' values
unknown_discrepancies = pr_po_discrepancies[
    (pr_po_discrepancies['PR_Number'] == 'Unknown') |
    (pr_po_discrepancies[f'{column_to_compare}_PR'] == 0) |
    (pr_po_discrepancies[f'{column_to_compare}_PO'] == 0)
]

# Save discrepancies to CSV
pr_po_discrepancies.to_csv('pr_po_discrepancies.csv', index=False)
unknown_discrepancies.to_csv('unknown_discrepancies.csv', index=False)

# Display discrepancies
print("All Discrepancies:")
print(pr_po_discrepancies)

print("\nDiscrepancies involving 'Unknown' values:")
print(unknown_discrepancies)

# Visualization of all discrepancies
plt.figure(figsize=(10, 8))
sns.scatterplot(data=pr_po_discrepancies, x=f'{column_to_compare}_PR', y=f'{column_to_compare}_PO', hue='PR_Number', palette='viridis')
plt.plot([pr_po_discrepancies[f'{column_to_compare}_PR'].min(), pr_po_discrepancies[f'{column_to_compare}_PR'].max()],
         [pr_po_discrepancies[f'{column_to_compare}_PR'].min(), pr_po_discrepancies[f'{column_to_compare}_PR'].max()],
         'r--', label='Expected (PR = PO)')
plt.title('Discrepancies between PR and PO Quantities')
plt.xlabel('QTY in PR')
plt.ylabel('QTY in PO')
plt.legend()
plt.show()

# Visualization of discrepancies involving 'Unknown' values
plt.figure(figsize=(10, 8))
sns.scatterplot(data=unknown_discrepancies, x=f'{column_to_compare}_PR', y=f'{column_to_compare}_PO', hue='PR_Number', palette='coolwarm')
plt.plot([unknown_discrepancies[f'{column_to_compare}_PR'].min(), unknown_discrepancies[f'{column_to_compare}_PR'].max()],
         [unknown_discrepancies[f'{column_to_compare}_PR'].min(), unknown_discrepancies[f'{column_to_compare}_PR'].max()],
         'r--', label='Expected (PR = PO)')
plt.title('Discrepancies Involving Unknown Values between PR and PO Quantities')
plt.xlabel('QTY in PR')
plt.ylabel('QTY in PO')
plt.legend()
plt.show()
```

![image](https://github.com/user-attachments/assets/2721125f-761c-4ee4-b2e1-3cc76b2792dd)

![download - 2024-08-19T133942 495](https://github.com/user-attachments/assets/fa1a9048-8601-41f9-b66e-290fb09374c6)


## Discrepancies Analytics between Purchase Requisitions and Purchase Orders

### Under-Delivery (Negative QTY_Difference):

- **PO Number GG0439107 (ITEM Number 8000570):** Ordered 3 units but received 0 units (-3 QTY_Difference).
- **PO Number FF1039100 (ITEM Number 8003104):** Ordered 11 units but received 0 units (-11 QTY_Difference).
- **PO Number FF0839100 (ITEM Number 8001136):** Ordered 15 units but received 0 units (-15 QTY_Difference).
- **PO Number HH0839093 (ITEM Number 8000075):** Ordered 15 units but received 0 units (-15 QTY_Difference).
- **PO Number DD0939107 (ITEM Number 8000624):** Ordered 25 units but received 0 units (-25 QTY_Difference).
- **PO Number AA0139107 (ITEM Number 8001426):** Ordered 22 units but received 0 units (-22 QTY_Difference).
- **PO Number GG1439128 (ITEM Number 8000832):** Ordered 7 units but received 0 units (-7 QTY_Difference).
- **PO Number EE0339128 (ITEM Number 8001801):** Ordered 17 units but received 0 units (-17 QTY_Difference).
- **PO Number HH0639121 (ITEM Number 8000683):** Ordered 23 units but received 0 units (-23 QTY_Difference).
- **PO Number HH1639107 (ITEM Number 8001468):** Ordered 11 units but received 0 units (-11 QTY_Difference).

### What This Means:
These orders had significant under-delivery, where no items were received even though they were ordered. This could point to issues like incomplete deliveries, errors in processing the orders, or mistakes in recording the items received.

### Exact Matches (QTY_Difference is Zero):
There are no exact matches in the dataset, meaning that for all purchase orders, the received quantities did not exactly match the ordered quantities.

### Potential Recording Issues:
The consistent negative QTY_Difference suggests there may be problems with how quantities were recorded, such as incorrect data entry or loss of information during data transfer.

### Specific Cases to Highlight:
- **PO Number GG0439107** shows multiple instances of under-delivery, which may indicate a recurring problem with this item or supplier.
- **PO Number DD0939107** also has a significant under-delivery, suggesting there might be a bigger issue in the supply chain or ordering process.

### Summary of Observations:
- **Under-Deliveries:** All cases observed are under-deliveries, suggesting there might be serious challenges in ensuring that the quantities ordered are actually delivered.
- **Recording Discrepancies:** The negative QTY_Difference values across all entries point to potential errors in data recording or processing.
- **Operational Impact:** These discrepancies could lead to stock shortages, delays in operations, or financial issues if not properly addressed.


# Step 6: Analyzing Discrepancies Between Invoices and Payments

### Function to Compare Columns from Purchase Requisitions and Purchase Orders

```python
def compare_requisitions_with_orders(df1, df2, key, col1, col2):
    # Merge the dataframes on the specified key (e.g., ITEM Number) to align the data
    merged_df = pd.merge(df1[[key, col1]], df2[[key, col2]], on=key, how='inner', suffixes=('_req', '_order'))

    # Identify discrepancies where the quantity ordered does not match the quantity requisitioned
    discrepancies = merged_df[merged_df[f'{col1}_req'] != merged_df[f'{col2}_order']]

    return discrepancies
```

### Compare QTY between Purchase Requisitions and Purchase Orders

```python
pr_vs_po_discrepancies = compare_requisitions_with_orders(
    purchase_requisition_df,
    purchase_orders_df,
    key='ITEM Number',
    col1='QTY',
    col2='QTY'
)
```


### Save the Discrepancies to a CSV File for Further Analysis

```python
pr_vs_po_discrepancies.to_csv('pr_vs_po_discrepancies.csv', index=False)
print(f"Discrepancies between Purchase Requisitions and Purchase Orders saved to 'pr_vs_po_discrepancies.csv'")
```

### Visualize the Discrepancies

```python
plt.figure(figsize=(12, 8))
sns.scatterplot(x='QTY_req', y='QTY_order', data=pr_vs_po_discrepancies, hue='ITEM Number', palette='coolwarm')
plt.title('Discrepancies between Requisitioned Quantity and Ordered Quantity')
plt.xlabel('Quantity Requisitioned')
plt.ylabel('Quantity Ordered')
plt.axline((0, 0), slope=1, color='red', linestyle='--', label='Expected (QTY_req = QTY_order)')
plt.legend()
plt.show()
```

![download - 2024-08-19T140035 488](https://github.com/user-attachments/assets/4fc5e98b-4eb2-4886-8aa2-9eaf925ece8f)


### Key Observations:

- **Multiple Discrepancies for the Same ITEM Number:**
  - Some ITEM numbers show discrepancies with different quantities, suggesting possible repeated orders or adjustments. For example:
    - **ITEM Number 8003153:** 20 units were requisitioned, but only 11 were ordered, and in another instance, 11 units were requisitioned, but 20 were ordered.
    - **ITEM Number 8001326:** Has multiple discrepancies with varying quantities, such as 16 requisitioned but only 10 or 17 ordered.

- **Significant Discrepancies:**
  - Some ITEM numbers show large differences between the quantities requisitioned and the quantities ordered, which may need further investigation. Examples include:
    - **ITEM Number 8003130:** 3 units were requisitioned, but 15 were ordered.
    - **ITEM Number 8001815:** 23 units were requisitioned, but only 4 were ordered.

- **Unknown ITEM Numbers:**
  - There are several entries with "Unknown" ITEM numbers, which likely represent cases where data is missing or incorrectly entered. This can complicate reconciliation and should be addressed to improve data accuracy.

- **Frequent Small Discrepancies:**
  - Some ITEM numbers have small differences in quantities, which might occur due to rounding errors, partial deliveries, or minor adjustments in orders.
  
- **Example Cases:**
  - **ITEM Number 8000891:** There are discrepancies where 21 units were requisitioned but only 20 were ordered, and vice versa.
  - **ITEM Number 8000241:** There is a case where 1 unit was requisitioned, but 12 were ordered, indicating a possible data entry error or miscommunication.

 
  # Step 7 ; Discrepancies between Invoice Dates and Payment Dates

```python
# Merge the dataframes on the specified key (e.g., SUPPLIER_NAME or INVNO)
merged_df = pd.merge(invoices_received_df[['INVNO', 'INVDT_DATE']], payments_df[['INVNO', 'PAYMENT DATE']], on='INVNO', how='inner')

# Inspect the merged DataFrame
print(merged_df.head())

# Function to compare expected payment dates (Invoice Date) with actual payment dates
def compare_payment_dates(df1, df2, key, col1, col2):
    # Merge the dataframes on the specified key and include 'SUPPLIER_NAME'
    merged_df = pd.merge(df1[[key, 'SUPPLIER_NAME', col1]], df2[[key, 'SUPPLIER_NAME', col2]], on=[key, 'SUPPLIER_NAME'], how='inner')

    # Identify discrepancies where the invoice date does not match the actual payment date
    discrepancies = merged_df[merged_df[col1] != merged_df[col2]]

    return discrepancies

# Compare Invoice Dates with Payment Dates
invoice_vs_payment_discrepancies = compare_payment_dates(
    invoices_received_df,  # Invoices dataframe
    payments_df,           # Payments dataframe
    key='INVNO',           # Assuming invoices and payments can be matched on 'INVNO' (Invoice Number)
    col1='INVDT_DATE',     # Invoice date in invoices dataframe
    col2='PAYMENT DATE'    # Actual payment date in payments dataframe
)

# Save the discrepancies to a CSV file
invoice_vs_payment_discrepancies.to_csv('invoice_vs_payment_discrepancies_Dates.csv', index=False)
print(f"Discrepancies between Invoice Dates and Payment Dates saved to 'invoice_vs_payment_discrepancies.csv'")

# Visualize the discrepancies
plt.figure(figsize=(12, 8))
sns.histplot(data=invoice_vs_payment_discrepancies, x='INVDT_DATE', y='PAYMENT DATE', hue='SUPPLIER_NAME')
plt.title('Discrepancies between Invoice Dates and Payment Dates')
plt.xlabel('Invoice Date')
plt.ylabel('Payment Date')
plt.show()
```

![download - 2024-08-19T145645 625](https://github.com/user-attachments/assets/59c14648-324f-4edc-8a9f-cf0320c48ea6)

### Analytics of Discrepancies between Invoice Dates and Payment Dates

#### Wide Range of Payment Delays
The gaps between when invoices were issued and when payments were made vary widely. Some payments were made on time, while others were delayed by a few days to several weeks.

#### Consistent Late Payments
Certain suppliers, like **BRIGGS EQUIPMENT UK LTD** and **R F AMIES(KIDDERMINSTER)LTD**, consistently experienced payment delays across multiple invoices. This might indicate ongoing issues in the payment process for these suppliers.

#### Suppliers with Timely Payments
A few suppliers, such as **WORTHINGTON & GRAHAM LTD** and **NIFTYLIFT LTD.**, have payments that closely align with their invoice dates, indicating that these suppliers were paid on time.

#### Clusters of Discrepancies
There are noticeable clusters of payment discrepancies, especially around certain dates in January 2018. This suggests that payment processing was particularly problematic during these periods.

### Significant Delays
For some suppliers, like **FILPLASTICS(UK)LTD** and **MACE INDUSTRIES LTD**, payments were significantly delayed, sometimes by more than 20 days. These delays could harm supplier relationships and potentially lead to late fees or penalties.

#### Supplier-Specific Issues
Some suppliers, like **BRIGGS EQUIPMENT UK LTD**, show discrepancies across multiple invoices, which may indicate recurring issues with how their invoices are processed or prioritized.

#### Unexpected Early Payments
In some cases, payments were made either before or very close to the invoice date, such as with **WORTHINGTON & GRAHAM LTD** (paid on the same day as invoicing). This could suggest either preferential treatment or errors in the payment processing system.

#### Unidentified Patterns
The heatmap reveals a few blank spots where no payments were made on expected dates, such as on January 7th. These gaps might be worth investigating to see if they represent missed payments or data entry issues.


# Step 8; Discrepancies Between PO Date and Invoice Date

To ensure that our procurement and invoicing processes are aligned, it's crucial to check if there are any discrepancies between the Purchase Order (PO) date and the Invoice Date. A significant discrepancy where an invoice date is before the purchase order date may indicate errors in the documentation process.

### Code to Detect Discrepancies

The following code is used to identify discrepancies where the invoice date is earlier than the purchase order date:

```python
def check_po_vs_invoice_date(po_df, inv_df, po_date_col, inv_date_col, po_number_col, inv_po_number_col):
    # Convert the date columns to datetime format
    po_df[po_date_col] = pd.to_datetime(po_df[po_date_col])
    inv_df[inv_date_col] = pd.to_datetime(inv_df[inv_date_col])

    # Merge the DataFrames on the Purchase Order Number
    merged_df = pd.merge(po_df[[po_number_col, po_date_col]],
                         inv_df[[inv_po_number_col, inv_date_col]],
                         left_on=po_number_col,
                         right_on=inv_po_number_col,
                         how='inner')

    # Identify rows where the invoice date is before the purchase order date
    discrepancies = merged_df[merged_df[inv_date_col] < merged_df[po_date_col]]

    return discrepancies

# Checking for discrepancies
po_vs_inv_date_discrepancies = check_po_vs_invoice_date(
    purchase_orders_df,
    invoices_received_df,
    po_date_col='PO_Date',
    inv_date_col='INVDT_DATE',
    po_number_col='PO_Number',
    inv_po_number_col='ORDER_NO'
)

# Save the discrepancies to a CSV file
po_vs_inv_date_discrepancies.to_csv('po_vs_inv_date_discrepancies.csv', index=False)
print(f"Discrepancies where Invoice Date is before Purchase Order Date saved to 'po_vs_inv_date_discrepancies.csv'")

# Display the discrepancies DataFrame
po_vs_inv_date_discrepancies.head()
```

![image](https://github.com/user-attachments/assets/3e3ea55d-244b-42fb-a6a0-cde2e22e1b06)


The table above shows instances where the invoice date is before the purchase order date, which is typically not expected in a well-functioning procurement system. These discrepancies might be due to data entry errors, or it could indicate a problem in the invoicing process where invoices are issued before formal purchase orders are completed.

# Step 9 ; Identifying and Analyzing Discrepancies Between Invoice Dates and Purchase Order Dates

```python
# Convert INVDT_DATE and PO_Date to datetime format
merged_df['INVDT_DATE'] = pd.to_datetime(merged_df['INVDT_DATE'])
merged_df['PO_Date'] = pd.to_datetime(merged_df['PO_Date'])

# Identify discrepancies where INVDT_DATE is earlier than PO_Date
earlier_invoices = merged_df[merged_df['INVDT_DATE'] < merged_df['PO_Date']]

# Group by supplier to find which suppliers have the most discrepancies
discrepancies_by_supplier = earlier_invoices['SUPPLIER_NAME_x'].value_counts()

# Count the total number of orders with invoices earlier than the PO date
count_earlier_invoices = earlier_invoices.shape[0]

# Display the results
print("Suppliers with the most discrepancies:")
print(discrepancies_by_supplier)
print(f"Total orders with invoices earlier than the PO date: {count_earlier_invoices}")

# Optionally save the results to CSV files for further analysis
discrepancies_by_supplier.to_csv('discrepancies_by_supplier.csv', index=True)
earlier_invoices.to_csv('earlier_invoices_discrepancies.csv', index=False)
```
![download - 2024-08-19T155459 901](https://github.com/user-attachments/assets/21768121-0682-42b3-af45-e923d6db103d)

### Total Discrepancies:
- 27 orders had invoices dated earlier than the purchase orders.

### Top Suppliers with Discrepancies:
- **BRIGGS EQUIPMENT UK LTD**: 4 discrepancies
- **GRAFTERS**: 2 discrepancies
- **ALLBUILD PRODUCTS**: 2 discrepancies

### Other Suppliers with Discrepancies (Each with 1 discrepancy):
- ROUDEN PIPETEK
- JOSEPH PARR (ALCO) LTD.
- HILCREST DESIGN LTD
- SIMMONS OF STAFFORD
- SOUNDSORBA LTD
- SALEPOINT LTD
- TAYLOR MAXWELL & CO LTD
- LYRECO UK LIMITED
- DOUGLAS SCOTT
- NORMAN JAMIESON LTD
- MEDLAND SANDERS AND TWOSE LIMITED
- ERNEST BENNETT (SHEFFIELD) LTD.
- ELB PARTNERS LTD
- VIBROPLANT PLC
- THE SPECTRA GROUP LTD
- BLUE DIAMOND TECHNOLOGIES LTD
- CHEMAIDE LTD
- RED FUNNEL
- A C YULE GROUP


### Step 10 : Analyzing Unit Cost Discrepancies

```python
import pandas as pd

# Calculate the expected unit cost (e.g., average cost for each item)
expected_cost_df = purchase_orders_df.groupby('ITEM Number')['Unit Cost'].mean().reset_index()
expected_cost_df.columns = ['ITEM Number', 'Expected Unit Cost']

# Merge the expected costs with the original purchase orders dataframe
merged_cost_df = pd.merge(purchase_orders_df, expected_cost_df, on='ITEM Number', how='left')

# Identify discrepancies where the actual unit cost deviates significantly from the expected cost
cost_discrepancies = merged_cost_df[merged_cost_df['Unit Cost'] != merged_cost_df['Expected Unit Cost']]

# Display the cost discrepancies
print("Cost Discrepancies:")
print(cost_discrepancies)

# Save the cost discrepancies to a CSV file
cost_discrepancies.to_csv('cost_discrepancies.csv', index=False)

# Group the discrepancies by supplier
cost_discrepancies_by_supplier = cost_discrepancies.groupby('SUPPLIER_NAME').agg({
    'ITEM Number': 'count',
    'Unit Cost': 'sum',
    'Expected Unit Cost': 'sum'
}).reset_index()
cost_discrepancies_by_supplier.columns = ['Supplier Name', 'Number of Discrepancies', 'Total Actual Cost', 'Total Expected Cost']

# Save the discrepancies by supplier to a CSV file
cost_discrepancies_by_supplier.to_csv('cost_discrepancies_by_supplier.csv', index=False)
```

![image](https://github.com/user-attachments/assets/12f30f3a-c950-4ef4-9c4d-e5d2a3d523f4)

