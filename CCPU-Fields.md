# Core Certified Power User - Topic Two: Fields

Simple example that generates fake data through `makeresults` and then demonstrates renaming a Field using `eval`, removing a Field using `fields -`, renaming a Field using `rename`, then searching without that Field:

```splunk
| makeresults count=1000 
| eval status_code = case((random()%3)==0, "200", (random()%3)==1, "404", 1=1, "500")
| eval status_msg = case(status_code=="200", "OK", status_code=="404", "Not Found", 1=1, "Error")
| eval uuid = md5(random() . now()) 

| eval status_msg_new = status_msg
| fields - status_msg 
| rename status_msg_new AS status_msg 
| search status_msg="OK" 
```

1. Fields renamed with `eval` create additional, new, Fields that will appear alongside the previous ones.
1. Fields renamed with `rename` are "inplace" and overwrite the existing Field (only the new name will populate).
1. Fields that are renamed must be referenced in that fashion thereafter. Appending: `... | search status_msg_new="OK" ` to the above Query will ***not*** return any results.

