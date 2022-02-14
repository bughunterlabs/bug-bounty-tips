# Bug Bounty Tips #2

Second video of the bug bounty tips series [[Youtube](https://www.youtube.com/watch?v=BLOXlUa_WKs)] with 5 bug bounty tips, which will improve your recon process and help you to find those juicy bugs.

## #1 Grep URLs

[Tweet](https://twitter.com/imranparray101/status/1274344698547539970) by [@imranparray101](https://twitter.com/imranparray101)

This simple grep command can be used to extract URLs from a text file or websites source code. So let’s try to extract some websites from DoD to hunt on. 

This website hosts many links belonging to DoD. Let’s curl the source code:

```bash
curl -o "dod_source.html" https://www.defense.gov/Resources/Military-Departments/DOD-Websites/
```

Now that we have the source code, let’s extract all URLs witht the grep command:

```bash
cat dod_source.html | grep -Eo "(http|https)://[a-zA-Z0-9./?=_-]*" | tee dod_sites.txt
```

We get a lot of results, but there are a few sites in there, which we do not want to hunt on since they are out-of-scope. So let’s do some post-processing and filter for .mil.

```bash
cat dod_sites.txt | tr '[:upper:]' '[:lower:]'| grep ".mil" | sort -u | tee dod_sites_filtered.txt
```

Finally, let’s extract only the basic domains  form the urls

```bash
cat dod_sites_filtered.txt | cut -d'/' -f3 | cut -d':' -f1 | rev | cut -d'.'  -f1,2 | rev | sort -u > dod_domains.txt
```

---

## #2 Bufferover Subdomains

[Tweet](https://twitter.com/bughunterlabs/status/1491601173153202177) by [@bughunterlabs](https://twitter.com/bughunterlabs)

Bufferover provides fast domain name lookups for TLS certificates in IPv4 space. On the site you can get an API-Token. Let’s start with an example on [starbucks.com](http://starbucks.com)

```bash
DOMAIN="starbucks.com"; curl "https://tls.bufferover.run/dns?q=.${DOMAIN}" -H "x-api-key: ${API_KEY}" | jq  .Results  | cut -d ',' -f4 | sed 's/\".*$//g' | grep "${DOMAIN}"
```

Let’s use tls.bufferover to find all subdomains of our previously found DoD domains.

```bash
while read DOMAIN; do 
		echo "Searching subdomains for ${DOMAIN}..."
    curl -s "https://tls.bufferover.run/dns?q=.${DOMAIN}" -H "x-api-key: ${API_KEY}" | jq  .Results  | cut -d ',' -f4 | sed 's/\".*$//g' | grep "${DOMAIN}" | sort -u | tee -a dod_subdomains.txt
done < "dod_domains.txt"
sort -o dod_subdomains.txt dod_subdomains.txt
```

---

## #3 Probe domains with [httpx](https://github.com/projectdiscovery/httpx)

To probe, which subdomains are alive, we can use httpx by [projectdiscovery](https://github.com/projectdiscovery). 

```bash
cat dod_subdomains.txt | httpx --silent | tee dod_subdomains_alive.txt
```

---

## #4 Favicon Hashes

[Tweet](https://twitter.com/bughunterlabs/status/1491243290553098243) by [@bugunterlabs](https://twitter.com/bughunterlabs)

Favicons are a file containing the small icon associated with a particular web page. Their location is commonly at `./favicon.ico` . By calculating the hash of the favicon files, we can search for the hash with shodan to identify new targets. Here is a one-liner to caluclate the hash of the yahoo.com favicon. 

```bash
curl -s -L -k http://yahoo.com/favicon.ico | python3 -c 'import mmh3,sys,codecs;print(mmh3.hash(codecs.encode(sys.stdin.buffer.read(),"base64")))'
```

Let’s look for one of the hashes with shodan

```bash
/search?query=http.favicon.hash:1213881203
```

Let’s apply this technique to our domain list. Luckily httpx has a feature which implements this already for us. 

```bash
cat dod_subdomains.txt | httpx --silent -favicon
```

Let’s check for one of the hashes

```bash
/search?query=http.favicon.hash:-1840324437
```

---

## #5 Google Dorks 
[Live Recon with Dawgyg](https://www.youtube.com/watch?v=GeNJvOvzVSk) by [@nahamsec](https://twitter.com/NahamSec)

Copy and Paste these dorks into google and see what you can find. 

```bash
site:.mil ext:php intitle:Setup
site:.mil ext:pdf intitle:Setup
site:.mil inurl:"img_url"
site:.mil inurl:"/admin/"
site:.mil inurl:"/api/"
```
