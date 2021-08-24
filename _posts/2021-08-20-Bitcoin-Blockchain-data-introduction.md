---
layout: post
categories: "en"
title: "Introduction to Bitcoin Blockchain Data"
date: 2021-08-20
tags: bitcoin sql blog
desc: "Using BigQuery to do an exploratory analysis on bitcoin open data"
---

This post follows a [notebook I made on Kaggle](https://www.kaggle.com/statsfromarg/btc-blockchain-data-exploratory-analysis) as an exploratory approach to Bitcoin Blockchain data. The goal of this EDA is to become familiarized with blockchain data structure, and where to find information about blocks, transactions, inputs and output - the main elements of the blockchain.

The highest level are the blocks, blocks are containers of:
- a link to the previous block (this is how they are chained)
- metadata about the block itself (size, timestamp)
- transaction data stored in a Merkle Tree structure.

The second level are transactions, which include:
- a block hash, to reference the block that they are related to
- metadata about the transaction (timestamp)
- a flag to show if it's coinbase or not
- aggregate information about the amounts, number of inputs and outputs.

Since each transaction can have multiple inputs and outputs, there are two nested tables within transaction table that provide the highest detail of a transaction. With inputs and outputs tables, you can see how much bitcoin was sent in satoshis, how much was received, how much was used for fees, the origin of each input and the recipient(s) of each transaction.

Transactions can be:
- Coinbase transactions: transactions with 0 inputs, created from mining
- Typical transactions: in which the input includes 1 or more sources from were the amount was gathered to be sent.

Lastly, an important concept for bitcoin is the UTXO model: in this model, all the "available" bitcoin is the bitcoin that has not been spent. So if a transaction has a hash 123, and this hash is not used as an input in any other transaction, it means that the bitcoin is available to be spent. Otherwise, any bitcoin that is used as an input, is bitcoin that has been spent and can never be used again.

Back to blockchain data - altogether, there are four tables on Big Query:

1. [Blocks](#blocks)
2. [Transactions](#txns)
3. [Inputs](#inputs)
4. [Outputs](#outputs)

To explore Blocks table, I extract the average size by month since 2009 and plot its distribution through time. To understand transaction table, I explore the first recorded transaction in the blockchain, and to understand how inputs and outputs are used, I analyze the pizza transaction 🍕.  

<br />


---

### Connect to BigQuery

I currently use two ways to use BigQuery - kaggle and python. There are plenty of tutorials for python and BigQuery, so here I'll just share the chunk needed to access it through [kaggle](https://www.kaggle.com/bigquery/bitcoin-blockchain).

<pre>
from google.cloud import bigquery
client = bigquery.Client()
</pre>
<br>

### Blocks Table <a name="blocks"></a>

Field descriptions [from the bitcoin-etl-airflow repo](https://github.com/blockchain-etl/bitcoin-etl-airflow/blob/master/dags/resources/stages/enrich/schemas/blocks.json).

* **hash**: Hash of this block
* **size**: The size of block data in bytes
* **stripped_size**: The size of block data in bytes excluding witness data
* **weight**: Three times the base size plus the total size. [More info.](https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki)
* **number**: The number of the block
* **version**: Protocol version specified in block header
* **merkle_root**: The root node of a Merkle tree, where leaves are transaction hashes
* **timestamp**: Block creation timestamp specified in block header
* **timestamp_month**: Month of Block creation timestamp specified in block header
* **nonce**: Difficulty solution specified in block header
* **bits**: Difficulty threshold specified in block header
* **coinbase_param**: Data specified in the coinbase transaction of this block
* **transaction_count**: Number of transactions included in this block


<pre>
<i>#Lets see the first 10 blocks</i>

q_blocks ='''SELECT
             *       
             FROM   `BigQuery-public-data.crypto_bitcoin.blocks`  
             order by timestamp_month
             limit 10
          '''

blocks = client.query(q_blocks).to_dataframe()
blocks.to_csv('blocks_head.csv')
blocks.head()

</pre>

<img src="/images/bitcoin_kaggle1.png" height=180px>


<pre>
<i># Aggregating block size and count over months</i>

q_blocks_m ='''SELECT
                timestamp_month
              , count(*)   as n_blocks
              , avg(size)  as mean_size
              , avg(stripped_size)  as mean_stripped_size        
              FROM   `BigQuery-public-data.crypto_bitcoin.blocks`  
              group by 1
            '''

blocks_m = client.query(q_blocks_m).to_dataframe()
blocks_m.to_csv('blocks_size_month.csv')
blocks_m.head()
</pre>

<img src="/images/bitcoin_kaggle2.png" height=180px>


<pre>
blocks_m['month'] = blocks_m.timestamp_month.astype(str).str[0:7]
blocks_m = blocks_m.sort_values(by=['month']).copy()

p = sns.barplot(x = blocks_m['month'], y= blocks_m['n_blocks'], color='teal')
p.set_title('Number of blocks by month', size = 20)
p.set_xticklabels(p.get_xticklabels(), rotation=90, size = 9);

p = sns.barplot(x = blocks_m['month'], y= blocks_m['mean_stripped_size'], color='teal')
p.set_title('Average block stripped size by month', size = 20)
p.set_xticklabels(p.get_xticklabels(), rotation=90, size = 9);
</pre>

<img src="/images/bitcoin_kaggle3.png" height=280px>
<br>
<img src="/images/bitcoin_kaggle4.png" height=280px>



### Transactions <a name="txns"></a>

Field descriptions:

* **hash**: The hash of this transaction
* **size**: The size of this transaction in bytes
* **virtual_size**: The virtual transaction size (differs from size for witness transactions)
* **version**: Protocol version specified in block which contained this transaction
* **lock_time**: Earliest time that miners can include the transaction in their hashing of the Merkle root to attach it in the latest block of the blockchain
* **block_hash**: Hash of the block which contains this transaction
* **block_number**: Number of the block which contains this transaction
* **block_timestamp**: Timestamp of the block which contains this transaction
* **block_timestamp_month**: Month of the block which contains this transaction
* **input_count**: The number of inputs in the transaction
* **output_count**: The number of outputs in the transaction
* **input_value**: "Total value of inputs in the transaction
* **output_value**: Total value of outputs in the transaction
* **is_coinbase**: true if this transaction is a coinbase transaction
* **fee**: The fee paid by this transaction
* **inputs**: This includes a json formatted field with all the variables in the Inputs table, which is explored later in the notebook.
* **outputs**: Also json field with all the outputs, explored later as well.


<pre>
<i>#Lets see the first 50 transactions</i>

trx = """
      SELECT  
     `hash`            as txn_hash
     ,size             as txn_size
     ,version          as txn_version
     ,lock_time        as txn_lock_time
     ,block_hash       as txn_block_hash
     ,block_number     as txn_block_number
     ,block_timestamp  as txn_block_timestamp
     ,input_count      as txn_input_count
     ,input_value      as txn_input_value
     ,output_count     as txn_output_count
     ,output_value     as txn_output_value      
     ,is_coinbase      as txn_is_coinbase
     ,fee              as txn_fee      

    FROM `BigQuery-public-data.crypto_bitcoin.transactions`
    order by block_timestamp
    limit 50

     """

query_job = client.query(trx)
iterator  = query_job.result(timeout=60)
rows      = list(iterator)

<i># Transform the rows into a nice pandas dataframe</i>
trx = pd.DataFrame(data=[list(x.values()) for x in rows], columns=list(rows[0].keys()))
trx.to_csv('transactions_head.csv')
trx.head()

</pre>

<img src="/images/bitcoin_kaggle5.png" height=260px>

Without those nested tables, there's no much "transaction" information you can get, other than the timestamp, the output value in satoshis, and a flag if it's coinbase or not. Coinbase = True means that it is the first transaction included in a block, that is, the one is created for mining. We can use the hash and go online and explore a transaction.

<pre>
<i># Get the hash of the very first bitcoin transaction and explore it on blockchain.com</i>
trx.txn_hash[0]
</pre>

`4a5e1e4baab89f3a32518a88c31bc87f618f76673e2cc77ab2127b7afdeda33b`



Use this hash and check it on the [blockchain explorer](https://www.blockchain.com/btc/tx/4a5e1e4baab89f3a32518a88c31bc87f618f76673e2cc77ab2127b7afdeda33b) to confirm some values from the table. For example: is_coinbase = True since it's the first transaction included in a block, the output value of 5000000000, in sats, was the 50 BTC reward for processing a block back in 2009. Other pieces of info such as addresses can't be obtained from the transactions table alone, but we need to see the inputs and outputs.

<img src="/images/bitcoin_kaggle6.png" height=400px>



### Inputs <a name="inputs"></a>

Field descriptions:

* **index**: 0-indexed number of an input within a transaction
* **spent_transaction_hash**: The hash of the transaction which contains the output that this input spends
* **spent_output_index**: The index of the output this input spends
* **script_asm**: Symbolic representation of the bitcoin's script language op-codes
* **script_hex**: Hexadecimal representation of the bitcoin's script language op-codes
* **sequence**: A number intended to allow unconfirmed time-locked transactions to be updated before being finalized; not * **currently** used except to disable locktime in a transaction
* **required_signatures**: The number of signatures required to authorize the spent output
* **type**: The address type of the spent output
* **addresses**: Addresses which own the spent output
* **value**: The value in base currency attached to the spent output

### Outputs <a name="outputs"></a>

Field descriptions very similar to the inputs table:

* **index**: 0-indexed number of an output within a transaction used by a later transaction to refer to that specific output
* **script_asm**: Symbolic representation of the bitcoin's script language op-codes
* **script_hex**: Hexadecimal representation of the bitcoin's script language op-codes
* **required_signatures**: The number of signatures required to authorize spending of this output
* **type**: The address type of the output
* **addresses**: Addresses which own this output
* **value**: The value in base currency attached to this output

<br>
<br>

**Exploring a specific transaction**

The previously mentioned transaction was the first one, and involves 50 btc that have never been spent, so it has no input (was created from mining) and no linked transaction after that (never spent). To go deeper into a more typical transaction that has inputs and outputs, we need the nested tables inputs and outputs.

A good example for is the 🍕 transaction, with hash `a1075db55d416d3ca199f55b6084e2115b9345e16c5cf302fc80e9d5fbf5d48d` we can pull information from the three tables. The pizza transaction is the first known use of Bitcoin to buy delicious food. It was broadcasted to the network on May 22, 2010, after Laszlo Hanyecz paid 10,000 for two pizzas.




<pre>
<i>#Lets see the pizza transaction, and how the recipient of those 10K btc spent the coins)</i>


q_anatomy = """
      SELECT  
     `hash`           as txn_hash
     ,block_timestamp as txn_block_tms
     ,input_count     as txn_input_count
     ,input_value     as txn_input_value
     ,i.spent_transaction_hash   as nested_hash  
     ,i.value         as nested_input_value
     ,output_count    as txn_output_count
     ,output_value    as txn_output_value
     ,o.value         as nested_output_value     

     FROM `bigquery-public-data.crypto_bitcoin.transactions`
     JOIN UNNEST(inputs)  as i
     JOIN UNNEST(outputs) as o
     WHERE `hash` in
     ('a1075db55d416d3ca199f55b6084e2115b9345e16c5cf302fc80e9d5fbf5d48d',
     'cca7507897abc89628f450e8b1e0c6fca4ec3f7b34cccf55f3f531c659ff4d79')

     ORDER BY block_timestamp
     """

query_job = client.query(q_anatomy)
iterator  = query_job.result(timeout=60)
rows      = list(iterator)

<i># Transform the rows into a nice pandas dataframe</i>
anatomy = pd.DataFrame(data=[list(x.values()) for x in rows], columns=list(rows[0].keys()))
anatomy

</pre>

<img src="/images/bitcoin_kaggle7.png" height=400px>

Here we get 133 rows - for just two transactions! Let's analyze some information we get from this table:

1. There are 131 rows that have a txn_hash starting with a1075... these are the *inputs* of the pizza purchase. This means that the pizza eater pulled coins from 131 different sources, adding up to 10,000 BTC, which were sent to only one address.

2. The input count is 131 (gathering from 131 sources) and the output count is just 1 (sent to just one address).  

3. The transaction input value is 1000099000000, because it's including the fee.

4. The transaction output is 1000000000000, because the recipient doesn't receive the fee.

5. There are two rows that have a txn_hash starting with cca750... this transaction is what the recipient of the 10,000 did: he split the 10K in two addresses, sending 577700000000 to one and 422300000000 to the other.

6. We can see how each transaction is linked to the other, in these two rows the nested_hash shows the input of the transaction, which is the hash a1075... - the pizza transaction.

<img src="/images/bitcoin_kaggle8.png" height=280px>

Mapping the elements of the inputs and outputs table with concrete transactions was my only method to understand what each table contains. After mapping the most essential fields with its meaning, it could be possible to actually analyze Bitcoin Blockchain data -- but these are baby steps, as bitcoin blockchain data is hard to load and manipulate.


Resources:
- [Mastering Bitcoin](https://github.com/bitcoinbook/bitcoinbook)
- A quick [Relational graph](https://medium.com/google-cloud/full-relational-diagram-for-bitcoin-public-data-on-google-BigQuery-3c4772af6325)  mapping the links among these four tables
- A tutorial to [Getting started with Bitcoin data](https://towardsdatascience.com/https-medium-com-nocibambi-getting-started-with-bitcoin-data-on-kaggle-with-python-and-BigQuery-d5266aa9f52b)  on Kaggle with Python and BigQuery
- Some [queries from Big Query](https://cloud.google.com/blog/products/data-analytics/introducing-six-new-cryptocurrencies-in-BigQuery-public-datasets-and-how-to-analyze-them), when they published the blockchain data
- [Bitcoin ETL repo](https://github.com/blockchain-etl), the one that's dumping btc data on BigQuery
