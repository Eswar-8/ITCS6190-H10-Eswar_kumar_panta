# Hands-on L10: Report

**Name:**
**Student ID:**
**Email:**

---

## Seed and commands

Seed used for `datagen.py`: 801505751

The commands you ran, in order. If you deviated from the steps in the README, say where and
why.

```bash
python3 datagen.py 801505751
docker compose -f docker-compose.codespaces.yml up -d
docker cp main.py spark-master:/opt/spark/work-dir/

docker exec -it spark-master /opt/spark/bin/spark-submit \
  --master spark://spark-master:7077 \
  /opt/spark/work-dir/main.py task1 /opt/spark/work-dir/shared

cat shared-folder/output/task1/*.csv | grep -vc trip_id

docker cp main.py spark-master:/opt/spark/work-dir/

docker exec -it spark-master /opt/spark/bin/spark-submit \
  --master spark://spark-master:7077 \
  /opt/spark/work-dir/main.py task2 /opt/spark/work-dir/shared

docker exec -it spark-master /opt/spark/bin/spark-submit \
  --master spark://spark-master:7077 \
  /opt/spark/work-dir/main.py task2 /opt/spark/work-dir/shared update
```

---

## Results

### Task 1: ingest and parse

Paste the micro-batch summary printed at the end of the run, and the number of rows in
`shared-folder/output/task1` (`cat shared-folder/output/task1/*.csv | grep -vc trip_id`).

```
=== task1: micro-batch summary ===
batch input rows            watermark state rows
    0         24                    -          -
    1         23                    -          -
    2         24                    -          -
    3         21                    -          -
    4         29                    -          -
    5         29                    -          -
    6         28                    -          -
    7         22                    -          -
    8         24                    -          -
    9         21                    -          -
   10         23                    -          -
   11         26                    -          -

   number of rows in shared-folder/output/task1 294
```

How many micro-batches ran, and why that number?

### Task 2: trips and fares per window

Paste the windows written by the stream (`cat shared-folder/output/task2/*.csv | grep -v
window_start | sort`), the summary, and the batch table that `main.py` prints at the end.

```
=== task2: micro-batch summary ===
batch input rows            watermark state rows
    0         24                    -          2
    1         23  2026-10-27 07:58:50          2
    2         24  2026-10-27 07:59:53          3
    3         21  2026-10-27 08:00:46          2
    4         29  2026-10-27 08:01:57          3
    5         29  2026-10-27 08:02:57          2
    6         28  2026-10-27 08:03:48          3
    7         22  2026-10-27 08:04:47          2
    8         24  2026-10-27 08:05:48          3
    9         21  2026-10-27 08:06:50          2
   10         23  2026-10-27 08:07:53          3
   11         26  2026-10-27 08:08:51          2
   12          0  2026-10-27 08:09:54          2

=== task2: the same windows computed in batch over all files ===
+-------------------+-------------------+-----+----------+
|window_start       |window_end         |trips|total_fare|
+-------------------+-------------------+-----+----------+
|2026-10-27 07:58:00|2026-10-27 08:00:00|4    |176.82    |
|2026-10-27 08:00:00|2026-10-27 08:02:00|51   |2350.95   |
|2026-10-27 08:02:00|2026-10-27 08:04:00|52   |2015.4    |
|2026-10-27 08:04:00|2026-10-27 08:06:00|57   |2206.26   |
|2026-10-27 08:06:00|2026-10-27 08:08:00|51   |2431.99   |
|2026-10-27 08:08:00|2026-10-27 08:10:00|40   |1630.77   |
|2026-10-27 08:10:00|2026-10-27 08:12:00|39   |1626.05   |
+-------------------+-------------------+-----+----------+
```

---

## Questions

**1. Append versus update.** Run task 2 a second time with `update` (it prints to the
console). In which batch does each window first appear in update mode, and in which batch
is it written in append mode? Explain the difference with the watermark values from the
summary.



**2. Where did the rows go?** Compare the stream's windows with the batch table. Which
windows have fewer trips in the stream, by how many, and why? Which windows are missing
from the stream's output altogether, and what would make Spark write them? (Look at
`datagen.py` for how the late events were made.)



**3. Checkpoint.** Run task 1 again without deleting `shared-folder/checkpoint/task1`. What
happens, and why? List the subfolders of the checkpoint and say in one line what each one
holds.



---

## Problems and fixes

Anything that went wrong and what resolved it. Paste the actual error message. If nothing
went wrong, say so.
