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
