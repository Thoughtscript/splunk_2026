# Splunk_2026

![](https://img.shields.io/badge/splunk-2026-orange)

Review of some basics.

## Setup

```bash
$ docker run --platform linux/amd64 -d -p 8000:8000 -e "SPLUNK_START_ARGS=--accept-license" -e "SPLUNK_GENERAL_TERMS=--accept-sgt-current-at-splunk-com" -e "SPLUNK_PASSWORD=mypassword" --name splunk splunk/splunk:latest
```

1. Splunk doesn't release Apple ARM native distros. Use the `--platform linux/amd64` for Rosetta compatibility.
1. The above command will spin up an instance without having to pass the `-it --name so1 splunk/splunk:latest` flag.
1. [localhost:8000](http://localhost:8000/en-US/app/search/search) with login `admin` and `mypassword`.

    * It can take a moment for the Splunk instance to get up and running:

      ![](./splunk-docker-start.png)

## Queries

### Verifying Count Values()

> I guess I did something like this post: https://community.splunk.com/t5/Splunk-Search/Stats-Values-and-Count/m-p/305743

Comparing `distinct_count()` vs. `stats count values()`:

```splunk
| makeresults count=1000 
| eval status_code = case((random()%3)==0, "200", (random()%3)==1, "404", 1=1, "500")
| eval status_msg = case(status_code=="200", "OK", status_code=="404", "Not Found", 1=1, "Error")
| eval uuid = md5(random() . now())
| stats distinct_count(status_code), values(status_code)
```

![](./distinct_count.png)

```splunk
| makeresults count=1000 
| eval status_code = case((random()%3)==0, "200", (random()%3)==1, "404", 1=1, "500")
| eval status_msg = case(status_code=="200", "OK", status_code=="404", "Not Found", 1=1, "Error")
| eval uuid = md5(random() . now())
| stats count values(status_code) as distinct_codes
```

![](./values.png)

> So, I should keep using `distinct_count()` and not `count values` - syntatically, `stats count, values(status_code) as distinct_codes` and `stats count values(status_code) as distinct_codes`.

And indeed they are!

```splunk
| makeresults count=1000 
| eval status_code = case((random()%3)==0, "200", (random()%3)==1, "404", 1=1, "500")
| eval status_msg = case(status_code=="200", "OK", status_code=="404", "Not Found", 1=1, "Error")
| eval uuid = md5(random() . now())
| stats count, values(status_code) as distinct_codes
```

![](./count-comma-values.png)

Also:
```splunk
| makeresults count=1000 
| eval status_code = case((random()%3)==0, "200", (random()%3)==1, "404", 1=1, "500")
| eval status_msg = case(status_code=="200", "OK", status_code=="404", "Not Found", 1=1, "Error")
| eval uuid = md5(random() . now())
| stats count by status_code
```
![](./count-by.png)

This also returns the correct count of `3` with `stats count as ... by`:

```splunk
| makeresults count=1000 
| eval status_code = case((random()%3)==0, "200", (random()%3)==1, "404", 1=1, "500")
| eval status_msg = case(status_code=="200", "OK", status_code=="404", "Not Found", 1=1, "Error")
| eval uuid = md5(random() . now())
| stats count as code_counts values(status_code) by status_code
```

![](./stats-count-as-by.png)

As does this one:

```splunk
| makeresults count=1000 
| eval status_code = case((random()%3)==0, "200", (random()%3)==1, "404", 1=1, "500")
| eval status_msg = case(status_code=="200", "OK", status_code=="404", "Not Found", 1=1, "Error")
| eval uuid = md5(random() . now())
| stats values(status_code) as unique_codes 
| stats count by unique_codes
```

![](./double-stats.png)

**Note:** the following will ***NOT*** give the desired count of `3`:

```splunk
| makeresults count=1000 
| eval status_code = case((random()%3)==0, "200", (random()%3)==1, "404", 1=1, "500")
| eval status_msg = case(status_code=="200", "OK", status_code=="404", "Not Found", 1=1, "Error")
| eval uuid = md5(random() . now())
| stats values(status_code) as unique_codes
```

![](./just-values.png)

### Non-Uniqueness

> Hey, the complement of the above.

```splunk
| makeresults count=1000 
| eval status_code = case((random()%3)==0, "200", (random()%3)==1, "404", 1=1, "500")
| eval status_msg = case(status_code=="200", "OK", status_code=="404", "Not Found", 1=1, "Error")
| eval uuid = md5(random() . now())
| stats count by status_code > 1
```