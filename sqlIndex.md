# MySQL インデックスについて
## 環境:MySQL5.5.32

### インデックスとは
- インデックスとは、特定のカラム値のある行のデータを素早く参照したいときに使う。  
テーブルから検索するのではなく、インデックスを検索することで大規模なデータを高速に抽出することができる。
例えば、大量の名前が無造作に入っているカラムを対象にあいうえお順で並び替えたインデックスを作成しておけば、抽出が早くなる。
- データの量が少ないカラムのインデックスを作成しても効果が薄い。
- 主キーはそのテーブル内での重複がないため、自動的にインデックスが作成される。  
### インデックス情報の確認
- SHOW INDEX構文を使用する。  
```SQL
SHOW INDEX FROM テーブル名
```  
- filmテーブルのインデックス情報を確認する。
```SQL
SHOW INDEX FROM film;
```

| table | Non_unique | Key_name | Seq_in_index | Columun_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment| Index_comment |
|:--:|:------------: |:-----:|:-----:|:------------:|:------------:|:------------:|:------------:|:------------:|:-----------:|:-----------:|:-----------:|:-----------:|
| film | 0 | PRIMARY | 1 | film_id | A | 879 | null | null | | BTREE | | | 
| film | 1 | idx_title | 1 | title | A | 879 | null | null | | BTREE | | | 
| film | 1 | idx_fk_language_id | 1 | language_id | A | 1 | null | null | | BTREE | | | 
| film | 1 | idx_fk_original_language_id | 1 | original_language_id | A | 1 | null | null | YES | BTREE | | | 

SHOW INDEX構文は、テーブル名に対応したINDEXの情報のフィールドを返す。注目すべき点は以下である。 
- table  
テーブル名。  
- Non_unique  
インデックス内のレコードが重複した値を含むことができるかできないかを表す。  
- Seq_in_index  
インデックスのフィールド番号。  
- columun_name  
インデックスに対応したカラムの名前。  
- collation  
インデックス内でのソートの方法。  
- Cardinality  
ユニークな値や文字を数えて表示されている。  
この数が多いほどインデックスを利用した際の効果が高い。  
レコード数に比べて値や文字の少ないものはインデックスを作成しても効果が薄いということになる。  
- Sub_part  
カラムが部分的にインデックス設定されている場合に設定された文字の数を表す。  
### インデックスの作成  
- テーブルを作成する時にINDEXを作成  
```SQL
CTEATE FROM テーブル名(カラム名 型,INDEX(カラム名))
```  
- 既存のテーブルのINDEXを作成  
```SQL
CREATE INDEX インデックス名 ON テーブル名(カラム名)
``` 
カラム名の後にはソートを指定できる。  
### インデックスの削除  
- DROP INDEX構文を使用する。  
```SQL
DROP INDEX インデックス名 ON テーブル名
```
### 複合インデックスの作成
- 複合インデックスとは、1つのテーブルの複数のカラムにINDEXを作成することである。 絞込みの際に複数の列をキーとして抽出する場合に有効である。

```SQL
CREATE INDEX インデックス名 ON テーブル名(カラム名,カラム名)
```
### ユニークインデックスについて　　
- ユニークインデックスとは、テーブルのレコード内に重複をさせないインデックスである。　　
- 主キーと違い、ユニークインデックスは複数のカラムにつけることができる。  
```SQL
CREATE UNIQUE INDEX インデックス名 ON テーブル名(カラム名1,カラム名2)
```
### インデックスが利用されているかの調査(クエリの実行計画の調査)  
- EXPLAINコマンドを使用する。  
- EXPLAINコマンドとは、SELECT句で使用されるテーブルに関する情報を返す。使用する際には、SELECT文の前にEXPLAINを記述する。(利用しているMySQLサーバはversion5.5.52)  
- MySQL5.6.3からはSELECT文以外のDELETE、INSERT、REPLACE、UPDATEで使用可能である。
```SQL
EXPLAIN SELECT * FROM film WHERE title ='WORST BANGER';
```
| id | select_type   | table |  type | possible_keys| key          | key_len      | ref          | rows         | extra       |
|:--:|:------------: |:-----:|:-----:|:------------:|:------------:|:------------:|:------------:|:------------:|:------------:|
| 1  |  SIMPLE       | film  | ref   | idx_title    | idex_title   | 767          | const        | 1            |Using index condition
- id  
SELECT句の識別子。  
- select_type  
SELECT句の型。  
SIMPLEという表示は、一つのSELECT文という意味で、サブクエリが使用されていないということである。  
- table  
出力行の元のテーブル。
- type  
結合の型。対象のテーブルに対してどのようにアクセスをするのかを示す。  
INDEXが使用されているかどうかを確認することができる。
constは主キーによるアクセス、INDEXによるアクセスを意味しているので、処理が早い。  
refはユニークでないINDEXを使って検索したという意味である。  
- possible_keys  
選択可能なインデックス。
- key  
実際にSELECT句の中で選択したキー。
- key_ren   
選択されたキーの長さ。短いほど処理が早い。
- ref 
インデックスと比較されるカラム。  
検索条件に定数を指定した場合にはconstと表示される。  
テーブル結合をしている場合には結合したテーブルのカラムが表示される。  
- rows  
テーブルから調査された行数。  
- extra  
その他の追加情報。  
Using index conditionはクエリがINDEXだけでデータを抽出できたという意味である。  
### インデックス検索を使用した例
```SQL
EXPLAIN SELECT * 
FROM address
WHERE city_id = 10; 
```
| id | select_type   | table |  type | possible_keys| key          | key_len      | ref          | rows         | extra       |
|:--:|:------------: |:-----:|:-----:|:------------:|:------------:|:------------:|:------------:|:------------:|:-----------:|
| 1  |  SIMPLE       | address  | ref | idx_fk_city_id   | idx_fk_city_id| 2        | const        | 1          |           |  

- INDEXを使用しているaddressのcity_idカラムに対して等価検索を行っている。インデックスを使って等価検索を行った場合、typeはrefと表示される。
 
### インデックスでの検索が効いていないパターンの例
#### 1.INDEXが張られていないカラムを条件にして検索する
```SQL
EXPLAIN SELECT address_id
FROM address
WHERE address2 IS NULL
```

| id | select_type   | table |  type | possible_keys| key          | key_len      | ref          | rows         | extra       |
|:--:|:------------: |:-----:|:-----:|:------------:|:------------:|:------------:|:------------:|:------------:|:-----------:|
| 1  |  SIMPLE       | address  | ALL |null          | null |          | null         | 628         |   Using where       |  

- INDEXが張られていないカラムで検索条件を記述することはINDEXでの検索にはならないため、typeがALLとなる。
#### 2.インデックス全体を読み込んでいる
```SQL
EXPLAIN SELECT title FROM film;
```
| id | select_type   | table |  type | possible_keys| key          | key_len      | ref          | rows         | extra       |
|:--:|:------------: |:-----:|:-----:|:------------:|:------------:|:------------:|:------------:|:------------:|:-----------:|
| 1  |  SIMPLE       | film  | index |null          | idx_title| 767         | null         | 953          |   Using index          |  

- typeがindexとなっており、インデックス全体を読み込んでいるため処理が遅い。 
#### 3.INDEXが張られているカラムの左辺で計算をする
```SQL
EXPLAIN SELECT
film_id
FROM film
WHERE film_id -1 = 500;
```
| id | select_type   | table |  type | possible_keys| key          | key_len      | ref          | rows         | extra       |
|:--:|:------------: |:-----:|:-----:|:------------:|:------------:|:------------:|:------------:|:------------:|:-----------:|
| 1  |  SIMPLE       | film  | index   |null          | idx_fk_language_id     | 1     | null         | 978        | Using where;Using index           |

- indexを使ったカラムに対しての計算式を左辺に記述してしまうとフルインデックススキャンになる。計算式は右辺に書く。
### インデックスでの検索において効率の良い例  
```SQL
EXPLAIN SELECT * FROM film WHERE film_id < 100 AND title LIKE 'A%';
```  
| id | select_type   | table |  type | possible_keys| key          | key_len      | ref          | rows         | extra       |
|:--:|:------------: |:-----:|:-----:|:------------:|:------------:|:------------:|:------------:|:------------:|:-----------:|
| 1  |  SIMPLE       | film  | range |PRIMARY,idx_title    | PRIMARY | 2         | null         | 98       |   Using Where   |  

- Cardinalityが高い主キーとINDEXが作成されているCardinalityが高いfilmのタイトルから抽出した結果を出力していて、INDEXを利用した検索として効率が良い。  
### 複合インデックスの利用例
```SQL
EXPLAIN SELECT * FROM inventory WHERE store_id = 1 AND film_id < 500; 
```
| id | select_type   | table |  type | possible_keys| key          | key_len      | ref          | rows         | extra       |
|:--:|:------------: |:-----:|:-----:|:------------:|:------------:|:------------:|:------------:|:------------:|:------------:|
| 1  |  SIMPLE       | inventory  | ref   | idx_fk_film_id,idx_store_id_film_id    | idex_store_id_film_id  |  1  | const        | 1145            |Using index condition

- 複合インデックスになっているstore_idとfilm_idを使い絞込みをかけている。  
### 複数行のテーブル情報が返ってくる例  
#### 1.サブクエリを使用した例
```SQL
EXPLAIN SELECT film.title,  
	(SELECT name FROM language WHERE language_id = film.language_id)
FROM film  
WHERE film_id < 100;
```  
| id | select_type   | table |  type | possible_keys| key          | key_len      | ref          | rows         | extra       |
|:--:|:------------: |:-----:|:-----:|:------------:|:------------:|:------------:|:------------:|:------------:|:-----------:|
| 1  | PRIMARY      | film  | range   |PRYMARY       | PRYMARY         |   2       | NULL        | 98      |     Using where       |
| 2  | DEPENDENT SUBQUERY  | language  | eq_ref   | PRYMARY    | PRYMARY    |   1      | astrskdb.film.language_id | 1     |          |

- select_typeがDEPENDENT SUBQUERYとなっている。これは依存性のあるサブクエリを1度だけ呼び出したという意味である。
#### 2.JOINを使用した例
```SQL
EXPLAIN SELECT 
	film.title, language.name 
FROM film 
INNER JOIN language
ON language.language_id = film.language_id
WHERE film.film_id < 100;
```
| id | select_type   | table |  type | possible_keys| key          | key_len      | ref          | rows         | extra       |
|:--:|:------------: |:-----:|:-----:|:------------:|:------------:|:------------:|:------------:|:------------:|:-----------:|
| 1  | SIMPLE  | film  | range   |PRYMARY,idx_fk_languade_id | PRYMARY |  2 | NULL       | 98      |     Using where       |
| 2  | SIMPLE  | language | ALL | PRYMARY    |   NULL     | NULL | NULL  | 6 | Using where; Using join buffer(flat,BNL join)|

- サブクエリを使用した例と同じ検索結果になるクエリを使用した。languageテーブルがフルテーブルスキャンとなった。
### 2つのSQL文の実行計画の違いについて
- 二つのSQLで結果がことなるのはなぜか。
```SQL
EXPLAIN SELECT * FROM film_actor ORDER BY actor_id, film_id;
```
| id | select_type   | table |  type | possible_keys| key          | key_len      | ref          | rows         | extra       |
|:--:|:------------: |:-----:|:-----:|:------------:|:------------:|:------------:|:------------:|:------------:|:-----------:|
| 1  | SIMPLE  | film_actor  | index   | NULL | PRYMARY |  4 | NULL | 5920   |   |

```SQL
EXPLAIN SELECT * FROM film_actor ORDER BY film_id, actor_id;
```
| id | select_type   | table |  type | possible_keys| key          | key_len      | ref          | rows         | extra       |
|:--:|:------------: |:-----:|:-----:|:------------:|:------------:|:------------:|:------------:|:------------:|:-----------:|
| 1  | SIMPLE  | film_actor  | ALL  | NULL | NULL |  NULL | NULL | 5131   | Using filesort

- インデックス情報を確認する。
```SQL
SHOW INDEX FROM film_actor
```
| table | Non_unique | Key_name | Seq_in_index | Columun_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment| Index_comment |
|:--:|:------------: |:-----:|:-----:|:------------:|:------------:|:------------:|:------------:|:------------:|:-----------:|:-----------:|:-----------:|:-----------:|
| film_actor | 0 | PRIMARY | 1 | actor_id | A | 5131 | null | null | | BTREE | | | 
| film_actor | 0 | PRIMARY | 2 | film_id | A | 5131 | null | null | | BTREE | | | 
| film_actor | 1 | idx_fk_film_id | 1 | film_id | A | 5131 | null | null | | BTREE | | | 

- 2行目のカラム名がfilm_idのSeq_in_indexを確認すると、インデックスのフィールド番号が2となっている。これにより、actor_idとfilm_idは結合インデックスであるとわかる。インデックスによる検索はactor_idを元にしないとfilm_idが分からないということなので、film_idを優先するとactor_idはインデックスからは分からない。このため、film_idを優先したクエリはフルテーブルスキャンになっている。

### 処理が遅いクエリ  
- Cardinalityが高いカラムに対してインデックスを貼り付けてないときに、そのカラムを集計する(特に、order byで並び替えをする)と、処理は遅くなる。  
- 複合インデックスを使用しているが、絞込みにカラム名を片方しか入れていない場合には、インデックスがフルスキャンされるため、処理は遅くなる。  
- 条件検索で、インデックスを張ったカラムに直接に関数や計算式を指定するとインデックスはフルスキャンされてしまう。

