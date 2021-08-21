---
layout: post
categories: "en"
title: "Introduction to Bitcoin Blockchain Data"
date: 2021-08-20
tags: bitcoin sql blog
desc: "Using BigQuery to do an exploratory analysis on bitcoin open data"
---

🚧🚧 site in construction 🚧🚧

This post follows a [notebook I made on Kaggle](https://www.kaggle.com/statsfromarg/btc-blockchain-data-exploratory-analysis) as a first exploratory approach to understanding Bitcoin blockchain data. The goal of this EDA is to become familiarized with the blockchain data structure, and where to find information about blocks, transactions and addresses.

Bitcoin blockchain data at BigQuery is stored in two main tables - blocks and transactions, within transactions there are two nested tables, inputs and outputs.

To start exploring blockchain data I extracted the first rows of all four tables, mapped each field with its meaning, and made some aggregates to get insights.

* [**Blocks**](#blocks), I visualized number and size of blocks by month, since 2009.
* [**Transactions**](#txns) I analyze one transaction to understand how transactions, inputs and outputs are linked (Pending to do)  
* [**Inputs & Outputs**](#outputs) Extract the current number of unspent btc. (Pending to do)

Resources:
- [Mastering Bitcoin (book on github)](https://github.com/bitcoinbook/bitcoinbook)
- [Bitcoin ETL repo, the one that's dumping btc data on bigquery](https://github.com/blockchain-etl)
- [Getting started with Bitcoin data on Kaggle with Python and BigQuery](https://towardsdatascience.com/https-medium-com-nocibambi-getting-started-with-bitcoin-data-on-kaggle-with-python-and-bigquery-d5266aa9f52b)

<br />


---
### Blocks <a name="blocks"></a>

Blocks table contains (...).
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
# Exploring bigquery-public-data.crypto_bitcoin.blocks blocks content

q_blocks ='''SELECT
              *       
             FROM   `bigquery-public-data.crypto_bitcoin.blocks`  
             order by timestamp_month
             limit 10
          '''

blocks = client.query(q_blocks).to_dataframe()
blocks.to_csv('blocks_head.csv')
blocks.head()

</pre>

<pre>

# Aggregating block size and count over months

q_blocks_m ='''SELECT
              timestamp_month
            , count(*)   as n_blocks
            , avg(size)  as mean_size
            , avg(stripped_size)  as mean_stripped_size        
           FROM   `bigquery-public-data.crypto_bitcoin.blocks`  
           group by 1
        '''

blocks_m = client.query(q_blocks_m).to_dataframe()
blocks_m.to_csv('blocks_size_month.csv')
blocks_m.head()
</pre>

<pre>
blocks_m['month'] = blocks_m.timestamp_month.astype(str).str[0:7]
blocks_m = blocks_m.sort_values(by=['month']).copy()
</pre>

<pre>
p = sns.barplot(x = blocks_m['month'], y= blocks_m['n_blocks'], color='teal')
p.set_title('Number of blocks by month', size = 20)
p.set_xticklabels(p.get_xticklabels(), rotation=90, size = 9);

p = sns.barplot(x = blocks_m['month'], y= blocks_m['mean_stripped_size'], color='teal')
p.set_title('Average block stripped size by month', size = 20)
p.set_xticklabels(p.get_xticklabels(), rotation=90, size = 9);
</pre>



### Transactions <a name="txns"></a>

Exploring Transactions table, ignoring the nested tables inputs and outputs.

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

    FROM `bigquery-public-data.crypto_bitcoin.transactions`
    order by block_timestamp
    limit 50

     """

query_job = client.query(trx)
iterator  = query_job.result(timeout=60)
rows      = list(iterator)

# Transform the rows into a nice pandas dataframe
trx = pd.DataFrame(data=[list(x.values()) for x in rows], columns=list(rows[0].keys()))
trx.to_csv('transactions_head.csv')
trx.head()

</pre>


<pre>
# Get the hash of first transaction and explore it on blockchain.com
trx.txn_hash[0]
</pre>



Without those nested tables, there's no much "transaction" information you can get. For example, for the first transaction, we can see it is_coinbase = True since it's the first transaction included in a block, and the output value of 5000000000, in sats, was the 50 BTC reward for processing a block back in 2009. Checking Bitcoin Explorer, using the transaction hash, we can see the same info + some info about output (the address that it was sent to).

**Exploring a specific transaction**

The above-mentioned transaction was the first one, and involves 50 btc that have never been spent.
To better understand the anatomy of a more typical transaction, let's see how bitcoins are spent.

Take for example the hash `4ce18f49ba153a51bcda9bb80d7f978e3de6e81b5fc326f00465464530c052f4`. This one has two outputs - one is unspent (available balance for the hodler) and the other one is spent in the transaction with the hash `9a9294fec01d85438d1ecbdfce636b26e896b7a307fd448c3b2e224ef4bf2bae`. This one, in turn, also has two outputs, one spent and one unspent. I know this


<pre>
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
     ('4ce18f49ba153a51bcda9bb80d7f978e3de6e81b5fc326f00465464530c052f4'
      ,'9a9294fec01d85438d1ecbdfce636b26e896b7a307fd448c3b2e224ef4bf2bae')

     ORDER BY block_timestamp
     """

query_job = client.query(q_anatomy)
iterator  = query_job.result(timeout=60)
rows      = list(iterator)

# Transform the rows into a nice pandas dataframe
anatomy = pd.DataFrame(data=[list(x.values()) for x in rows], columns=list(rows[0].keys()))
anatomy
</pre>


<img src="/images/blockhain_data_txn.jpg" class=middleimg>
