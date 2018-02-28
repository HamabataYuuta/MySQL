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
`SHOW INDEX FROM テーブル名`    

SHOW INDEX構文は、テーブル名に対応したINDEXの情報のフィールドを返す。  
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
`CTEATE FROM テーブル名(カラム名 型,INDEX(カラム名))`  
- 既存のテーブルのINDEXを作成  
`CREATE INDEX インデックス名 ON テーブル名(カラム名)` 
カラム名の後にはソートを指定できる。  
### インデックスの削除  
- DROP INDEX構文を使用する。  
`DROP INDEX インデックス名 ON テーブル名`

### 複合インデックスの作成
- 複合インデックスとは、1つのテーブルの複数のカラムにINDEXを作成することである。 絞込みの際に複数の列をキーとして抽出する場合に有効である。
`CREATE INDEX インデックス名 ON テーブル名(カラム名,カラム名)`  

### ユニークインデックスについて　　
- ユニークインデックスとは、テーブルのレコード内に重複をさせないインデックスである。　　
- 主キーと違い、ユニークインデックスは複数のカラムにつけることができる。  

`CREATE UNIQUE INDEX インデックス名 ON テーブル名(カラム名1,カラム名2)`

### インデックスが利用されているかの調査(クエリの実行計画の調査)  
- EXPLAINコマンドを使用する。  
- EXPLAINコマンドとは、SELECT句で使用されるテーブルに関する情報を返す。使用する際には、SELECT文の前にEXPLAINを記述する。(利用しているMySQLサーバはversion5.5.52)  
- MySQL5.6.3からはSELECT文以外のDELETE、INSERT、REPLACE、UPDATEで使用可能である。
  
  
`EXPLAIN SELECT * FROM film WHERE title ='WORST BANGER';`  

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
### サブクエリにより複数行のテーブル情報が返ってくる例  
`EXPLAIN SELECT film.title,  
	(SELECT name FROM language WHERE language_id = film.language_id)
FROM film  
WHERE film_id < 100;`  

| id | select_type   | table |  type | possible_keys| key          | key_len      | ref          | rows         | extra       |
|:--:|:------------: |:-----:|:-----:|:------------:|:------------:|:------------:|:------------:|:------------:|:-----------:|
| 1  | PRIMARY      | film  | range   |PRYMARY       | PRYMARY         |   2       | null         | 98      |     Using where       |
| 2  | DEPENDENT SUBQUERY  | language  | eq_ref   | PRYMARY    | PRYMARY    |   1      | astrskdb.film.language_id | 1     |          |

##### サブクエリとして呼び出したlanguageテーブル
- select_typeがDEPENDENT SUBQUERYとなっている。これは依存性のあるサブクエリを1度だけ呼び出したという意味である。

### インデックスでの検索が効いていないパターンの例   
`EXPLAIN SELECT * FROM film;`  

| id | select_type   | table |  type | possible_keys| key          | key_len      | ref          | rows         | extra       |
|:--:|:------------: |:-----:|:-----:|:------------:|:------------:|:------------:|:------------:|:------------:|:-----------:|
| 1  |  SIMPLE       | film  | ALL   |null          | null         | null         | null         | 953          |             |

- typeがALLとなっており、テーブル全てを読み込んでいるためインデックスが利用されていないことを示している。  

### インデックスでの検索ができていない条件つきのクエリ  
`EXPLAIN SELECT * FROM inventory WHERE film_id < 500; `  

| id | select_type   | table |  type | possible_keys| key          | key_len      | ref          | rows         | extra       |
|:--:|:------------: |:-----:|:-----:|:------------:|:------------:|:------------:|:------------:|:------------:|:-----------:|
| 1  |  SIMPLE       | inventory  | ALL   |   idx_fk_film_id      | null         | null         | null         | 5007          |          Using where   |

- 複合インデックスとして登録されたインデックスを使用する際には、(カラム1,カラム2)で登録されているものならば、カラム1からの条件検索か、カラム1、カラム2と条件を順番に絞り込んだクエリでないとインデックスは機能しない。  そのため、カラム2のみの絞込みではインデックスは利用されない。

### インデックス全体を読み込んでいる例
`EXPLAIN SELECT title FROM film;`  

| id | select_type   | table |  type | possible_keys| key          | key_len      | ref          | rows         | extra       |
|:--:|:------------: |:-----:|:-----:|:------------:|:------------:|:------------:|:------------:|:------------:|:-----------:|
| 1  |  SIMPLE       | film  | index |null          | idx_title| 767         | null         | 953          |   Using index          |  

- typeがindexとなっており、インデックス全体を読み込んでいるため処理が遅い。  

### インデックスでの検索において効率の良い例  
`EXPLAIN SELECT * FROM film WHERE film_id < 100 AND title = 'A%';`  

| id | select_type   | table |  type | possible_keys| key          | key_len      | ref          | rows         | extra       |
|:--:|:------------: |:-----:|:-----:|:------------:|:------------:|:------------:|:------------:|:------------:|:-----------:|
| 1  |  SIMPLE       | film  | range |PRIMARY,idx_title    | PRIMARY | 2         | null         | 98       |   Using Where   |  

- Cardinalityが高い主キーとINDEXが作成されているCardinalityが高いfilmのタイトルから抽出した結果をマージしているため、INDEXを利用した検索として効率が良い。


### 複合インデックスの利用例

`EXPLAIN SELECT * FROM inventory WHERE store_id = 1 AND film_id < 500; `

| id | select_type   | table |  type | possible_keys| key          | key_len      | ref          | rows         | extra       |
|:--:|:------------: |:-----:|:-----:|:------------:|:------------:|:------------:|:------------:|:------------:|:------------:|
| 1  |  SIMPLE       | inventory  | ref   | idx_fk_film_id,idx_store_id_film_id    | idex_store_id_film_id  |  1  | const        | 1145            |Using index condition

- 複合インデックスになっているstore_idとfilm_idを使い絞込みをかけている。  

### 処理が遅いクエリ  
- Cardinalityが高いカラムに対してインデックスを貼り付けてないときに、そのカラムを集計する(特に、order byで並び替えをする)と、処理は遅くなる。  
- 複合インデックスを使用しているが、絞込みにカラム名を片方しか入れていない場合には、インデックスがフルスキャンされるため、処理は遅くなる。  
