# PostgreSQL -> BigQuery Pipeline  
This repo accompanies [this blog post](link-to-blog-post) and contains a deployed `dlt` pipeline that loads [this](https://github.com/fortunewalla/dvdstore) data from a PostgreSQL database into a BigQuery instance.  
  
To create your own pipeline to load data from a PostgreSQL database into a BigQuery instance, clone this repository and follow the guide below to either load the entire database or only load selected columns. Before starting:  
1. Make sure you have the latest `dlt` version installed (or install it using `pip install dlt --upgrade`)
2. It is assumed that you already have a PostgreSQL database set up and populated with data. Alternatively, you can download the linked dataset and import it to a PostgreSQL database and follow this guide.
3. It is also assumed that you already have a BigQuery instance set up into which you wish to load data. Or else follow [this guide](https://dlthub.com/docs/destinations/bigquery) for information on how to set it up.
  
## Loading the entire database into your data warehouse  
   
1. Clone this repo
2. Install necessary requirements using `pip install -r requirements_github_action.txt`
3. Create a file `.dlt/secrets.toml` and insert the following with appropriate credentials:  
```
[sources.sql_database.credentials]  
# Enter your PostgreSQL credentials below
drivername = "drivername" # please set me up!
database = "database" # please set me up!
username = "username" # please set me up!
password = "password" # please set me up!
host = "localhost" # please set me up!
port = 5432 # please set me up!
[destination.bigquery.credentials]
# Enter your BigQuery credentials below
location = "US" # please set me up!
project_id = "project_id" # please set me up!
private_key = "private_key" # please set me up!
client_email = "client_email" # please set me up!
```  
4. In the `__main__` block inside the script `sql_data_pipeline.py`, comment out the function `load_select_tables_from_database()` and uncomment the function `load_entire_database()`  
```python  
if __name__ == '__main__':
    # Load selected tables with different settings
    # load_select_tables_from_database()

    # Load tables with the standalone table resource
    # load_standalone_table_resource()

    # Load all tables from the database.
    # Warning: The sample database is very large
    load_entire_database()
```  
5. Finally, run the pipeline using  
```python3 sql_database_pipeline.py```  
  
This load the entire database onto the BigQuery instance.  
  
## Loading selected tables into your data warehouse  
Follow the steps below to load only selected tables, specify the load method (incremental, replace, append), or to deploy the pipeline:
1. Follow steps 1-3 above  
2. Modify the function `load_select_tables_from_database()` inside the script `sql_data_pipeline.py` as below: 
    1. Specify all the tables that you wish to load incrementally as below:  
    ```python
    # Replace tables `orders` and `orderlines` with the tables that you wish to load incrementally
    source_1 = sql_database().with_resources('orders', 'orderlines')
    # Replace `orderdate` with the field with which you wish to merge the new data for the table `orders`
    source_1.orders.apply_hints(incremental=dlt.sources.incremental('orderdate'))
    # Replace `orderdate` with the field with which you wish to merge the new data for the table `orderlines`
    source_1.orderlines.apply_hints(incremental=dlt.sources.incremental('orderdate'))
    ```
    2. Specify all the tables that you wish to load with replace as below:  
    ```python
    # Replace tables `inventory`, `products`, and `categories` with the tables that you wish to load with full replace
    source_2 = sql_database().with_resources('inventory', 'products', 'categories')
    # Passing write_disposition='replace' into pipeline.run() ensures that the tables are loaded with full replace
    info = pipeline.run(source_2, write_disposition='replace')
    print(info)
    ```  
    3. In case you wish to load some tables with append, do the same as 2, but pass `write disposition='append'` inside `pipeline.run()`:
    ```python
    info = pipeline.run(source_2, write_disposition='append')
    ```
3. Make sure the function `load_select_tables_from_database()` is uncommented in the `__main__` block inside the script `sql_data_pipeline.py`(and any other function is commented out):
```python  
if __name__ == '__main__':
    # Load selected tables with different settings
    load_select_tables_from_database()

    # Load tables with the standalone table resource
    # load_standalone_table_resource()

    # Load all tables from the database.
    # Warning: The sample database is very large
    # load_entire_database()
```  
4. Run the pipeline using  
```python3 sql_database_pipeline.py```  
This loads only the specified tables with the mentioned load condition.
5. Finally, if you also wish to delpoy the pipeline, follow [this guide](https://dlthub.com/docs/walkthroughs/deploy-a-pipeline)