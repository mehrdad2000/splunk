# splunk

## cluster errors (trim first lines for exception)
index="myindex"  fatal* OR err* OR exception* OR timeout* OR waiting* OR fail* OR unable* OR lock* OR block*

| cluster showcount=t t=0.7 labelonly=t | table _time cluster_count cluster_label _raw  | dedup 1 cluster_label | sort - cluster_count cluster_label _time  | chart values(cluster_count) as count by _raw  | sort limit=20 - count 
|rex "(?<_raw>[^\n]*)\n"

## Java Exceptions
index="myindex" Caused* OR table* OR refuse* OR throw* OR ssl* OR "not found*" OR alert* OR db* OR sql* OR "not valid*" OR full* OR busy* OR down* OR terminate* OR timeout* OR "can't*" OR not* OR fault* OR fatal* OR informix* OR warn* OR fail* OR err* OR  Exception*   

| rex field=source  "\/data\/(?<product>\w+)\/(?<date>\d+)\/(?<servername>\w+)" 
  
| rex "Caused by:\s*(?P<Causedby>.*)"
| rex "(?<javaException>java[x]?\..*Exception)"
| rex "(?<javaException1>java[x]?\..*?Exception)"
| rex "(?<javaException2>java\..*Exception)"

| eval ExceptionName=coalesce(Causedby,javaException,javaException1,javaException2)

| fields ExceptionName, servername 
| stats count(ExceptionName) as totalCount by servername, ExceptionName
| eventstats sum(totalCount) as _total
| eventstats sum(totalCount) as _totalPerServer by servername
| eval percentageTotal=round((totalCount/_total)*100,2) 
| eval precentagePerServer=round((totalCount/_totalPerServer)*100,2) | sort -precentagePerServer
| stats list(precentagePerServer) as Percentage list(totalCount) as Counts list(ExceptionName) as ExceptionName by servername 
| sort - totalCount

 ## Jboss Exceptions
 
index="myindex" AMQ OR ARJUNA OR COM OR EJBCLIENT OR ELY OR HCANN OR HHH OR HSEARCH OR HV OR IJ OR ISNPHIB OR ISPN OR JBERET OR JBREM OR JBTHR OR JBWEB OR JBWS OR JIPI OR JNDIWFHTTP OR MODCLUSTER OR MSC OR PBOX OR PROBE OR RESTEASY OR TXNWFHTTP OR UT OR UTJS OR VFS OR WELD OR WFCMTOOL OR WFHTTP OR WFHTTPEJB OR WFLY* OR WFMIGRCLI OR WFNAM OR WFSM OR WFTXN OR XNIO OR jlibaio  | rex field=source "\/data\/(?<product>\w+)\/(?<date>\d+)\/(?<servername>\w+)" 
|rex "(?<ErrorCode1>WFLY[^:]+):" 
| rex "(?<ErrorCode2>(AMQ|ARJUNA|COM|EJBCLIENT|ELY|HCANN|HHH|HSEARCH|HV|IJ|ISNPHIB|ISPN|JBERET|JBREM|JBTHR|JBWEB|JBWS|JIPI|JNDIWFHTTP|MODCLUSTER|MSC|PBOX|PROBE|RESTEASY|TXNWFHTTP|UT|UTJS|VFS|WELD|WFCMTOOL|WFHTTP|WFHTTPEJB|WFLY*|WFMIGRCLI|WFNAM|WFSM|WFTXN|XNIO|jlibaio)\d+)" 
| eval ErrorCode=coalesce(ErrorCode1,ErrorCode2)

| fields ErrorCode, servername 
| stats count(ErrorCode) as totalCount by servername, ErrorCode
| eventstats sum(totalCount) as _total
| eventstats sum(totalCount) as _totalPerServer by servername
| eval percentageTotal=round((totalCount/_total)*100,2) 
| eval precentagePerServer=round((totalCount/_totalPerServer)*100,2) | sort -precentagePerServer
| stats list(precentagePerServer) as Percentage list(totalCount) as Counts list(ErrorCode) as ErrorCode by servername 
| sort - totalCount | lookup jboss-errors.csv ErrorCode  OUTPUT  description
