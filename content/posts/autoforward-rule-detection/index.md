---
title: "Detecting Outlook Autoforward Rules With Email Headers"
date: 2021-05-05
# weight: 1
# aliases: ["/first"]
tags: ["outlook", "exchange", "proofpoint", "mimecast", "T1114"]
author: "Kevin Pham"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Get one step ahead of the attackers by detecting a common persistence tactic"
canonicalURL: ""
disableHLJS: false # to disable highlightjs
disableShare: true
disableHLJS: true
enableMermaid: true
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: false
ShowPostNavLinks: false
cover:
    image: "/posts/autoforward-rule-detection/email-exif.jpg" # image path/url
    alt: "Email exifration" # alt text
    caption: "" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: false # only hide on current single page
editPost:
    URL: "https://github.com/deoxykev/deoxykev.github.io"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---


# Ghost in the Mailbox 

{{<mermaid align="left">}}
graph LR;
    A[Hard edge] -->|Link text| B(Round edge)
    B --> C{Decision}
    C -->|One| D[Result one]
    C -->|Two| E[Result two]
{{< /mermaid >}}


hello :smile: world
## Being on the Outlook for Autoforwarding

### Mimecast Configuration

### Proofpoint Configuration
