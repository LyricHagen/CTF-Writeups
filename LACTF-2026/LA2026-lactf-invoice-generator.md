# LACTF Invoice Generator (February 7, 2026)

**Category:** web  
**Points:** 101  
**Files Provided:** dist.tar.gz, containing the following:
```
dist
├── docker-compose.yml
├── flag
│   ├── Dockerfile
│   └── flag.js
└── invoice-generator
    ├── Dockerfile
    ├── package-lock.json
    ├── package.json
    ├── public
    │   ├── index.html
    │   └── style.css
    └── server.js
```
As well as a site: `lactf-invoice-generator-[instance ID].instancer.lac.tf`

## Challenge Description

"I need an itemized list of everything purchased for LA CTF 2026."

## Step 1: Exploring

The site lets you enter a Customer Name, Item Description, Cost ($), and Date Purchased. You can then click "Generate Invoice PDF" to get a pdf of an invoice based on what you entered. I looked at the provided server.js - there's no sanitization on the `${name}` (see below,) or any other field, so the vulnerability is HTML injection. I figured the best tags to try are <iframe> or <script> to perform SSRF.
```
<div class="customer-info">
    <strong>Bill To:</strong>
    ${name}
</div>
```

## Step 2: The Exploit

I looked at more of the provided files. In docker-compose.yaml, you can see two services, invoice-generator and flag; flag.js shows that the flag is being served by a small web server on port 8081. Containers can talk to each other using their service names as hostnames, so I decided the goal was to get server.js to "visit" `http://flag:8081/flag` and include the result in my generated pdf.

I entered the following into the Customer Name field and put filler values for the other fields.

`<iframe src="http://flag:8081/flag" width="100%" height="200px"></iframe>`

I opened the downloaded pdf, and the flag was right there.

`lactf{plz_s4n1t1z3_y0ur_purch4s3_l1st}`

## Takeaways
- When a server renders user-supplied HTML into a pdf, that's an XSS opportunity where the "victim" is the server's internal "browser"
- Read the Docker compose file to find service names to reach other containers
