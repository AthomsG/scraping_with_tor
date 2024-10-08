![Build Status](https://github.com/AthomsG/scraping_with_tor/actions/workflows/docker-hub.yml/badge.svg)

# Trustpilot Scraper Tool

This repo contains a scraper for Trustpilot (see [here](https://github.com/AthomsG/scraping_with_tor/tree/main/scrape_reviews_website) for more information). For successfuly installing this tool you either use Docker or manually install TOR following the information here. 

## Scraper with Tor Using Docker

This guide will help you set up and run the scraper with Tor inside a Docker container. This ensures a consistent environment without needing to manually install Tor or manage dependencies on your machine.

### 1. Build the Docker Image

First, you need to build the Docker image. For that you need to have [Docker](https://docs.docker.com/get-started/get-docker/) installed. This will install all necessary dependencies, including Tor and the scraper.

In the terminal, navigate to the project directory containing the `Dockerfile` and run:

```bash
docker build --no-cache -t tor-scraper .
```

This command will create a Docker image called `tor-scraper`.

or pull from the [Docker Hub](https://hub.docker.com/r/gengiskhan/trustpilot-scraper).

### 2. Run the Docker Container

To run the Docker container and start an interactive session, use the following command:

```bash
docker run -it tor-scraper
```

This command will start the container and drop you into a shell where you can use the scraper.

### 3. Running the Scraper Inside Docker

Once inside the Docker container, you can run the scraper with the `scrape` command. The `scrape` command allows you to specify the website to scrape, the number of pages, and whether or not to use Tor. Basically, the `scrape` command is just an alias for the python script (see [here](https://github.com/AthomsG/scraping_with_tor/tree/main/scrape_reviews_website) for documentation).

#### Example

```bash
scrape --website idealista.com --pages 1 --tor --get_countries
```

The `scrape` command runs the Python script inside the container and outputs the scraped data as a `.csv` file.

### 4. Retrieving Scraped Data

The scraped data is stored inside the Docker container at `/app/output`. To retrieve the `.csv` files from the container and copy them to your local machine, follow these steps:

1. **Find the Container ID:** Run the following command on your host machine to get the container ID:
```bash
docker ps -a
```
This will list all running and stopped containers. Note the `CONTAINER ID` of the container where the scraper ran.

2. **Copy the Data:** Use the `docker cp` command to copy the `output` directory from the container to host machine:
```bash
docker cp <container_id>:/app/output .
```
This will copy the contents of `/app/output` inside the container to the current directory on your host machine.

## Manual TOR Setup and Usage Guide

This guide explains how to set up and run TOR to use it for applications such as scraping, generating new IPs, and anonymized browsing.

### 1. Install TOR

#### macOS (using Homebrew)

If you're using macOS, you can install TOR using Homebrew:

```bash
brew install tor
```

#### Linux

For Debian/Ubuntu:

```bash
sudo apt update
sudo apt install tor
```

For Fedora:

```bash
sudo dnf install tor
```

### 2. Starting and Stopping TOR

#### Start TOR Manually

After setting up the `torrc` file, you can start TOR manually with:

```bash
tor -f /path/to/your/torrc
```

For example, if you are using a sample `torrc` file, you can run:

```bash
tor -f ../../usr/local/etc/tor/torrc.sample
```

This will start TOR with the custom configuration file. If successful, you should see output indicating that TOR is bootstrapping and establishing circuits.

#### Stop TOR

You can stop TOR with `Ctrl + C` if running manually, or if it is running in the background, use the `kill` command:

```bash
sudo killall tor
```

#### Check If TOR Is Running

To check if TOR is running and if the control port is accessible, run:

```bash
nc -v 127.0.0.1 9051
```

If it connects, you'll see:

```
Connection to 127.0.0.1 port 9051 [tcp/*] succeeded!
```

Otherwise, it will say 'Connection refused.'

### 3. TOR Configuration (Creating the `torrc` File)

#### Control Port Setup

To enable the TOR control port, create or modify your `torrc` file. This file is usually located at `/usr/local/etc/tor/torrc` or `/etc/tor/torrc`.

Here’s an example configuration:

```bash
ControlPort 9051
CookieAuthentication 1
```

#### Setting Up a Hashed Control Password

If you want to use password-based authentication for the control port, you’ll need to add a hashed password in the `torrc` file.

Generate the hashed password with the following command:

```bash
tor --hash-password your_password # (we used 'abc123' for the example)
```

This will output a hashed password. Add it to your `torrc` like this:

```bash
ControlPort 9051
HashedControlPassword your_hashed_password
```

Then, restart TOR.

### 4. Verifying the TOR Connection

To verify that TOR is working, you can check your IP by using a Python script like this:

```python
import socks
import socket
from urllib.request import urlopen

# Set up the proxy
socks.setdefaultproxy(socks.PROXY_TYPE_SOCKS5, '127.0.0.1', 9050, True)
socket.socket = socks.socksocket

# Check the IP
print(urlopen('https://icanhazip.com').read().decode().strip())
```

This script will print the IP address that TOR is using.

### 5. Troubleshooting

- If you get 'Connection refused' when trying to connect to the control port, make sure the `torrc` file is correctly configured and TOR is started with the correct configuration.
- Make sure no other process is using port `9051` by running:

```bash
sudo lsof -i :9051
```

- Check TOR logs for errors by running:

```bash
tail -f /usr/local/var/log/tor.log
```

---
