	# 100knock-data_analytics

import numpy as np
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt
%matplotlib inline

from sklearn.cluster import KMeans
from sklearn.preprocessing import StandardScaler
from sklearn.decomposition import PCA
from dateutil.relativedelta import relativedelta

## ---------- 第１章：ウェブからの注文数を分析 ---------- ##

# データの読み込み
customer_master = pd.read_csv('/content/drive/MyDrive/Colab Notebooks/1章/customer_master.csv')
item_master = pd.read_csv('/content/drive/MyDrive/Colab Notebooks/1章/item_master.csv')
transaction_1 = pd.read_csv('/content/drive/MyDrive/Colab Notebooks/1章/transaction_1.csv')
transaction_2 = pd.read_csv('/content/drive/MyDrive/Colab Notebooks/1章/transaction_2.csv')
transaction_detail_1 = pd.read_csv('/content/drive/MyDrive/Colab Notebooks/1章/transaction_detail_1.csv')
transaction_detail_2 = pd.read_csv('/content/drive/MyDrive/Colab Notebooks/1章/transaction_detail_2.csv')

# データの結合
transaction = pd.concat([transaction_1, transaction_2], ignore_index=True)
transaction_detail = pd.concat([transaction_detail_1, transaction_detail_2], ignore_index=True)

# transaction_detailとtransactionを横に結合（粒度の細かいtransaction_detailがメイン）
join_data = pd.merge(transaction_detail, transaction[["transaction_id", "payment_date", "customer_id"]], on="transaction_id", how="left")

# join_dataとcustomer_master,item_masterを横に結合（粒度の細かいtransaction_detailがメイン）
join_data = pd.merge(join_data, customer_master, on="customer_id", how="left")
join_data = pd.merge(join_data, item_master, on="item_id", how="left")

# quantityとitem_priceからpriceを算出
join_data["price"] = join_data["quantity"] * join_data["item_price"]

# object ⇒ datetime へ変換
join_data['payment_date'] = pd.to_datetime(join_data['payment_date'])
join_data['payment_month'] = join_data['payment_date'].dt.strftime('%Y%m') # 年月単位のカラムを作成
join_data[['payment_date', 'payment_month']].head()

# カラム別でデータを集計
pd.pivot_table(join_data, index='item_name', columns='payment_month', values=['price', 'quantity'], aggfunc='sum')
