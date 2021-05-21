---
title: "Automating Akamai / LetsEncrypt Cert Validation"
date: 2021-05-05
# weight: 1
# aliases: ["/first"]
tags: ["akamai", "letencrypt", "acme", "dns", "https", "bash"]
author: "Kevin Pham"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Implementing a missing feature with the Akamai CLI tools"
canonicalURL: ""
disableHLJS: true # to disable highlightjs
disableShare: true
disableHLJS: true
enableMermaid: true
enableECharts: false
enableAsciiCinema: true
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: false
ShowPostNavLinks: false
cover:
    image: "/posts/automating-akamai-cert-validation/automation-concept.jpg" # image path/url
    alt: "Email exifration" # alt text
    caption: "" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: false # only hide on current single page
editPost:
    URL: "https://github.com/deoxykev/deoxykev.github.io"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---



# CPS workflow
{{< mermaid >}}
graph TD;
  start[START] --> A
  subgraph Updates To Certs Initiated
    A[Create certificate with all subdomains in Akamai CPS GUI] -->
    B[Create Certificate Signing Request - CSR] -->
    C[Submit CSR to LetsEncrypt]
  end

  subgraph Manual Validation -- very time consuming!
    C -->
    D[List of DNS ACME Validation records returned] -->
    E[Add each record by hand in the Akamai DNS GUI] -->
    F[Wait an hour until LetsEncrypt validates ownership of domains] 
  end

  subgraph Akamai Cert Provisioning System 
    F -->
    G[Retrieve certificates] -->
    H[Push to Akamai staging] -->
    I{Is always test<br>on staging on?}
    I -- yes --> test[Wait until manual validation]
    I -- no --> deploy[Deploy to production]
    test --> wait[Wait 60 days until certificate expiry]
    deploy --> wait[Wait 60 days until certificate expiry]
  end
    wait --> return[Do it all over again]

    class D orange;
    class E orange;
    class F orange;
        classDef orange fill:#f1b185, color: black;
{{< /mermaid >}}


# Demo 
{{< asciicinema file="validator.cast" autoplay="true" loop="true" title="ACME Validator Demo" >}}

## [Code](https://github.com/deoxykev/akamai-ACME-validator)

```bash
#!/bin/bash
# automates ACME validation for Akamai
# 1. fetch all pending CPS ACME validation DNS records
# 2. fetch all zone files 
# 3. update zone files with new ACME validation TXT records 
# 4. upload the new zone files

set -e

# fetching all CNs with pending cert changes"
CNs=$(akamai cps list | grep 'dv san' | grep '*Yes*' | cut -f3 -d'|' | awk '{print $1}')


# fetch all ACME validation records for each Domain in Akamai
rawRecords=""
for CN in ${CNs}; do
  rawRecords=$(echo -e "$rawRecords\\n$(akamai cps status --cn "$CN" --validation-type dns 2>&1 | grep Awaiting)")
done


# fetch all zones"
zones=$(akamai dns list-zoneconfig --summary | grep ACTIVE | awk '{print $1}')

# clean up files from our previous run 
[[ -e "./zonefiles" ]] && rm -rf "./zonefiles"
mkdir zonefiles

for zone in ${zones}; do

  # skip over zones which don't have any pending changes
  [[ $(echo "$rawRecords" | grep $zone) ]] || continue

  # fetch zone file for each zone
    akamai dns retrieve-zoneconfig $zone -dns --output "./zonefiles/${zone}.zone.tmp2"

  # increment SOA serial for each zone file 
  awk 'BEGIN{ OFS="\t" } /SOA/{$7=$7+1} 1' "./zonefiles/${zone}.zone.tmp2" > "./zonefiles/${zone}.zone.tmp"

  # delete old acme records
  grep -v "_acme-challenge." "./zonefiles/${zone}.zone.tmp" > "./zonefiles/${zone}.zone"

  # append new acme records for $zone ${NC}"
  echo "$rawRecords"  \
         | grep $zone \
         | awk  '{print "_acme-challenge." $2 ".\t" "60\t" "IN\t" "TXT\t" $7}' \
         >> "./zonefiles/${zone}.zone"


  # upload our edited zonefile
  akamai dns update-zoneconfig $zone -dns -file ./zonefiles/${zone}.zone

done

exit 0
```
