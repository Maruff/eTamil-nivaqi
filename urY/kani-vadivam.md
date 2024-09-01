Here's the redesigned table structure with the addition of a `company` table to store company details and configurations for the accounting module. The following design is written in PostgreSQL.

### 1. **Company Table**
   - **Table**: `company`
   - **Purpose**: Stores the company's basic details and configurations for the accounting module.
   - **Fields**:
     ```sql
     CREATE TABLE company (
         company_id SERIAL PRIMARY KEY,
         company_name VARCHAR(255) NOT NULL,
         address TEXT,
         email VARCHAR(255),
         phone VARCHAR(50),
         tax_id VARCHAR(50),
         currency VARCHAR(10) NOT NULL DEFAULT 'INR',
         fiscal_year_start DATE NOT NULL,
         fiscal_year_end DATE NOT NULL,
         created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
         updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
     );
     ```

### 2. **Partners Table (Consolidating Customers and Vendors)**
   - **Table**: `partners`
   - **Purpose**: Stores information about all business partners, including customers, vendors, and others.
   - **Fields**:
     ```sql
     CREATE TABLE partners (
         partner_id SERIAL PRIMARY KEY,
         partner_name VARCHAR(255) NOT NULL,
         partner_type VARCHAR(50) CHECK (partner_type IN ('Customer', 'Vendor', 'Both')) NOT NULL,
         email VARCHAR(255),
         phone VARCHAR(50),
         address TEXT,
         tax_id VARCHAR(50),
         is_active BOOLEAN DEFAULT TRUE,
         created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
         updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
     );
     ```

### 3. **Chart of Accounts (CoA)**
   - **Table**: `accounts`
   - **Purpose**: Defines all the accounts used in the system, organized by account types.
   - **Fields**:
     ```sql
     CREATE TABLE accounts (
         account_id SERIAL PRIMARY KEY,
         account_name VARCHAR(255) NOT NULL,
         account_code VARCHAR(50) UNIQUE NOT NULL,
         account_type VARCHAR(50) CHECK (account_type IN ('Asset', 'Liability', 'Equity', 'Revenue', 'Expense')) NOT NULL,
         parent_account_id INT REFERENCES accounts(account_id) ON DELETE SET NULL,
         is_active BOOLEAN DEFAULT TRUE,
         created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
         updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
     );
     ```

### 4. **Journal Entries**
   - **Table**: `journal_entries`
   - **Purpose**: Records the header information for journal entries.
   - **Fields**:
     ```sql
     CREATE TABLE journal_entries (
         journal_entry_id SERIAL PRIMARY KEY,
         entry_date DATE NOT NULL,
         reference TEXT,
         partner_id INT REFERENCES partners(partner_id) ON DELETE SET NULL,
         total_debit NUMERIC(15, 2) DEFAULT 0,
         total_credit NUMERIC(15, 2) DEFAULT 0,
         is_posted BOOLEAN DEFAULT FALSE,
         created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
         updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
     );
     ```

### 5. **Journal Items**
   - **Table**: `journal_items`
   - **Purpose**: Details each debit and credit transaction within a journal entry.
   - **Fields**:
     ```sql
     CREATE TABLE journal_items (
         journal_item_id SERIAL PRIMARY KEY,
         journal_entry_id INT REFERENCES journal_entries(journal_entry_id) ON DELETE CASCADE,
         account_id INT REFERENCES accounts(account_id) ON DELETE RESTRICT,
         debit_amount NUMERIC(15, 2) DEFAULT 0,
         credit_amount NUMERIC(15, 2) DEFAULT 0,
         description TEXT,
         partner_id INT REFERENCES partners(partner_id) ON DELETE SET NULL,
         created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
         updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
     );
     ```

### 6. **Payments**
   - **Table**: `payments`
   - **Purpose**: Manages payment transactions related to invoices, purchases, or general payments.
   - **Fields**:
     ```sql
     CREATE TABLE payments (
         payment_id SERIAL PRIMARY KEY,
         payment_date DATE NOT NULL,
         amount NUMERIC(15, 2) NOT NULL,
         payment_type VARCHAR(50) CHECK (payment_type IN ('Customer Payment', 'Vendor Payment')) NOT NULL,
         journal_entry_id INT REFERENCES journal_entries(journal_entry_id) ON DELETE CASCADE,
         partner_id INT REFERENCES partners(partner_id) ON DELETE SET NULL,
         created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
         updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
     );
     ```

### 7. **Invoices**
   - **Table**: `invoices`
   - **Purpose**: Records customer and vendor invoices.
   - **Fields**:
     ```sql
     CREATE TABLE invoices (
         invoice_id SERIAL PRIMARY KEY,
         invoice_date DATE NOT NULL,
         due_date DATE NOT NULL,
         total_amount NUMERIC(15, 2) NOT NULL,
         partner_id INT REFERENCES partners(partner_id) ON DELETE SET NULL,
         invoice_status VARCHAR(50) CHECK (invoice_status IN ('Draft', 'Posted', 'Paid')) NOT NULL,
         journal_entry_id INT REFERENCES journal_entries(journal_entry_id) ON DELETE CASCADE,
         created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
         updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
     );
     ```

### 8. **General Ledger (Ledger)**
   - **Table**: `general_ledger`
   - **Purpose**: Tracks the balance of each account by summarizing journal items.
   - **Fields**:
     ```sql
     CREATE TABLE general_ledger (
         ledger_id SERIAL PRIMARY KEY,
         account_id INT REFERENCES accounts(account_id) ON DELETE CASCADE,
         balance_date DATE NOT NULL,
         debit_total NUMERIC(15, 2) DEFAULT 0,
         credit_total NUMERIC(15, 2) DEFAULT 0,
         balance NUMERIC(15, 2) DEFAULT 0,
         created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
         updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
     );
     ```

### 9. **Taxes**
   - **Table**: `taxes`
   - **Purpose**: Manages different tax rates and details applied to transactions.
   - **Fields**:
     ```sql
     CREATE TABLE taxes (
         tax_id SERIAL PRIMARY KEY,
         tax_name VARCHAR(255) NOT NULL,
         tax_rate NUMERIC(5, 2) NOT NULL,
         account_id INT REFERENCES accounts(account_id) ON DELETE RESTRICT,
         created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
         updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
     );
     ```

### Summary of the PostgreSQL Design
1. **`company`**: Stores company details and configuration for the accounting module.
2. **`partners`**: Consolidates all entities involved in transactions (customers, vendors, etc.).
3. **`accounts`**: The Chart of Accounts, organizing all financial categories.
4. **`journal_entries`**: The header for financial entries, summarizing transactions.
5. **`journal_items`**: The detailed transactions within a journal entry.
6. **`payments`**: Manages the financial payments.
7. **`invoices`**: Stores both customer and vendor invoices.
8. **`general_ledger`**: Tracks the balance for each account.
9. **`taxes`**: Manages tax-related details.

This structure is simplified and follows Odoo's design principles but adapted for ease of use in a PostgreSQL environment.
