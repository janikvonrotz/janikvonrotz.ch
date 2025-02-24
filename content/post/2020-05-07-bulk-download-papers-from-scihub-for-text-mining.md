---
title: "Bulk download papers from scihub for text mining"
slug: bulk-download-papers-from-scihub-for-text-mining
date: 2020-05-07T18:09:55+02:00
categories:
 - Scripting
tags:
 - scihub
 - education
 - papers
images:
 - "/images/bright theme code.jpg"
---

Last evening I wanted to play some games with my brother. Instead we ended up writing a simple script to bulk download papers from scihub. He studies bioinformatics and is currently doing some meta research in his field. By crawling a publication database by specific keywords he got list of papers which need to be analyzed. However, most of the paper are hidden behind pay walls. Luckily there's scihub. The most hated and beloved platform to get your hands on almost any scientific paper. He asked me wether I could help him building script that downloads papers from scihub based on a list of dois.
<!--more-->

Sure I said, opended the browser and had a look at the requests that need to be performed to download a single paper.

When submitting the doi the following request is sent.

![](/images/scihub/request1.png)

The http code 302 means that there is a redirection happening. If we want automate this process following redirects must be enabled.

![](/images/scihub/request2.png)

The redirected response contains the paper. The url to the pdf file can be extracted using regex.

The initial request can be copied as curl command. This makes writing a script and bypassing any agent-checks much easier.

![](/images/scihub/curl.png)

We both started Visual Studio Code and connected via the live coding plugin. My brother sent me a excerpt of the doi list:

**doi-list.txt**

```txt
10.3390/ijms21062239
10.1007/s12094-019-02276-8
10.31782/IJCRR.2019.11242
10.1016/j.chom.2019.05.007
10.3390/v11070638
10.1128/IAI.00733-18
10.1186/s12861-019-0183-y
10.1155/2019/8472712
10.1002/stem.2852
10.3390/genes9040176
10.1016/j.neulet.2018.01.040
10.1093/molehr/gax070
10.15252/emmm.201708213
10.1007/s40139-017-0137-7
10.1111/cas.13155
10.4103/0366-6999.191782
10.1126/science.aaf5211
```

And together we built this bash simple script:

**scihub-download.sh**

```bash
urlencode() {
    # urlencode <string>
    old_lc_collate=$LC_COLLATE
    LC_COLLATE=C

    local length="${#1}"
    for (( i = 0; i < length; i++ )); do
        local c="${1:i:1}"
        case $c in
            [a-zA-Z0-9.~_-]) printf "$c" ;;
            *) printf '%%%02X' "'$c" ;;
        esac
    done

    LC_COLLATE=$old_lc_collate
}

readarray -t list <doi-list.txt

for doi in "${list[@]}"
do
    echo "Download for doi: $(urlencode $doi)"
    link=$(curl -s -L 'https://sci-hub.tw/' --compressed \
        -H 'User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:75.0) Gecko/20100101 Firefox/75.0' \
        -H 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8' \
        -H 'Accept-Language: en-US,en;q=0.5' \
        -H 'Content-Type: application/x-www-form-urlencoded' \
        -H 'Origin: https://sci-hub.tw' \
        -H 'DNT: 1' \
        -H 'Connection: keep-alive' \
        -H 'Referer: https://sci-hub.tw/' \
        -H 'Cookie: __ddg1=SFEVzNPdQpdmIWBwzsBq; session=45c4aaad919298b2eb754b6dd84ceb2d; refresh=1588795770.5886; __ddg2=6iYsE2844PoxLmj7' \
        -H 'Upgrade-Insecure-Requests: 1' \
        -H 'Pragma: no-cache' \
        -H 'Cache-Control: no-cache' \
        -H 'TE: Trailers' \
        --data "sci-hub-plugin-check=&request=$(urlencode $doi)". | grep -oP  "(?<=//).+(?=#)")

    echo "Found link: $link"
    curl -s -L $link --output $(urlencode $doi).pdf
done
```

It reads the text file and sends a request to scihub for each entry. If the response contains a valid pdf link, the file is downloaded. The doi request parameter must be url encoded.

In the next step he will process the content of the papers and analyze them for specific keywords and applied methods.
