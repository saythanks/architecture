Data Model
==========

Overview
--------

**Tables**

* users
* apps
* app-user
* accounts
* statements
* transactions
* pay_buckets


Fields are NOT NULLABLE unless otherwise specified.

### Users

| Type | Name | Notes |
|:--|:--|:--|
| UUID | id | Primary |
| UUID | account_id | FOREIGN |
| TEXT | name ||
| TEXT | email ||
| FLOAT | balance ||
| TEXT | stripe_cust_id | nullable |
| * | timestamps ||

### Apps

| Type | Name | Notes |
|:--|:--|:--|
| UUID | id | Primary |
| UUID | account_id | FOREIGN |
| TEXT | secret | App’s secret key|
| TEXT | name ||
| TEXT | url | nullable |
| TEXT | description | nullable |
| * | timestamps ||

### App-User

| Type | Name | Notes |
|:--|:--|:--|
| UUID | user_id | FOREIGN KEY|
| UUID | app_id | FOREIGN KEY|
| TEXT | name ||
| ENUM | role | [owner, user] |
| BOOLEAN | active | default=false |
| * | timestamps ||

### PayBuckets

This represents each concrete payable item, likely represented by a tip/unlock button on a webpage. This means a PayBucket could represent an article, a webpage, a single newsletter, etc.

| Type | Name | Notes |
|:--|:--|:--|
| UUID | id | PRIMARY |
| UUID | app_id | FOREIGN KEY |
| TEXT | display_name ||
| FLOAT | display_price | *may be different from real transaction price*|
| TEXT | permalink ||
| * | timestamps ||

### Accounts

Represents a pool of money held by a user or app with a balance, and which can be debited or credited. Very basic table now, but this is likely more extensible in the future.

| Type | Name | Notes |
|:--|:--|:--|
| UUID | id | PRIMARY |
| FLOAT | balance | *Not sure about this one. Would have to be updated transactionally. Which is more complexity.* |

### Statements

This represents period tallying of account balances, such that we can easily figure out account balances without having to sum the whole table.

| Type | Name | Notes |
|:--|:--|:--|
| UUID | id | PRIMARY |
| UUID | account_id | FOREIGN |
| UUID | last_tx_id | FOREIGN (just in case)|
| FLOAT | balance ||
| DATETIME | created_at ||	




### Transactions

This is this most important table, and it contains the most varied info. It needs to represent the following actions:

- User adds money to their balance (Cash => User) (Debit User)
- Users pays for item (User => App) (Credit User, Debit App)
- App owner cashes out to bank (App => Cash) (Credit App)

If the the source account is NULL, then we simply have a credit to the Dest account (i.e. a cash out).

If the dest account is NULL, then we have a debit to the source account (Adding money to the balance)

| Type | Name | Notes |
|:--|:--|:--|
| UUID | id | PRIMARY |
| UUID | source_account | FOREIGN_KEY, NULLABLE |
| UUID | dest_account | FOREIGN_KEY, NULLABLE |
| UUID | paybucket_id | FOREIGN_KEY, NULLABLE |
| FLOAT | amount ||
| TEXT | stripe_charge_id||
| JSONB | stripe_data| *rest of JSON charge object stripe returns*|
| * | timestamps ||
