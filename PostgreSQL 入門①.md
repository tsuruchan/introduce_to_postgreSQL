# PostgreSQL 入門
![スクリーンショット 2017-04-01 17.41.48.png](https://qiita-image-store.s3.amazonaws.com/0/83641/a7adc000-1eda-5914-d4cb-b348adf09b6b.png "スクリーンショット 2017-04-01 17.41.48.png")

---
### select

~~~SQL
select * from variable ;
~~~

>from variable

variableテーブルのデータを検索対象にする。

>select *

*（すべてのカラムを対象にする）

---

~~~SQL
select * from variable limit 10;
~~~

結果行数の上限を指定

---

~~~SQL
select count(*) from variable;
~~~

テーブルの行数を数える

※ count(*)の*は行のことを指す。

---

### where節
~~~SQL
select * from variable where variable >= timestamp '2010-01-01 00:00:00'
~~~

### SQLの比較演算子
|演算子|意味|
|:--|:--|
|a = b|aとbが等しい|
|a <> b|aとbが異なる|
|val between a and b|valはa以上b以下|

### 論理句
|式|意味|
|:--|:--|
|X and Y|XかつY|
|X or Y|XまたはY|
|not X|Xではない|

~~~SQL
select * from variable where variable in ('str', 'str')
~~~

こういう書き方もできる。

### 文字列の絞り込み

---
### order by 節による行の並び替え

~~~SQL
select * from variable order by variable
~~~

無指定の場合、昇順にソートします。
descを指定すれば、降順にソートします。

~~~SQL
select * from variable order by variavle desc;
~~~

#### 複数の並び替え条件を指定する
例えば、customer_idの順で並び替えて、同じcustomer_idの中ではさらにrequest_timeが古い順にソートする。

~~~SQL
select * from variable order by customer_id, request_time;
~~~

---
## 集計
### 集約関数
|関数|効果|
|:--|:--|
|count()|行数を数える|
|sum()|合計|
|avg()|平均|
|min()|最小値|
|max()|最大値|

### count関数
> count(*)
行の数を数える

> count(variable)
その変数の非null値の個数を数える

> count(distinct variable)
その変数の非null値かつ重複しない要素の個数を数える

→　ユニークユーザーなどを調べるときに活躍！


### nullの扱い
![スクリーンショット 2017-04-01 19.38.27.png](https://qiita-image-store.s3.amazonaws.com/0/83641/19b64830-11d0-4d5c-77c3-6ad004c92181.png "スクリーンショット 2017-04-01 19.38.27.png")

### group by

~~~SQL
select customer_id, count(*) from access_log group by customer_id;
~~~

group by costomer_idでIDごとにグループ化します。

しして各グループの中で、select節の式を集約計算します。

![スクリーンショット 2017-04-01 20.50.24.png](https://qiita-image-store.s3.amazonaws.com/0/83641/61841c36-d3d2-33af-e233-d38700428984.png "スクリーンショット 2017-04-01 20.50.24.png")


※　select節にはgroup by節に書いたカラムか集約変数しか使えません。

月間UU

~~~SQL
select request_month, count(distinct customer_id) from access_log_wide group by request_month 
~~~

月間UUが10000以下の付きだけ抜き出す
### having節
whereはgruop byの前にしか実行できない。

~~~SQL
select request_month, count(distinct customer_id) from access_log_wide group by request_month having count(*) < 10000
~~~

テーブルの集計結果＝*

## SQL節のまとめ
書く順番もこのとおりじゃないとイケない。

|節名|機能|
|:--|:--|
|select|取得カラムを指定|
|from|対象テーブルを指定|
|where|絞り込みの条件を指定|
|group by|グループ化の条件を指定|
|having|グループ化後の絞り込み条件の指定|
|order by|ソート指定|
|limit|取得行数の指定|

処理の順番

※ select以外は書いてある順番と同じ

![スクリーンショット 2017-04-01 21.18.17.png](https://qiita-image-store.s3.amazonaws.com/0/83641/e2e1acf5-c00d-1367-d9bb-ca3f3866f878.png "スクリーンショット 2017-04-01 21.18.17.png")


## 新しいカラムの生成
> select bariable + 1
のようにできる。

#### 型のキャスト
> cast(cariable as real) = (variable :: real)

### 文字列処理の関数
> select customer_name, char_length(customer_name) from customers

空白もカウントするので注意！

### 主要な文字列処理の演算子
|演算子・関数|意味|
|:--|:--|
|a||b|aとbを連結する|
|substring(str, begin, end)|インデックスは１〜|
|trim(str)|strの両端の空白文字を消去|
|split_part(str, delim, nth)|strを区切り文字delimで分割し、nth番目の文字列を返す|


### 日付と時刻の演算
|関数・演算子|例|効果|
|:--|:--|:--|
|current_data|current_data|現在の日付|
|current_timestamp|current_timesamp|現在の時刻|
|extract|extract(year from date '2015-05-15')|日付や時刻の特定の部分を取り出す|
|data_trunc|data_trunc('month', date '2015-05-15')|日付や時刻の特定の部分までを切り詰める|

#### extract関数
~~~SQL
select * from access_log where extract(year from request_time) = 2015
~~~

このようにすることで、2015年のデータをもってくる。
year, month, dayしか指定できない。

#### date_trunc関数
~~~SQL
select date_trunc('month', request_time) from access_log;
~~~

2014-02のデータが欲しい場合はextract関数では対処できません。

> date_trunc('month', tm)

と書くと、任意のtimestamp型の値tmから月より小さい単位（日分秒）をすべて切り捨てます。

ex.) '2014-02-04 12:34:25' →　'2014-02-01 00:00:00'


2014-02の行だけ抜き出したい場合

~~~SQL
select * from access_log where date_trunc('month', request_time) = timestamp '2014-02-01 00:00:00';
~~~

### as句

~~~SQL
select date_trunc('month', request_time) as request_month from access_log;
~~~

この名前は同じselect分の中で使うことも出来ます。

![スクリーンショット 2017-04-02 11.11.01.png](https://qiita-image-store.s3.amazonaws.com/0/83641/e0b71983-1954-51ed-52d9-135ae30a34db.png "スクリーンショット 2017-04-02 11.11.01.png")

![スクリーンショット 2017-04-02 11.12.42.png](https://qiita-image-store.s3.amazonaws.com/0/83641/8986da4e-b780-36eb-231a-e79fc63342e4.png "スクリーンショット 2017-04-02 11.12.42.png")


### join句
~~~SQL
select * from テーブル名２ as 別名１ join テーブル名２ as 別名２ on ジョイン条件;
~~~

ジョイン条件　

> 別名１.外部キー = 別名２.プライマリーキー

![スクリーンショット 2017-04-02 11.37.34.png](https://qiita-image-store.s3.amazonaws.com/0/83641/238a7a17-ad73-3907-2e22-dcc662b1ab6c.png "スクリーンショット 2017-04-02 11.37.34.png")

### テーブルを作成する
~~~SQL
create table テーブル名(
カラム名１　型１,
カラム名２　型２,
カラム名３　型３,
);
~~~

カラム数はpostgreSQLでは１０００以上使えます。

### テーブルを削除する
~~~SQL
drop table テーブル名;
~~~

### テーブルにデータを入れる
~~~SQL
insert into テーブル名(カラム１, カラム２,カラム３, …)
valurs (値１,値２,値３,…)
~~~

連続して入れることもできます。

~~~SQL
insert into テーブル名(カラム１, カラム２,カラム３, …)
valurs (値１,値２,値３,…),
valurs (値１,値２,値３,…),
valurs (値１,値２,値３,…)
~~~

#### CSVファイルからデータを入れる
~~~SQL
copy テーブル名 from 'CSVファイルのフルパス' with format csv;
~~~

### select文の結果をテーブルに投入するinsert select文
~~~SQL
insert into daily_sales
select sales_date, sum(sales_amount)
from sales gruop by sales_date
~~~

※　postgreSQLでは、 __create table as 文__ に上記のプログラムをまとめることができます。

~~~SQL
create table daily asales as
select sales_date, sum(sales_amount)
from sales gruop by sales_date
~~~

### outer join句
対応する行がなくても行を残す場合

~~~SQL
select request_time, request_path, prefecture from access_log as a left outer join customer_locations as l on a.customer_id = l.customer_id
~~~

#### 種類
left, right, full

> left outer join

左のテーブル（select文で先に書いてある方のテーブル）の行を結果に残して、右のテーブルのカラムの値はnullにする。

> full outer join

両方のテーブルの行を保持します。

### coalesce(カラム, '文字')
カラムがnullでなければそれを返し、nullならば不明を返します。

### セルフジョイン
セルフジョインは一つのテーブルを自分自身とジョインする方法です。

![スクリーンショット 2017-04-03 16.20.00.png](https://qiita-image-store.s3.amazonaws.com/0/83641/d1259d4f-50cb-7b27-c210-f5d84b08372e.png "スクリーンショット 2017-04-03 16.20.00.png")

年単位に店ごとの売上を記録しているテーブルがあるとすると、


~~~SQL
select curr.year, curr.shop_id, cast(curr.sales_amount as real) / prev.sales_amount as growth_rate 
from yearly_sales as curr left outer join 
on curr.year - 1 = prev.year and curr.shop_id = prev.shop_id
~~~



![スクリーンショット 2017-04-03 16.01.33.png](https://qiita-image-store.s3.amazonaws.com/0/83641/406cc024-4b29-e140-ce53-db7cc3412fae.png "スクリーンショット 2017-04-03 16.01.33.png")


### クロスジョイン
ジョインの行う処理は、正確には２つのテーブルの行どうしの組み合わせをすべて生成します。

その後、on句の条件を満たす行だけを空いてのテーブルからもってきます。

![スクリーンショット 2017-04-04 9.51.00.png](https://qiita-image-store.s3.amazonaws.com/0/83641/f28ba1b7-c89d-45b5-8b6a-e8885e9cdd2a.png "スクリーンショット 2017-04-04 9.51.00.png")


そして、 __全ての組み合わせを生成する__ 処理だけを、SQLではcross join句と呼びます。

### 

