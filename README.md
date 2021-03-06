# splunk-notes
Splunk notes.

## Parse Tomcat Access Logs

```
"some search" sourcetype="LogFiles.access_combined_wcookie" | rex field=_raw "] \"(?<httpMethod>[^\s]+) (?<urlPath>[^\s]+) (?<httpVersion>[^\s]+)\" (?<httpStatus>[^\s]+) (?<respBytes>[^\s]*) (?<respTimeMillis>[^\s]*) \"(?<userAgent>.*)\" \"(?<referrerUrl>.*)\""
```

## Filters

### Regex

```
"some search" | regex someFieldName = ".*SOME.*REGEX.*"
```

### Regex (negative)

```
"some search" | regex someFieldName != ".*SOME.*REGEX.*"
```

### Where

```
"some search" | where someFieldName = "someValue"
```

### Where(negative)

```
"some search" | where someFieldName != "someValue"
```

## Find User Agents by Referrer & URL without query

```
"some search" sourcetype="LogFiles.access_combined_wcookie" | rex field=_raw "] \"(?<httpMethod>[^\s]+) (?<urlPath>[^\s]+) (?<httpVersion>[^\s]+)\" (?<httpStatus>[^\s]+) (?<respBytes>[^\s]*) (?<respTimeMillis>[^\s]*) \"(?<userAgent>.*)\" \"(?<referrerUrl>.*)\"" | rex field=urlPath "(?<nonQuery>[^\?]*).*" | eval urls=referrerUrl + " -> " + nonQuery | chart count by userAgent, urls
```

## Multi-line
"transactions"

```
"" | transaction startswith="Request:" endswith=("Unable to add to cart") maxevents=4
```

## Correlation
Different event occurrences compared on graph.

```

host=myhost* sourcetype="LogFiles.tomcat-wrapper" "Launching a JVM" | eval event="jvm" | stats count by event, _time | eval occurred=if (count > 0, 1, 0) | append [search host=myhost* sourcetype="LogFiles.tomcat-wrapper" "permgen" | eval event="permgen" | stats  count by event, _time  | eval occurred=if (count > 0, 1, 0) ] | search event="jvm"

```

## Errors by message and host

```

host=myhost* sourcetype="LogFiles.tomcat-wrapper" | rex field=_raw "\s*(?<level>[^\s]+).*?\|.*?\|.*?\|(?<msg>.{0,80})" | regex msg!="\s+at" | regex msg!=".*\d+ more" | eval hostMsg=host+msg | chart count by hostMsg | sort by count desc

```

## Filter out timestamps and low-level JVM (GC, classloader) from messages

```

host=myhost* source="E:\\LogFiles\\user-services\\tomcat-wrapper.log" | rex field=_raw "\s*(?<level>[^\s]+).*?\|.*?\|.*?\|(\s*.*?\s\d+.*?(AM|PM))?(?<msg>.{0,80})" | regex msg!="(\[GC)|(Full GC\[)|(PSYoungGen)|(Unloading class)" | timechart limit=0 count by msg span=30min | sort by count desc

```

## Response times for URLS

```

"/my/url/* " host="myhost*" sourcetype="LogFiles.access_combined_wcookie" | rex field=_raw "\" [^\s]+ (?<respBytes_>[^\s]+) (?<respTimeMillis_>[^\s]+) " | timechart eval(avg(respTimeMillis_) + stdevp(respTimeMillis_)) AS AvgPlusSDRespTime by host

```
