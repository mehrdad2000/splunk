# splunk

## cluster errors (trim first lines for exception)
index="myindex"  fatal* OR err* OR exception* OR timeout* OR waiting* OR fail* OR unable* OR lock* OR block*

| cluster showcount=t t=0.7 labelonly=t | table _time cluster_count cluster_label _raw  | dedup 1 cluster_label | sort - cluster_count cluster_label _time  | chart values(cluster_count) as count by _raw  | sort limit=20 - count 
|rex "(?<_raw>[^\n]*)\n"

## Java Exceptions
  
| rex "Caused by:\s*(?P<Causedby>.*)"
| rex "(?<javaException>java[x]?\..*Exception)"
| rex "(?<javaException1>java[x]?\..*?Exception)"
| rex "(?<javaException2>java\..*Exception)"

| eval ExceptionName=coalesce(Causedby,javaException,javaException1,javaException2)

| fields ExceptionName, servername 
| stats count(ExceptionName) as totalCount by servername, ExceptionName
| eventstats sum(totalCount) as _total
| eventstats sum(totalCount) as _totalPerBank by servername
| eval percentageTotal=round((totalCount/_total)*100,2) 
| eval precentagePerServer=round((totalCount/_totalPerServer)*100,2) | sort -precentagePerServer
| stats list(precentagePerServer) as Percentage list(totalCount) as Counts list(ExceptionName) as ExceptionName by servername 
| sort - totalCount
