# splunk

## cluster errors
index="myindex"  fatal* OR err* OR exception* OR timeout* OR waiting* OR fail* OR unable* OR lock* OR block*

| cluster showcount=t t=0.7 labelonly=t | table _time cluster_count cluster_label _raw  | dedup 1 cluster_label | sort - cluster_count cluster_label _time  | chart values(cluster_count) as count by _raw  | sort limit=20 - count 
|rex "(?<_raw>[^\n]*)\n"
