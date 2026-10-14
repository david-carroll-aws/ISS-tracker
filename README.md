# 🛰️ Live ISS Tracker

About every 90 minutes, the International Space Station completes another lap of the Earth, moving at roughly 27,500 km/h some 430 km above us. I wanted to watch that happen in real time, so I set out to build a live ISS tracker and host it on AWS.

**Live demo:** davidcarroll.cloud

<img width="1031" height="579" alt="image" src="https://github.com/user-attachments/assets/193805e5-3b9d-4348-b6b6-0a4140474aa9" />

## Features

- **Live position:** see where the station is right now
- **Speed and altitude:** current velocity and height above Earth
- **Recent path:** every country and ocean it has crossed in the last hour
- **Crew:** the astronauts currently on board

## Tech Stack

- **Frontend:** HTML, CSS, JavaScript
- **Backend:** Python
- **Hosting:** AWS (EC2)
- **Deployment:** automated pipeline from dev environment to GitHub to live server
- **Data sources:** [list the APIs you use for position, location names, and crew]

## How It Works

1. [Python fetches the station's position from ___ on a schedule / on request]
2. [Coordinates are reverse-geocoded to a country or ocean]
3. [The frontend polls ___ and updates the map and stats]

## Deployment

Changes pushed to GitHub deploy automatically to the live server, so there are no manual release steps.

## Running Locally

```bash
git clone https://github.com/[your-username]/ISS.git
cd ISS
[install and run commands]
```

## Roadmap

- [ ] Automated tests and linting in the pipeline
- [ ] Staging environment

## License

[MIT / your choice]
