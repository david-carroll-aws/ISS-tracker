# 🛰️ Live ISS Tracker on AWS

About every 90 minutes, the International Space Station completes another lap of the Earth, moving at roughly 27,500 km/h some 430 km above us. I wanted to watch that happen in real time, so I set out to build a live ISS tracker and host it on AWS.

The page shows where the station is right now, how fast and how high it's travelling, every country and ocean it has crossed in the last hour, and the astronauts on board.

**Live tracker:** [davidcarroll.cloud/iss.html](https://davidcarroll.cloud/iss.html)
**Dashboard:** [davidcarroll.cloud](https://davidcarroll.cloud)
**Full write-up:** [What I Learned Building a Live ISS Tracker on AWS](YOUR_LINKEDIN_ARTICLE_URL)

![ISS tracker screenshot](docs/screenshot.png)

This README covers how it works, the AWS services behind it, and the issues I encountered along the way.

## Architecture

![Architecture diagram](docs/architecture.png)

Here's the whole site at a glance. Apache is the single front door: it serves the pages and proxies the Flask app and the crew list. The browser calls three public APIs directly, and reaches the visit log through API Gateway.

## What the tracker shows

The tracker turns raw orbital data into something you can read at a glance. Four things on the page do most of the work:

- **A live headline.** The position refreshes every 3 seconds, and the page names whatever country or ocean the station is over.
- **Live stats.** Altitude, speed, latitude and longitude, and whether the station is in daylight or Earth's shadow.
- **A ground track.** A running list of every country, ocean and locality the ISS has passed over, with how long it spent in each. When the page loads it rebuilds the last 60 minutes at one-minute steps, then keeps adding live lookups roughly every 10 seconds.
- **The crew aboard.** A column of the people on the station right now, with photos and a one-line description pulled in automatically.

![Raw data panel](docs/raw-data-panel.png)

Below the stats, a raw-data panel shows exactly what the position and geocoding APIs returned on the latest refresh, so anyone curious can see what's behind the headline.

## How it's built on AWS

The whole site runs on one Amazon EC2 instance, with a few other AWS services around it. The tracker page itself has no server code: the instance serves it as static HTML and JavaScript, and each visitor's browser calls the data APIs directly.

- **Amazon EC2 (Amazon Linux 2023).** One instance runs the Apache web server and serves the dashboard at davidcarroll.cloud, including the ISS, earthquake and F1 pages.
- **HTTPS everywhere.** I installed a free Let's Encrypt certificate with Certbot and `mod_ssl`, forced every HTTP request to redirect to HTTPS, and set a daily cron job that renews the certificate automatically before it expires.
- **Apache as a reverse proxy.** Browsers refuse to load plain-HTTP data from a secure page, and one of my data sources, the crew list, is HTTP-only. Apache fetches it server-side and serves it from my own HTTPS domain. The same proxy puts my Flask lottery simulator behind the one certificate instead of exposing it on its own port.
- **Amazon S3.** A bucket holds the site's images, including the preview thumbnail that LinkedIn shows when the tracker is shared.
- **Amazon API Gateway and AWS Lambda.** A serverless backend powers the visit log on the dashboard. I locked down its CORS settings to my own HTTPS origin and protect it with an admin key header.
- **Amazon CloudWatch.** The Lambda's logs land in CloudWatch, which is the first place I look when an API call fails.

## Tech stack at a glance

Plain web technologies on the front end, AWS for hosting and the API layer, and four public APIs for the data.

- **Front end:** HTML, CSS and vanilla JavaScript, with no framework and no build step.
- **AWS:** EC2, S3, API Gateway, Lambda and CloudWatch.
- **Server:** Amazon Linux 2023, Apache httpd with `mod_ssl` and `mod_proxy`, Let's Encrypt and Certbot, cron.
- **Other backend:** Python and Flask, for the lottery simulator that sits behind the same proxy.
- **Position data:** [wheretheiss.at](https://wheretheiss.at) gives the station's latitude, longitude, altitude, speed and daylight status.
- **Place names:** [BigDataCloud](https://www.bigdatacloud.com) turns each position into a country, ocean or locality.
- **Crew list:** [Open Notify](http://open-notify.org) reports who is aboard the ISS.
- **Photos and descriptions:** the [Wikipedia REST API](https://en.wikipedia.org/api/rest_v1/) supplies each astronaut's photo and one-line summary.

## What went wrong, and what I learned

Most of what I learned came from things breaking. These were the five that taught me the most:

1. **API limits shape the design.** The position API allows about one request per second, so the page rebuilds its hour of history in six paced batches of 10 timestamps, retries once on failure, and skips a batch that fails twice. Each page load also makes 60 geocoding lookups. After the free, keyless geocoder blocked my IP, I switched to an API key.
2. **HTTPS changes your origin.** The day I enabled HTTPS, the visit log stopped loading. API Gateway's CORS settings still allowed only `http://davidcarroll.cloud`, and to a browser the HTTPS site is a different origin. Adding it fixed the problem.
3. **A secure page can't load insecure data.** The crew API only speaks HTTP, so browsers blocked it. Routing it through an Apache reverse proxy on my own domain solved that without any extra servers.
4. **Proxying an app means proxying its paths.** Once the lottery simulator moved behind `/lottery/`, its page loaded but its calls to `/api/simulate` returned 404. The app needed a proxy rule for `/api/` as well.
5. **Minimal images are minimal.** Amazon Linux 2023 ships without cron and without `mod_ssl`. Certbot's first attempt failed with "Could not find ssl_module", and my renewal job needed cron installed first.

I built this with Claude as a pairing partner, and debugging live on the server taught me more about Apache, TLS and CORS than any tutorial had.

## Deployment

Code changes push from my dev environment to GitHub and deploy automatically to the EC2 server.

## What's next

- [ ] **History.** Live lookups run about every 10 seconds, but the rebuilt hour is sampled once a minute, so refreshing the page loses some live detail. The planned fix is logging every lookup to Amazon DynamoDB and replaying from there.
- [ ] **Days in space.** Add launch dates to the crew cards for a "days in space" counter.
- [ ] Automated testing in the CI/CD pipeline.
- [ ] A second application on the same pipeline.

## Links

- Live tracker: [davidcarroll.cloud/iss.html](https://davidcarroll.cloud/iss.html)
- Dashboard: [davidcarroll.cloud](https://davidcarroll.cloud)

If you're building on AWS too, I'd love to hear what you'd do differently.
