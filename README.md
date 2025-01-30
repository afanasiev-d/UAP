# UAP Indexer

### Overview

The **UAP Indexer** is a Python-based tool that processes transactions from the TRON blockchain. It specifically focuses on **Update Account Permission (UAP)** transactions, as well as **Delegate/UnDelegate Resource Contracts**, and **Reclaim** transactions. This tool enables the efficient extraction of these transactions and stores them in CSV files for further analysis. Additionally, the tool provides various utilities for updating specific fields based on new block data.

### Features

- **Extract UAP Transactions**: The indexer identifies **Update Account Permission (UAP)** transactions from blocks and stores them in CSV files.
- **Extract Delegate and Reclaim Transactions**: The indexer also captures **DelegateResourceContract** and **UnDelegateResourceContract** (Reclaim) transactions.
- **TRON Address Conversion**: Converts addresses from hexadecimal format to TRON base58 format for easier identification.
- **CSV Storage**: Outputs extracted transaction data to CSV files, including important fields like block number, owner and receiver addresses, contract types, and transaction metadata.
- **Field Updates**: Automatically increments specific fields like `stake_count`, `delegate_reclaim_after_uap_count`, and `additional_uap` for transactions matching certain conditions.
- **Parallel Block Processing**: Processes blocks sequentially in parallel for optimized performance.

### Requirements

- **Python 3.x**
- **pandas**: For data manipulation and CSV handling.
- **requests**: To interact with the TRON blockchain API.
- **psutil**: For monitoring system resources (CPU, memory) and managing parallel execution.
- **tqdm**: For progress bars in loops.

Install the dependencies using pip:

```bash
pip install pandas requests psutil tqdm
```

### Functionality

1. **Extracting Transactions**:
   - The main task is to fetch transactions from the TRON blockchain (via TRON API or node) and extract those related to **Update Account Permission (UAP)**, **Delegate/UnDelegate Resource Contracts**, and **Reclaim** actions.
   - The function `get_block_transactions(block_number)` is responsible for fetching the block data for a given block number.
   - The function `save_to_csv(transactions, output_dir, block_number)` processes and saves UAP transactions to CSV.

2. **CSV Storage**:
   - The `save_to_csv()` function creates a CSV file per block. It organizes the data into monthly CSV files based on the timestamp of the transactions.
   - The output CSV file contains fields like `txID`, `signature`, `owner_address`, `block_timestamp`, and TRON converted addresses (`owner_address_1_tron`, `owner_address_2_tron`).

3. **Updating Fields**:
   - For **Delegate/UnDelegate Resource Contracts**, the indexer updates `transfer_count`, `delegate_reclaim_after_uap_count`, and other related fields in the CSV file.
   - The `save_delegate_reclaim_to_csv()` function handles Delegate and Reclaim transactions and saves them to a separate CSV file.

4. **Parallel Processing**:
   - The block processing is performed in parallel using `ThreadPoolExecutor` to speed up the data collection from multiple blocks.
   - This ensures efficient processing of large amounts of data, especially when querying blocks sequentially.

### Setup Instructions

1. **Clone the Repository**:

```bash
git clone https://github.com/your-username/UAP-Indexer.git
cd UAP-Indexer
```

2. **Configure the Environment**:
   - Make sure that the TRON node or the API URL you are using is configured correctly in the code.
   - For example, set `TRON_NODE_URL` and `PROXY_URL` to your desired endpoint.

3. **Run the Indexer**:
   - To process blocks and extract the required transactions, execute the script as follows:

```bash
python indexer.py
```

   - This will start the process of fetching the block data, extracting UAP transactions, and storing the results in CSV files.

4. **Check the Output**:
   - The results will be saved in CSV files in the specified directory (e.g., `UAP_agg.csv`), and each block's data will be saved in separate monthly CSV files.

### Function Details

- **get_block_transactions(block_number)**:
  - Fetches transaction data for the given block number using the TRON node API.

- **save_to_csv(transactions, output_dir, block_number)**:
  - Processes the extracted transactions and saves them to a CSV file.
  - Converts addresses to TRON base58 format and includes important fields like `owner_address`, `contractRet`, `transfer_count`, etc.

- **save_delegate_reclaim_to_csv(transactions, output_dir, block_number)**:
  - Extracts **Delegate** and **Reclaim** transactions and saves them to CSV, with additional information like `receiver_address`, `balance`, and `resource`.

- **concat_and_save_to_csv(output_dir, existing_csv_name, block_number)**:
  - Combines new block data with existing CSV and saves the updated file.

- **update_additional_uap_from_transactions(get_block_temp, agg_file_dir)**:
  - Updates the `additional_uap` field for relevant transactions in the CSV.

- **filter_invalid_rows_and_save(agg_file_dir)**:
  - Filters out invalid or unwanted rows based on predefined conditions and saves the filtered data back to the CSV.

### Example Usage

1. **Process Transactions from a Block**:

```python
block_number = 65900116
get_block_temp = get_block_transactions(block_number)  # Fetch block data
save_to_csv(get_block_temp, '', block_number)  # Save to CSV
concat_and_save_to_csv('', agg_file_dir, block_number)  # Update the aggregated CSV
update_additional_uap_from_transactions(get_block_temp, agg_file_dir)  # Update additional_uap
update_delegate_reclaim_count_from_transactions(get_block_temp, agg_file_dir)  # Update delegate_reclaim counts
update_stake_count_from_transactions(get_block_temp, agg_file_dir)  # Update stake counts
filter_invalid_rows_and_save(agg_file_dir)  # Filter invalid rows and save the data
```

2. **Parallel Block Processing**:

To process blocks in parallel, use the following configuration:

```python
from concurrent.futures import ThreadPoolExecutor, as_completed

# Example of parallel block processing
block_numbers = range(65900000, 65901000)
with ThreadPoolExecutor(max_workers=4) as executor:
    future_to_block = {executor.submit(process_block, block_number): block_number for block_number in block_numbers}
    for future in as_completed(future_to_block):
        block_number = future_to_block[future]
        try:
            future.result()
        except Exception as e:
            print(f"Error processing block {block_number}: {e}")
```

### File Structure

- `indexer.py`: Main script that runs the UAP indexer.
- `UAP_agg.csv`: Aggregated CSV containing all processed transactions.
- `transactions-block-{block_number}.csv`: Monthly CSV files containing transactions for each block.
- `README.md`: This file with setup instructions and usage examples.

### Conclusion

The **UAP Indexer** is a powerful tool for extracting and processing specific types of transactions from the TRON blockchain. It provides flexibility for handling **Update Account Permission (UAP)** transactions, **Delegate/UnDelegate Resource Contracts**, and **Reclaim** transactions, and ensures efficient processing through parallelization and optimized memory usage.

For further questions or support, feel free to open an issue in the GitHub repository.
