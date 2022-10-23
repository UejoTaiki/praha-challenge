環境：MySQL

## テーブルを作成

```sql
CREATE TABLE `Student` (
  `student_id` int PRIMARY KEY,
  `name` varchar(255),
  `status` varchar(255),
  CONSTRAINT `status_check` CHECK ((`status` in ('studying','graduated','suspended')))
);
```

## 動作確認

```sql
mysql> INSERT INTO `Student` VALUES (1,"Takahashi","studying");
> Query OK, 1 row affected (0.01 sec)
```

```sql
mysql> INSERT INTO `Student` VALUES (2,"Ito","graduated");
> Query OK, 1 row affected (0.01 sec)
```

```sql
-- LOA（Leave of absense(休学)）という制約外の値をインサートしてみる
mysql> INSERT INTO `Student` VALUES (3,"Takahashi","LAO");
> Error Code: 3819. Check constraint 'status_check' is violated.
```

ちゃんとエラーが出た。

## 取りうる値を調べる・取得する場合

メタ情報として管理してしまっているため、ドロップダウンなどで取りうる値を一覧で取得したいというユースケースに対応できない。

```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.CHECK_CONSTRAINTS;
+--------------------+-------------------+-------------------------+----------------------------------------------------------------------------------------+
| CONSTRAINT_CATALOG | CONSTRAINT_SCHEMA | CONSTRAINT_NAME         | CHECK_CLAUSE                                                                           |
+--------------------+-------------------+-------------------------+----------------------------------------------------------------------------------------+
| def                | test_schema       | status_check            | (`status` in (_utf8mb4\'studying\',_utf8mb4\'graduated\',_utf8mb4\'suspended\'))       |
+--------------------+-------------------+-------------------------+----------------------------------------------------------------------------------------+
1 rows in set (0.01 sec)
```

- INFORMATION_SCHEMA について

  > INFORMATION_SCHEMA では、データベースメタデータへのアクセスを実現し、データベースまたはテーブルの名前、カラムのデータ型、アクセス権限などの MySQL Server に関する情報を提供します。 この情報に使用されることがある別の用語が、データディクショナリとシステムカタログです。
  > https://dev.mysql.com/doc/refman/8.0/ja/information-schema-introduction.html

- CHECK_CONSTRAINTS テーブルについて
  > CHECK_CONSTRAINTS テーブル (MySQL 8.0.16 で使用可能) は、テーブルに定義されている CHECK 制約に関する情報を提供します。
  > https://dev.mysql.com/doc/refman/8.0/ja/information-schema-check-constraints-table.html

## CHECK 制約の追加や更新する場合

既存のCHECK 制約である`status_check`に`LAO`を追加したい場合など、貼り直すしかなさそう。


1. 対象の CHECK 制約を削除する

   ```sql
   mysql> ALTER TABLE Student DROP CHECK status_check;
   > 0 row(s) affected Records: 0  Duplicates: 0  Warnings: 0	0.238 sec
   ```

2. 再度、`LAO`を含めて追加する
   ```sql
   mysql> ALTER TABLE Student ADD CONSTRAINT `status_check` CHECK ((`status` in ('studying','graduated','suspended','LAO')));
   > 2 row(s) affected Records: 2  Duplicates: 0  Warnings: 0	0.085 sec
   ```
