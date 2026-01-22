# TPC-H Data Generator CLI

`tpchgen-cli` is a high-performance, parallel TPC-H data generator command line
tool

This tool is more than 10x faster than the next fastest TPCH generator we know
of (`duckdb`). On a 2023 Mac M3 Max laptop, it easily generates data faster than
can be written to SSD. See [BENCHMARKS.md] for more details on performance and
benchmarking.

[BENCHMARKS.md]: https://github.com/clflushopt/tpchgen-rs/blob/main/benchmarks/BENCHMARKS.md

* See the tpchgen [README.md](https://github.com/clflushopt/tpchgen-rs) for
project details
* Watch this [awesome demo](https://www.youtube.com/watch?v=UYIC57hlL14)  by
[@alamb](https://github.com/alamb) to see `tpchgen-cli` in action
* Read the companion blog post in the
[Datafusion
blog](https://datafusion.apache.org/blog/2025/04/10/fastest-tpch-generator/) to learn about the project's history
* Try it yourself by following the instructions below

## Install via `pip`

```shell
pip install tpchgen-cli
```

## Install via `uv`

```shell
uv tool install tpchgen-cli 
```

## Install via Rust

[Install Rust](https://www.rust-lang.org/tools/install) and compile

```shell
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
RUSTFLAGS='-C target-cpu=native' cargo install tpchgen-cli
```

## Examples

```shell
# Scale Factor 10, all tables, in Apache Parquet format in the current directory
# (3.6GB, 8 files, 60M lineitem rows, in 5 seconds on a modern laptop)
tpchgen-cli -s 10 --format=parquet

# Scale Factor 10, all tables, in `tbl`(csv like) format in the `sf10` directory
# (10GB, 8 files, 60M lineitem rows)
tpchgen-cli -s 10 --output-dir sf10

Options:
  -s, --scale-factor <SCALE_FACTOR>
          Scale factor to address (default: 1) [default: 1]
  -o, --output-dir <OUTPUT_DIR>
          Output directory for generated files (default: current directory) [default: .]
  -T, --tables <TABLES>
          Which tables to generate (default: all) [possible values: region, nation, supplier, customer, part, partsupp, orders, lineitem]
  -p, --parts <PARTS>
          Number of parts to generate (manual parallel generation) [default: 1]
      --part <PART>
          Which part to generate (1-based, only relevant if parts > 1) [default: 1]
  -f, --format <FORMAT>
          Output format: tbl, csv, parquet (default: tbl) [default: tbl] [possible values: tbl, csv, parquet]
  -n, --num-threads <NUM_THREADS>
          The number of threads for parallel generation, defaults to the number of CPUs [default: 8]
  -c, --parquet-compression <PARQUET_COMPRESSION>
          Parquet block compression format. Default is SNAPPY [default: SNAPPY]
      --parquet-row-group-bytes <PARQUET_ROW_GROUP_BYTES>
          Target parquet row group size in bytes [default: 7340032]
  -v, --verbose
          Verbose output (default: false)
      --stdout
          Write the output to stdout instead of a file
  -h, --help
          Print help (see more with '--help')
```

For example generating a dataset with a scale factor of 1 (1GB) can be done like this:
```shell
$ tpchgen-cli -s 1 --output-dir=/tmp/tpch
```

## Expected sizes

This table shows the expected sizes (table and row group) you should expect to see:

| scale | table | file_count | row_group_count | total_bytes | avg_bytes | min_bytes | max_bytes | total_rows | avg_rows | min_rows | max_rows | rows_per_partition |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 100 | customer | 1 | 15 | 2464649436 | 164309962.4 | 164242867 | 164353122 | 15000000 | 1000000 | 1000000 | 1000000 | 15000000 |
| 100 | lineitem | 6 | 605 | 41388102080 | 68410086.08 | 453787 | 69011596 | 600037902 | 991798.1851 | 5547 | 1000000 | 100006317 |
| 100 | nation | 1 | 1 | 2795 | 2795 | 2795 | 2795 | 25 | 25 | 25 | 25 | 25 |
| 100 | orders | 2 | 150 | 14729052530 | 98193683.53 | 98142541 | 98228417 | 150000000 | 1000000 | 1000000 | 1000000 | 75000000 |
| 100 | part | 1 | 20 | 1361754585 | 68087729.25 | 68075017 | 68102511 | 20000000 | 1000000 | 1000000 | 1000000 | 20000000 |
| 100 | partsupp | 1 | 80 | 11718565215 | 146482065.2 | 146366336 | 146581750 | 80000000 | 1000000 | 1000000 | 1000000 | 80000000 |
| 100 | region | 1 | 1 | 728 | 728 | 728 | 728 | 5 | 5 | 5 | 5 | 5 |
| 100 | supplier | 1 | 1 | 153939798 | 153939798 | 153939798 | 153939798 | 1000000 | 1000000 | 1000000 | 1000000 | 1000000 |
| 300 | customer | 1 | 45 | 7394342431 | 164318720.7 | 164242867 | 164379012 | 45000000 | 1000000 | 1000000 | 1000000 | 45000000 |
| 300 | lineitem | 18 | 1810 | 124224906537 | 68632545.05 | 62727 | 69050438 | 1799989091 | 994469.111 | 672 | 1000000 | 99999393.94 |
| 300 | nation | 1 | 1 | 2795 | 2795 | 2795 | 2795 | 25 | 25 | 25 | 25 | 25 |
| 300 | orders | 5 | 450 | 44338638901 | 98530308.67 | 98478923 | 98587235 | 450000000 | 1000000 | 1000000 | 1000000 | 90000000 |
| 300 | part | 3 | 60 | 4085286046 | 68088100.77 | 68070503 | 68104676 | 60000000 | 1000000 | 1000000 | 1000000 | 20000000 |
| 300 | partsupp | 1 | 240 | 35155157469 | 146479822.8 | 146366336 | 146583230 | 240000000 | 1000000 | 1000000 | 1000000 | 240000000 |
| 300 | region | 1 | 1 | 728 | 728 | 728 | 728 | 5 | 5 | 5 | 5 | 5 |
| 300 | supplier | 1 | 3 | 461828684 | 153942894.7 | 153924960 | 153963926 | 3000000 | 1000000 | 1000000 | 1000000 | 3000000 |
| 1000 | customer | 2 | 150 | 24648739689 | 164324931.3 | 164242867 | 164379012 | 150000000 | 1000000 | 1000000 | 1000000 | 75000000 |
| 1000 | lineitem | 60 | 6031 | 414158196610 | 68671563.03 | 44290 | 69063139 | 5999989709 | 994858.1842 | 474 | 1000000 | 99999828.48 |
| 1000 | nation | 1 | 1 | 2795 | 2795 | 2795 | 2795 | 25 | 25 | 25 | 25 | 25 |
| 1000 | orders | 15 | 1500 | 147907255618 | 98604837.08 | 98541474 | 98660235 | 1500000000 | 1000000 | 1000000 | 1000000 | 100000000 |
| 1000 | part | 2 | 200 | 13617002848 | 68085014.24 | 68061168 | 68104676 | 200000000 | 1000000 | 1000000 | 1000000 | 100000000 |
| 1000 | partsupp | 8 | 800 | 117184152586 | 146480190.7 | 146333837 | 146620954 | 800000000 | 1000000 | 1000000 | 1000000 | 100000000 |
| 1000 | region | 1 | 1 | 728 | 728 | 728 | 728 | 5 | 5 | 5 | 5 | 5 |
| 1000 | supplier | 1 | 10 | 1539348521 | 153934852.1 | 153894686 | 153971069 | 10000000 | 1000000 | 1000000 | 1000000 | 10000000 |
| 3000 | customer | 5 | 450 | 73946286395 | 164325080.9 | 164242867 | 164410288 | 450000000 | 1000000 | 1000000 | 1000000 | 90000000 |
| 3000 | lineitem | 180 | 18097 | 1242580943167 | 68662261.32 | 25649 | 69071159 | 18000048306 | 994642.6649 | 263 | 1000000 | 100000268.4 |
| 3000 | nation | 1 | 1 | 2795 | 2795 | 2795 | 2795 | 25 | 25 | 25 | 25 | 25 |
| 3000 | orders | 45 | 4500 | 443807441442 | 98623875.88 | 98559568 | 98681097 | 4500000000 | 1000000 | 1000000 | 1000000 | 100000000 |
| 3000 | part | 6 | 600 | 40849459152 | 68082431.92 | 68061168 | 68104676 | 600000000 | 1000000 | 1000000 | 1000000 | 100000000 |
| 3000 | partsupp | 24 | 2400 | 351552019040 | 146480007.9 | 146324064 | 146620954 | 2400000000 | 1000000 | 1000000 | 1000000 | 100000000 |
| 3000 | region | 1 | 1 | 728 | 728 | 728 | 728 | 5 | 5 | 5 | 5 | 5 |
| 3000 | supplier | 1 | 30 | 4618366357 | 153945545.2 | 153894686 | 154000548 | 30000000 | 1000000 | 1000000 | 1000000 | 30000000 |
| 10000 | customer | 15 | 1500 | 246985449774 | 164656966.5 | 164237608 | 165388102 | 1500000000 | 1000000 | 1000000 | 1000000 | 100000000 |
| 10000 | lineitem | 600 | 60306 | 4141913902535 | 68681622.1 | 4494 | 69070485 | 59999994267 | 994925.783 | 33 | 1000000 | 99999990.45 |
| 10000 | nation | 1 | 1 | 2795 | 2795 | 2795 | 2795 | 25 | 25 | 25 | 25 | 25 |
| 10000 | orders | 150 | 15000 | 1479465043099 | 98631002.87 | 98566302 | 98690948 | 15000000000 | 1000000 | 1000000 | 1000000 | 100000000 |
| 10000 | part | 20 | 2000 | 136163213150 | 68081606.58 | 68058388 | 68106202 | 2000000000 | 1000000 | 1000000 | 1000000 | 100000000 |
| 10000 | partsupp | 80 | 8000 | 1171840155056 | 146480019.4 | 146311741 | 146631280 | 8000000000 | 1000000 | 1000000 | 1000000 | 100000000 |
| 10000 | region | 1 | 1 | 728 | 728 | 728 | 728 | 5 | 5 | 5 | 5 | 5 |
| 10000 | supplier | 1 | 100 | 15394661948 | 153946619.5 | 153888103 | 154002932 | 100000000 | 1000000 | 1000000 | 1000000 | 100000000 |
| 30000 | customer | 45 | 4500 | 742947540878 | 165099453.5 | 164237608 | 165414321 | 4500000000 | 1000000 | 1000000 | 1000000 | 100000000 |
| 30000 | lineitem | 1800 | 180917 | 12426034850522 | 68683622.05 | 2038 | 69078038 | 179999978268 | 994931.2573 | 8 | 1000000 | 99999987.93 |
| 30000 | nation | 1 | 1 | 2795 | 2795 | 2795 | 2795 | 25 | 25 | 25 | 25 | 25 |
| 30000 | orders | 450 | 45000 | 4438390248107 | 98630894.4 | 98565035 | 98695743 | 45000000000 | 1000000 | 1000000 | 1000000 | 100000000 |
| 30000 | part | 60 | 6000 | 408489743423 | 68081623.9 | 68056040 | 68107338 | 6000000000 | 1000000 | 1000000 | 1000000 | 100000000 |
| 30000 | partsupp | 240 | 24000 | 3515520097434 | 146480004.1 | 146311741 | 146645377 | 24000000000 | 1000000 | 1000000 | 1000000 | 100000000 |
| 30000 | region | 1 | 1 | 728 | 728 | 728 | 728 | 5 | 5 | 5 | 5 | 5 |
| 30000 | supplier | 3 | 300 | 46184029467 | 153946764.9 | 153881335 | 154006241 | 300000000 | 1000000 | 1000000 | 1000000 | 100000000 |
# Scale Factor 1000, lineitem table, in Apache Parquet format in sf1000 directory, 
# 20 part(ititons), 100MB row groups
# (220GB, 20 files, 6B lineitem rows, 3.5 minutes on a modern laptop)
tpchgen-cli -s 1000 --tables lineitem --parts 20 --format=parquet --parquet-row-group-bytes=100000000 --output-dir sf1000

# Override column encodings (e.g., use PLAIN instead of dictionary for specific columns)
tpchgen-cli -s 1 --format=parquet --column-encoding=l_quantity=PLAIN --column-encoding=l_orderkey=DELTA_BINARY_PACKED

# Scale Factor 10, partition 2 and 3 of 10 in sf10 directory
#
# partitioned/
# ├── lineitem
# │   ├── lineitem.2.tbl
# │   └── lineitem.3.tbl
# └── orders
#    ├── orders.2.tbl
#    └── orders.3.tbl
#     
for PART in `seq 2 3`; do
  tpchgen-cli --tables lineitem,orders --scale-factor=10 --output-dir partitioned --parts 10 --part $PART
done
```

## Performance

| Scale Factor | `tpchgen-cli` | DuckDB     | DuckDB (proprietary) |
| ------------ | ------------- | ---------- | -------------------- |
| 1            | `0:02.24`     | `0:12.29`  | `0:10.68`            |
| 10           | `0:09.97`     | `1:46.80`  | `1:41.14`            |
| 100          | `1:14.22`     | `17:48.27` | `16:40.88`           |
| 1000         | `10:26.26`    | N/A (OOM)  | N/A (OOM)            |

- DuckDB (proprietary) is the time required to create TPCH data using the
  proprietary DuckDB format
- Creating Scale Factor 1000 data in DuckDB [required 647 GB of memory](https://duckdb.org/docs/stable/extensions/tpch.html#resource-usage-of-the-data-generator),
  which is why it is not included in the table above.

Times to create TPCH tables in Parquet format using `tpchgen-cli` and `duckdb` for various scale factors.

