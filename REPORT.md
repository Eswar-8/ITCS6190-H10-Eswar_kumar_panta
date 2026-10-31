  # Hands-on L10: Report
  
  **Name:** Eswar kumar panta
  
  **Student ID:** 801505751
  
  **Email:** epanta@charlotte.edu
  
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
  ``12 micro-batches ran: the stream folder holds 12 files and maxFilesPerTrigger = 1 makes each file one batch. The batch input rows add up to 294, the same as the rows written. Task 1 is stateless, so every row is final on arrival and none are dropped.``
  
  ### Task 2: trips and fares per window
  
  Paste the windows written by the stream (`cat shared-folder/output/task2/*.csv | grep -v
  window_start | sort`), the summary, and the batch table that `main.py` prints at the end.
  
  ```
  2026-10-27 07:58:00,2026-10-27 08:00:00,4,176.82
  2026-10-27 08:00:00,2026-10-27 08:02:00,47,2152.43
  2026-10-27 08:02:00,2026-10-27 08:04:00,49,1848.29
  2026-10-27 08:04:00,2026-10-27 08:06:00,57,2206.26
  2026-10-27 08:06:00,2026-10-27 08:08:00,51,2431.99
  
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
  ```
In update mode a window is printed in the batch where its first event arrives, and again in every later batch that changes it. The windows 07:58 and 08:00 first appear in batch 0, 08:02 in batch 2, 08:04 in batch 4, 08:06 in batch 6, 08:08 in batch 8 and 08:10 in batch 10. This matches the state-rows column: it rises to 3 in batches 2, 4, 6, 8 and 10, when a new window opens, and falls back to 2 in batches 3, 5, 7, 9 and 11, when a closed window is evicted. In update mode the value keeps changing until the window is final, for example 08:00–08:02 goes 20, 43, 47 trips in batches 0, 1 and 2, and 08:04–08:06 goes 21, 50, 55, 57 in batches 4 to 7. In append mode only the final row is written.
In append mode a window is written only once, in the first batch where the watermark has passed its end. The watermark is the largest event_time seen so far minus 2 minutes, so it reaches a window's end only about two minutes of event time after the window closes. From the summary: 07:58–08:00 is written in batch 3 (watermark 08:00:46), 08:00–08:02 in batch 5 (08:02:57), 08:02–08:04 in batch 7 (08:04:47), 08:04–08:06 in batch 9 (08:06:50) and 08:06–08:08 in batch 11 (08:08:51). The 07:58 window is written 3 batches after it first appears in update mode, and every other window 5 batches after (08:00: batch 0 → 5, 08:02: 2 → 7, 08:04: 4 → 9, 08:06: 6 → 11). It is written with its final value, while update mode shows intermediate values that keep changing. The windows 08:08–08:10 and 08:10–08:12 are never written in append mode, because the final watermark (08:09:54) never reaches 08:10.
```
  
  
  
  **2. Where did the rows go?** Compare the stream's windows with the batch table. Which
  windows have fewer trips in the stream, by how many, and why? Which windows are missing
  from the stream's output altogether, and what would make Spark write them? (Look at
  `datagen.py` for how the late events were made.)
  ```
  Two windows have fewer trips in the stream than in the batch table: 08:00–08:02 has 47 instead of 51 (4 fewer, fare 2152.43 instead of 2350.95) and 08:02–08:04 has 49 instead of 52 (3 fewer, fare 1848.29 instead of 2015.40). That is 7 trips, exactly the 7 events that datagen.py made "very late" (8 to 10 minutes before arrival, from file 9 on). They arrive in batches 9 to 11, when the watermark is already 08:06:50 to 08:08:51, so their windows (ending 08:02 and 08:04) are closed and their state is gone. Spark drops them silently. The windows 07:58–08:00 (4 trips), 08:04–08:06 (57) and 08:06–08:08 (51) are identical to the batch table. The 22 events that are only 1 to 2 minutes late are not lost: their windows are still open when they arrive, so they are counted.

The windows 08:08–08:10 (40 trips in the batch table) and 08:10–08:12 (39 trips) are missing from the stream's output. They are still open at the end, because the final watermark 08:09:54 is before 08:10:00, so append mode has not written them (update mode did print them, 40 and 39 trips, in batch 11). Spark would write them once the watermark passes their ends, which needs events with an event_time of at least 08:12:00 and 08:14:00, for instance a 13th file one minute later.
  ```
  
  
  
  
  **3. Checkpoint.** Run task 1 again without deleting `shared-folder/checkpoint/task1`. What
  happens, and why? List the subfolders of the checkpoint and say in one line what each one
  holds.
  ```
  Running task 1 again without deleting shared-folder/checkpoint/task1 ran one batch (batch 12) with 0 input rows and wrote nothing new. The output still holds the same 294 rows, with no duplicates. The checkpoint records which input files were already consumed and which batches were committed. On restart Spark compares the stream folder with that record, finds all 12 files already processed, and stops after an empty batch.

Subfolders of shared-folder/checkpoint/task1:

sources/: the state of the file source, i.e. which input files it has listed and read.
offsets/: one file per batch, written before the batch runs, saying which files that batch will process.
commits/: one marker per batch that finished and was written to the sink. Spark uses it to know which batches are complete.
metadata: a small file (not a folder) with the id of the query.
  ```
  
  ---
  
  ## Problems and fixes
  
  Anything that went wrong and what resolved it. Paste the actual error message. If nothing
  went wrong, say so.
  
  no problems encountered
