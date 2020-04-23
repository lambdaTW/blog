+++
title = "Create a GIN index with Django on AWS RDS"
author = "lambda@lambda.tw"
categories = ["Django"]
tags = ["Index", "postgres", "Django", "AWS", "RDS"]
date = "2019-04-26"
description = ""
featured = ""
featuredalt = ""
featuredpath = ""
linktitle = ""
type = "post"
+++
## GIN

### What is it
GIN 是一種 INDEX 可以幫助加速全文搜索的速度

> GIN stands for Generalized Inverted Index. GIN is designed for handling cases where the items to be indexed are composite values, and the queries to be handled by the index need to search for element values that appear within the composite items. For example, the items could be documents, and the queries could be searches for documents containing specific words.

### Normal SQL

在傳統 SQL 下可以用以下幾個步驟完成建立 GIN INDEX

- Install gin extension

{{< highlight postgresql >}}
CREATE EXTENSION IF NOT EXISTS pg_trgm;
{{< /highlight >}}

- Create index for table's column
{{< highlight postgresql >}}
CREATE INDEX <index_name>
    ON <schema_name>.<table_name> USING gin
    (<column_name>)
    TABLESPACE pg_default;
{{< /highlight >}}

#### Special type
但是如果你是特殊的欄位，例如：varchar、text，此時你就必須要給它特定的 operator 才能建立

##### Create GIN INDEX for varchar column

- Use gin_trgm_ops as operator

{{< highlight postgresql >}}
CREATE INDEX <index_name>
    ON <schema_name>.<table_name> USING gin
    (<column_name COLLATE pg_catalog."default" gin_trgm_ops)
    TABLESPACE pg_default;
{{< /highlight >}}

##### Or, you can set `gin_trgm_ops` as default

- Set default operator class

{{< highlight postgresql >}}
UPDATE pg_opclass SET opcdefault = true WHERE opcname='gin_trgm_ops';
{{< /highlight >}}

- Create the index like other types of column

{{< highlight postgresql >}}
CREATE INDEX <index_name>
    ON <schema_name>.<table_name> USING gin
    (<column_name>)
    TABLESPACE pg_default;
{{< /highlight >}}
### Use Django Postgres contribution library
對於 PostgreSQL 有較完善的 Django 對於 GIN INDEX 也是有支援的，所以你可以在 `models.py` 直接使用它

{{< highlight python3 >}}
from django.db import models
from django.contrib.postgres.fields import JSONField
from django.contrib.postgres.indexes import GinIndex


class Post(models.Model):
    content_segment = JSONField(default=list)

    class Meta:
        indexes = [GinIndex(fields=['content_segment'])]
{{< /highlight >}}

### The char field in Django

在 Django 中 Char、Text 等 `varchar` 類型的欄位要使用 GIN 和原生 SQL 一樣需要去設定需要使用的 `operator`， 產出 migration file, 並且 migrate 以後你應該會看到類似下面的錯誤

```
ERROR: data type character varying has no default operator class for access method "gin"
```

我們很簡單的可以在 Add Index 前加上設定 default operator 去 by pass

{{< highlight python3 >}}
import django.contrib.postgres.indexes
from django.db import migrations, models


class Migration(migrations.Migration):

    dependencies = [
        ('posts', '....'),
    ]

    operations = [
        migrations.RunSQL([
            "UPDATE pg_opclass SET opcdefault = true WHERE opcname='gin_trgm_ops';",
        ]),
        migrations.AddIndex(
            model_name='post',
            index=django.contrib.postgres.indexes.GinIndex(fields=['title'], name='posts_po_title_374d31_gin'),
        )
    ]

{{< /highlight >}}

你應該會看到下面的錯誤

```
django.db.utils.ProgrammingError: operator class "gin_trgm_ops" does not exist for access method "gin"
```

表示你家的 PostgreSQL 沒有安裝 GIN 的套件，這很簡單，只需要改改 migration file 就好

{{< highlight python3 >}}
import django.contrib.postgres.indexes
from django.db import migrations, models


class Migration(migrations.Migration):

    dependencies = [
        ('posts', '....'),
    ]

    operations = [
        migrations.RunSQL([
            "CREATE EXTENSION IF NOT EXISTS pg_trgm;",
            "UPDATE pg_opclass SET opcdefault = true WHERE opcname='gin_trgm_ops';"
        ]),
        migrations.AddIndex(
            model_name='post',
            index=django.contrib.postgres.indexes.GinIndex(fields=['title'], name='posts_po_title_374d31_gin'),
        )
    ]
{{< /highlight >}}

再次 migrate 相信你已經成功了



## AWS RDS is secure than your local DB server

AWS RDS 預設不會給你 superuser 權限，所以你沒有辦法直接執行

{{< highlight postgresql >}}
UPDATE pg_opclass SET opcdefault = true WHERE opcname='gin_trgm_ops';
{{< /highlight >}}

這會讓你在 Django migrate 時看到以下錯誤

```
permission denied for relation pg_opclass
```

### Fix it

記得我們在最一開始如何使用免設定預設 `operator` 就產生了一個 GIN 的 index 嗎？

如法炮製我們直接把 `operator` 插入在 CREATE INDEX 的 SQL 就可以達到了

#### Source code tour

- GINIndex forefathers
  我們可以在發現 SQL statement 是由 `schema_editor` 的 `_create_index_sql` 產生出來的

{{< highlight python3 >}}
  # django/contrib/postgres/indexes.py
  from django.db.models import Index


  class PostgresIndex(Index):
      # ...


  class GinIndex(PostgresIndex):
      def create_sql(self, model, schema_editor, using=''):
          statement = super().create_sql(model, schema_editor, using=' USING %s' % self.suffix)
          with_params = self.get_with_params()
          if with_params:
              statement.parts['extra'] = 'WITH (%s) %s' % (
                  ', '.join(with_params),
                  statement.parts['extra'],
              )
          return statement


  # django/db/models/indexes.py
  class Index:
      def create_sql(self, model, schema_editor, using=''):
          fields = [model._meta.get_field(field_name) for field_name, _ in self.fields_orders]
          col_suffixes = [order[1] for order in self.fields_orders]
          return schema_editor._create_index_sql(
              model, fields, name=self.name, using=using, db_tablespace=self.db_tablespace,
              col_suffixes=col_suffixes,
          )
{{< /highlight >}}

- How `schema_edit` create SQL statement

{{< highlight python3 >}}
  # django/db/backends/postgresql/schema.py
  from django.db.backends.base.schema import BaseDatabaseSchemaEditor


  class DatabaseSchemaEditor(BaseDatabaseSchemaEditor):
      # ...
      sql_create_index = "CREATE INDEX %(name)s ON %(table)s%(using)s (%(columns)s)%(extra)s"


  # schema.py
  class BaseDatabaseSchemaEditor:
      def _create_index_sql(self, model, fields, *, name=None, suffix='', using='',
                            db_tablespace=None, col_suffixes=(), sql=None):
          """
          Return the SQL statement to create the index for one or several fields.
          `sql` can be specified if the syntax differs from the standard (GIS
          indexes, ...).
          """
          tablespace_sql = self._get_index_tablespace_sql(model, fields, db_tablespace=db_tablespace)
          columns = [field.column for field in fields]
          sql_create_index = sql or self.sql_create_index
          table = model._meta.db_table

          def create_index_name(*args, **kwargs):
              nonlocal name
              if name is None:
                  name = self._create_index_name(*args, **kwargs)
              return self.quote_name(name)

          return Statement(
              sql_create_index,
              table=Table(table, self.quote_name),
              name=IndexName(table, columns, suffix, create_index_name),
              using=using,
              columns=Columns(table, columns, self.quote_name, col_suffixes=col_suffixes),
              extra=tablespace_sql,
          )
{{< /highlight >}}

- Override the `create_sql`

{{< highlight python3 >}}
  from django.contrib.postgres.indexes import GinIndex


  class CharGinIndex(GinIndex):

      def create_sql(self, model, schema_editor, using=''):
          assert len(self.fields_orders) == 1
          original_sql = schema_editor.sql_create_index
          schema_editor.sql_create_index = 'CREATE INDEX %(name)s ON %(table)s%(using)s (%(columns)s COLLATE pg_catalog."default" gin_trgm_ops)%(extra)s'
          statement = super().create_sql(model, schema_editor, using)
          schema_editor.sql_create_index = original_sql
          return statement
{{< /highlight >}}

我們在 `create_sql` 上面覆寫掉 `schema_editor` 的 `sql_create_index` 語法，並且在呼叫完 `create_sql` 以後把它還原（因為一次 migrate 中 schema editor 會被重複使用，若沒有還原其他在同一次 migrate 中使用到相同 schema editor 的就會被影響)

## Conclusion
在這邊用比較髒的方式處理了這個問題，主要是因為在 `BaseDatabaseSchemaEditor` 有以下方式可以更改 `sql` 參數就可以達到這功能，但是在 `Index` 類別並沒有把此參數讓我們可以丟進去，由於不會影響功能，暫時就不重複造輪子。

{{< highlight python3 >}}
class BaseDatabaseSchemaEditor:
    def _create_index_sql(self, model, fields, *, name=None, suffix='', using='',
                          db_tablespace=None, col_suffixes=(), sql=None):
		# ...
        sql_create_index = sql or self.sql_create_index
		# ...
{{< /highlight >}}

