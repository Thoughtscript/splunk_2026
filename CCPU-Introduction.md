# Core Certified Power User - Topic One: Introduction

Simple example that generates fake data through `makeresults` and then performs a simple `search` with boolean `OR`:

```splunk
| makeresults count=1000 
| eval status_code = case((random()%3)==0, "200", (random()%3)==1, "404", 1=1, "500")
| eval status_msg = case(status_code=="200", "OK", status_code=="404", "Not Found", 1=1, "Error")
| eval uuid = md5(random() . now()) 

| search status_msg="Error" OR status_msg="OK"
```

> Fake data is generated since this is a simple Docker example with now ETL Pipeline or Indexing Cluster. I've placed a line break between these two parts to seperate the "fake data" portion from the simple Query example.

Two interesting things here:
1. In a typical Splunk scenario, there's a real underlying Event (that gets Indexed). It'll typically have a `_raw` field. It's this `_raw` field that the Query: `search "Error" "OK"` queries against (equivalent to `search _raw="Error" AND _raw="OK"`).
1. So, `... | search "Error" "OK"` using the above Query will ***not*** return any results.