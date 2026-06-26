# Dynamically Generate IN List

```splunk
| makeresults count=50 
| eval status_code = case((random()%3)==0, "200", (random()%3)==1, "404", 1=1, "500")
| eval status_msg = case(status_code=="200", "OK", status_code=="404", "Not Found", 1=1, "Error")
| eval alphabet="ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789" 
| eval uuid=substr(alphabet, (random() % 36) + 1, 5)
| table _time, uuid, status_code, status_msg

| stats values(uuid) AS uuid_values 
| mvexpand uuid_values 

| where uuid_values IN ([ makeresults count=9999 
| eval alphabet="ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789" 
| eval matching_uuid=substr(alphabet, (random() % 36) + 1, 5)
| stats values(matching_uuid) AS matching_uuids 
| eval matching_uuids_list_string = mvjoin(matching_uuids, "\", \"") 
| return matching_uuids_list_string ])
| table uuid_values
```

First half generates `uuid_values` and converts them into individual rows.

```splunk
| makeresults count=50 
| eval status_code = case((random()%3)==0, "200", (random()%3)==1, "404", 1=1, "500")
| eval status_msg = case(status_code=="200", "OK", status_code=="404", "Not Found", 1=1, "Error")
| eval alphabet="ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789" 
| eval uuid=substr(alphabet, (random() % 36) + 1, 5)
| table _time, uuid, status_code, status_msg

| stats values(uuid) AS uuid_values 
| mvexpand uuid_values 
```

Second half dynamically generates a String List (with prefixed `|` required to run the sub-Query by itself):
```splunk
| makeresults count=9999 
| eval alphabet="ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789" 
| eval matching_uuid=substr(alphabet, (random() % 36) + 1, 5)
| stats values(matching_uuid) AS matching_uuids 
| eval matching_uuids_list_string = mvjoin(matching_uuids, "\", \"") 
| return matching_uuids_list_string
```
```plaintext
matching_uuids_list_string="01234", "12345", "23456", "34567", "45678", "56789", "6789", "789", "89", "9", "ABCDE", "BCDEF", "CDEFG", "DEFGH", "EFGHI", "FGHIJ", "GHIJK", "HIJKL", "IJKLM", "JKLMN", "KLMNO", "LMNOP", "MNOPQ", "NOPQR", "OPQRS", "PQRST", "QRSTU", "RSTUV", "STUVW", "TUVWX", "UVWXY", "VWXYZ", "WXYZ0", "XYZ01", "YZ012", "Z0123"
```

then checks if the generated `uuid_values` are `IN` that List. `IN` accepts a comma- and quote- delimitted String List so it must be formatted this way (it won't accept `| stats values(matching_uuid)` or `| stats values(matching_uuid) | return matching_uuids_list_string`):
```splunk
| where uuid_values IN ("01234", "12345", "23456", "34567", "45678", "56789", "6789", "789", "89", "9", "ABCDE", "BCDEF", "CDEFG", "DEFGH", "EFGHI", "FGHIJ", "GHIJK", "HIJKL", "IJKLM", "JKLMN", "KLMNO", "LMNOP", "MNOPQ", "NOPQR", "OPQRS", "PQRST", "QRSTU", "RSTUV", "STUVW", "TUVWX", "UVWXY", "VWXYZ", "WXYZ0", "XYZ01", "YZ012", "Z0123")
```

![](./_screenshots/dynamic-list.png)

> Be forewarned that `| return matching_uuids_list_string` appears to be required/must be present within the `IN` clause or the Query will fail. This part took a bit to figure out.

> Another gotcha: values generated in `matching_uuids_list_string` should have quotes (`"`) but `uuid_values` should not.