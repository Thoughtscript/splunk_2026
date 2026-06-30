# Splunk_2026

![](https://img.shields.io/badge/splunk-2026-orange) [![](https://img.shields.io/badge/My-Credly-darkgreen)](https://www.credly.com/users/adam-gerard/badges/credly)

Review of some basics.

## Setup

```bash
$ docker run --platform linux/amd64 -d -p 8000:8000 -e "SPLUNK_START_ARGS=--accept-license" -e "SPLUNK_GENERAL_TERMS=--accept-sgt-current-at-splunk-com" -e "SPLUNK_PASSWORD=mypassword" --name splunk splunk/splunk:latest
```

1. Splunk doesn't release Apple ARM native distros. Use the `--platform linux/amd64` for Rosetta compatibility.
1. The above command will spin up an instance without having to pass the `-it --name so1 splunk/splunk:latest` flag.
1. [localhost:8000](http://localhost:8000/en-US/app/search/search) with login `admin` and `mypassword`.

    * It can take a moment for the Splunk instance to get up and running:

      ![](./_screenshots/splunk-docker-start.png)

1. Also, within Docker Desktop, you'll likely need to adjust your **Settings** > **Resources** > **Disk usage limit** setting (and clear up some extra disk space) per usual.

## Queries

Splunk Core Certified Power User:
1. [Topic One: Introduction to Splunk](CCPU-Introduction.md)
1. [Topic Two: Fields](CCPU-Fields.md)
1. [Topic Three: Time](CCPU-Time.md)
1. [Topic Four: Statistical Processing](CCPU-Statistical-Processing.md)
1. [Topic Five: Comparing Values]()
1. [Topic Six: Result Modification]()
1. [Topic Seven: Correlation Analysis]()
1. [Topic Eight: Introduction to Knowledge Objects]()
1. [Topic Nine: Creating Knowledge Objects]()
1. [Topic Ten: Field Extractions]()
1. [Topic Eleven: Data Models]()

Misc. Command and Query review:
1. [Count, Values, Distinct Count](Queries-Count-Values.md)
1. [Split, mvexpand](Queries-Split-mvexpand.md)
1. [Dynamically generate IN List](Queries-Generate-IN-List.md)