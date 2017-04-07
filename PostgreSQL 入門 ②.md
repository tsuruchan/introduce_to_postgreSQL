PostgreSQL 入門 ②

### サブクエリー
select文の結果をまたselect文で処理する。

~~~SQL
select カラムリスト from (select文) as サブクエリー名:
~~~

サブクエリー → 外側のselect文という順で処理します。

### リレーション
テーブル（行×カラム）のような性質を持つデータ

リレーショナルデータベースとは、リレーションのデータベースのことです。

### サブクエリーの結果による絞り込み
where節のin演算子とサブクエリーを組み合わせることができます。

~~~SQL
select * from access_log where request_path in
(select requrst_path from web_pages where page_category = 'item');
~~~


候補が沢山になる場合、テーブルに入れておくと楽です。


### スカラーサブクエリー
スカラーサブクエリーとは、select節やwhere節の式の一部として使うサブクエリーのことです。

~~~SQL
select cast(order_count as real) / (order_countの合計)
from item_combination_order_counts;
~~~

~~~SQL
(order_countの合計) = (select sum(order_count) from item_combination_order_cunts)
~~~

## ウィンドウ関数
ウィンドウ関数は分析関数とも呼ばれる関数です。

|関数名|意味|
|:--|:--|
|count|グループ内の行数を数える|
|sum|グループ内の合計や累積和をとる|
|avg|グループ内の平均をとる|
|rank|グループ内をソートして順位をつける（重複あり）|
|raw_number|グループ内をソートして順位をつける（重複無し）|
|lag|グループ内をソートして前の行の値をとる|
|lead|グループ内をソートして後の行の値をとる|

### rankウィンドウ関数
月ごとの売上TOP３を出す。

~~~SQL
select sales_month, shop_id, sales_amount. rank() over (partition by sales_month order by sales_amount desc) as monthly_sales_rank from monthly_sales
~~~

![スクリーンショット 2017-04-05 15.57.30.png](https://qiita-image-store.s3.amazonaws.com/0/83641/fa4174f0-fbf9-bc5c-e9bf-72552e92ed8c.png "スクリーンショット 2017-04-05 15.57.30.png")

> partition by sales_month

sales_monthをキーにしてグループを作る（group byする）

> order by sales_amount desc

作ったグループ内で、行をsales_amountの多い順にソートする

【処理順番】
ウィンドウ関数の実行順序はhaving節の後

![スクリーンショット 2017-04-06 16.00.43.png](https://qiita-image-store.s3.amazonaws.com/0/83641/d897580e-3323-8b16-3e19-758277efff6a.png "スクリーンショット 2017-04-06 16.00.43.png")

### row_numberウィンドウ関数
rankウィンドウ関数と使い方は同じ。値が全く同じ場合にも重複しない順位を返します。

row_numberウィンドウ関数が役立つ典型的な例として、「履歴テーブル」から最新の行を得る場合が挙げられます。

#### 以下のようなテーブルがあったときに、最新のデータを取り出すSQL

![スクリーンショット 2017-04-06 16.07.38.png](https://qiita-image-store.s3.amazonaws.com/0/83641/5229dd3f-e568-74d7-e4bd-ecd670945f23.png "スクリーンショット 2017-04-06 16.07.38.png")

~~~SQL
select customer_id, customer_name, customer_age from
(select customer_id, customer_name, customer_age, row_number() over (partition by customer_name order by created_time desc) as newer_rank from customers) as temp
where newer_rank = 1;
~~~

### 月ごとの売上合計に対して各店舗の占める割合(対全体比)

~~~SQL
select sales_month, shop_id, 100.0 * sales_amount / sum(sales_amount) over (partition by sales_month) as sales_ratio from monthly_sales;
~~~

![スクリーンショット 2017-04-06 16.33.40.png](https://qiita-image-store.s3.amazonaws.com/0/83641/ea605821-f79e-a650-ab33-7c24a086519d.png "スクリーンショット 2017-04-06 16.33.40.png")

> sum(sales_amount) over (partitio by sales_month)

sales_monthでグループ化した物に対して、sum(sales_amount)することで、月ごとの総売上を計算する。


### 累積和の対全体比
店舗ごとの年間売上目標に対して、毎月の達成率を計算するような場合だと、年間全体の売上目標に対する累積売上の比率をだしたいでしょう。

#### ウィンドウフレーム

~~~SQL
rows between 前側の行の範囲 and 後ろ側の行の範囲
~~~

__前の行の範囲__
- 現在処理中の行自身を指定する「current row」
- 「n行前まで」を指定する「n preceding」
- 「前にある行全部」を指定する「unbounded preceding」

__後ろ側の行の範囲__
- 現在処理中の行自身を指定する「curent row」
- 「n行後まで」を指定する「n following」
- 「後ろにある行全部」を指定する「unbounded following」


~~~SQL
select sales_month, shop_id, sales_amount, sum(sales_amount) over(partition by shop_id order by sales_month rows between unbounded preceding and current row) as ruiseki from monthly_sales; 
~~~ 

### デシル分析
デシル分析は、購入金額の順に顧客を人数ベースで10等分する分析。
顧客をランク付けして、湯量顧客を探そうとする分析です。

select文の結果をデシル分析するにはいくつか方法がある。

①サブクエリーとして呼ぶ

②ビューとして定義する

ビュー（view）はSQLの機能で、あたかもテーブルのように見えるけれども、使われる度にselect文を実行してその結果を見せてくれる。

~~~SQL
create view yearly_orders as 
select …
~~~

### 時系列データの処理｜移動平均を計算する
移動平均とは、下図のように、直近のn個の平均をその時点の値とする計算手法です。

移動平均を使うと、日々の細かい変動要因をならして、グラフをなめらかにすることができます。


![スクリーンショット 2017-04-06 17.35.10.png](https://qiita-image-store.s3.amazonaws.com/0/83641/74a948ec-35dd-f201-a516-1a855bbe7970.png "スクリーンショット 2017-04-06 17.35.10.png")


monthly_salesテーブルを例にとって、売上の移動平気を計算するSQL

~~~SQL
select sales_month, shop_id, sales_amount, avg(sales_amount) over (partition by shop_id order by sales_month rows between 5 preceding and current row) as oving_age from monthly_sales;
~~~

## 横持ち・縦持ちテーブル
基本は縦持ち

__横持ちのメリット__
- 速度
- 行無いでの処理のしやすさ
- 縦持ちに比べて、行数がすくなくて済む場合がある
- 行数を少なく出来れば、Excelで処理できる

### 縦持ちから横持ちのテーブルに変換
~~~SQL
select * from (select order_id, unnest(array[item_id1, item_id2, item_id3, item_id4]) as item_id from order_details) as temp where item_id is not null; 
~~~

### case文

~~~SQL
case 式X
when 式１ then 結果１
when 式２ then 結果２
when 式３ then 結果３

else 結果E
end
~~~

## セッション分析

【目的】顧客が商品を検索した後に、カートに商品を入れている率

### セッショナイズ
アクションが所属するセッションを決めること。

今回は、同一ユーザーの連続するアクセスで、アクセス感覚が15分未満である一連のアクセスを１つのセッションとします。

~~~SQL
create view sessionized_access_log as
select *, sum(session_head_flag) over (partition by customer_id order by request_time rows between unbounded preceding and current row ) as session_id from (
select customer_id, 
case 
when lag(request_time) over (partition by customer_id order by request_time) is null then 1
when request_time - lag(request_time) over (partition by customer_id order by request_time) >= interval '15 minute' then 1
else 0 end as session_head_flag,
request_time, request_path from access_log) temp;
~~~


## explain文

~~~SQL
explain select文;
~~~

実行計画を見ることが出来る。


## 効率的な分析をするには
①対象データを減らす

②中間データをテーブルに保存する

基本的に、実行時間は対象データの量に比例します。クエリーを書いている途中は得に、本来より少量のデータで試すことを心がけましょう。


最終的に1ヶ月のデータを処理したいときに、最初から1ヶ月文のデータを使うとやたら時間がかかります。

まずは10分間などの時間に小さく分けて、SQLを実行していく方が、最終的には時間が短縮されるでしょう。


