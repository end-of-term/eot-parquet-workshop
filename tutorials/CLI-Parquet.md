# Linux Utils

```
$ export LC_ALL=C
$ zcat EOTNL.cdxj.gz | head
$ zcat EOTNL.cdxj.gz | wc -l
$ zcat EOTNL.cdxj.gz | ./cdxj2cdx.py | head
$ zcat EOTNL.cdxj.gz | ./cdxj2cdx.py | gzip > EOTNL.cdx.gz
$ zcat EOTNL.cdx.gz | head
$ time zcat EOTNL.cdx.gz | cut -d' ' -f5 | sort | uniq -c | sort -nr
$ time zcat EOTNL.cdx.gz | awk '{a[$5]++}END{for(k in a){print k,a[k]}}' | sort -nrk2
```

Investigate Parque file

```
$ pip install parquet-cli
$ parq --help
$ parq EOTNL.parquet
$ parq EOTNL.parquet -s
```

# SQL Examples

The examples below use a tool called DuckDB-CLI that can be downloaded from this website - https://duckdb.org/docs/installation/

Once installed, to start your DuckDB session and get your command prompt, use the following command.

```
$ duckdb
```

## How many rows are in the database?

```sql
SELECT * FROM EOTNL.parquet;
```

**Add timer**

```
.timer on
```

**Now, let’s add count**

```sql
SELECT COUNT(*) FROM EOTNL.parquet;
```

**Write the output to a Markdown file**

```
.mode markdown
.output op.md
```

```sql
SELECT COUNT(*) AS count FROM EOTNL.parquet;
```

**Write the output to a JSON file**

```
.mode json
.output op.json
```

```sql
SELECT COUNT(*) AS count FROM EOTNL.parquet;
```

**Write back to the STDOUT**

```
.output
```

```sql
SELECT COUNT(*) AS count FROM EOTNL.parquet;
```

```
.quit
```

```
$ cat op.md
$ cat op.json
```

## What are the fields in this database?

```sql
DESCRIBE FROM EOTNL.parquet;
```

## How many unique URLs?

```sql
SELECT COUNT(url_surtkey) AS total, COUNT(DISTINCT url_surtkey) AS unique FROM EOTNL.parquet;
```

## What National Lab domains have the most content in the dataset

```sql
SELECT url_host_name FROM EOTNL.parquet LIMIT 10;
```

```sql
SELECT url_host_registered_domain FROM EOTNL.parquet LIMIT 10;
```

```sql
SELECT DISTINCT url_host_registered_domain FROM EOTNL.parquet LIMIT 10;
```

```sql
SELECT url_host_registered_domain AS domain, COUNT(url_host_registered_domain) AS count FROM EOTNL.parquet GROUP BY domain;
```

```sql
SELECT url_host_registered_domain AS domain, COUNT(url_host_registered_domain) AS count FROM EOTNL.parquet GROUP BY domain ORDER BY count DESC;
```

## What are the status code frequencies for the dataset?

```sql
SELECT fetch_status AS status, COUNT(fetch_status) AS count FROM EOTNL.parquet GROUP BY status ORDER BY count DESC;
```

```sql
SELECT fetch_status//100 AS status, COUNT(fetch_status) AS count FROM EOTNL.parquet GROUP BY status;
```

```sql
SELECT FORMAT('{}xx', fetch_status//100) AS status, COUNT(fetch_status) AS count FROM EOTNL.parquet GROUP BY status ORDER BY status;
```

## How many captures were made in each month?

```sql
SELECT YEAR(fetch_time) AS year, MONTH(fetch_time) AS month, COUNT(fetch_time) AS count FROM EOTNL.parquet GROUP BY year, month ORDER BY year, month;
```

## What are the frequencies of various mime types in the dataset?

```sql
SELECT content_mime_type AS mime, COUNT(content_mime_type) AS count FROM EOTNL.parquet GROUP BY mime ORDER BY count DESC;
```

**Show more lines**

```
.maxrows 200
```

```sql
SELECT content_mime_type AS mime, COUNT(content_mime_type) AS count FROM EOTNL.parquet GROUP BY mime ORDER BY count DESC;
```

## How many images are archived?

```sql
SELECT COUNT(content_mime_type) AS count FROM EOTNL.parquet WHERE STARTS_WITH(content_mime_type, 'image/');
```

## How does that differ from the detected mime types?

```sql
SELECT content_mime_type AS original, content_mime_detected AS detected, COUNT(content_mime_type) AS count FROM EOTNL.parquet WHERE original != detected GROUP BY original, detected ORDER BY count DESC LIMIT 100;
```

## What are the languages in the dataset?

```sql
SELECT content_languages AS lang, COUNT(content_languages) AS count FROM EOTNL.parquet GROUP BY lang ORDER BY count DESC;
```

This can be converted to an array as Parquet supports nested structured data.

## What are the character sets in the dataset?

```sql
SELECT content_charset AS charset, COUNT(content_charset) AS count FROM EOTNL.parquet GROUP BY charset ORDER BY count DESC;
```

## What domains have the most PDFs (based on detected mime types)?

```sql
SELECT url_host_registered_domain AS domain, COUNT(content_mime_detected) AS count FROM EOTNL.parquet WHERE content_mime_detected = 'application/pdf' GROUP BY domain ORDER BY count DESC;
```

## What National Lab has the most unique host names?

```sql
SELECT url_host_registered_domain AS domain, COUNT(DISTINCT url_host_name) AS count FROM EOTNL.parquet GROUP BY domain ORDER BY count DESC;
```

## Top hosts with largest numbers of 404

```sql
SELECT url_host_registered_domain AS domain, COUNT(fetch_status) AS count FROM EOTNL.parquet WHERE fetch_status == 404 GROUP BY domain ORDER BY count DESC;
```
