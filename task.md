# Ronaut Radio - Deployment Task List

This document outlines the step-by-step process for deploying the Ronaut Radio website to the Contabo VPS. The architecture is detailed in `Architecture.md`.

---

### Phase 1: Server Preparation & Backend Deployment
*This phase focuses on setting up the server environment and getting the Node.js chat application running.*

- [ ] **Task 1: Server Cleanup & Dependency Installation**
  - [ ] SSH into the Contabo VPS.
  - [ ] Remove any old project directories and old Nginx configurations to ensure a clean slate.
  - [ ] Update package lists: `sudo apt update && sudo apt upgrade -y`.
  - [ ] Install required software: `sudo apt install nginx git nodejs npm -y`.
  - [ ] Install `pm2` globally using npm: `sudo npm install pm2 -g`.

- [ ] **Task 2: Code Deployment**
  - [ ] Clone the GitHub repository into the correct production directory: `git clone https://github.com/aeternitas14/ronaut-radio-website.git /var/www/ronautradio.la`.
  - [ ] Navigate into the chat server directory: `cd /var/www/ronautradio.la/chat-server`.
  - [ ] Install Node.js dependencies: `npm install`.

- [ ] **Task 3: Run Chat Server with PM2**
  - [ ] Start the chat server using `pm2`: `pm2 start server.js --name ronaut-chat`.
  - [ ] Verify the application is running: `pm2 list`.
  - [ ] Ensure `pm2` will restart on server reboot: `pm2 startup` and `pm2 save`.

---

### Phase 2: Nginx Configuration & DNS/Firewall Checks
*This phase focuses on getting the site live over HTTP and preparing for HTTPS.*

- [ ] **Task 4: Basic Nginx Configuration (HTTP)**
  - [ ] Create a new Nginx configuration file: `sudo nano /etc/nginx/sites-available/ronautradio.la`.
  - [ ] Add the basic HTTP server block that reverse proxies to the Node.js app on `localhost:3000`.
  - [ ] Enable the site: `sudo ln -s /etc/nginx/sites-available/ronautradio.la /etc/nginx/sites-enabled/`.
  - [ ] Test and restart Nginx. The site should be accessible via `http://ronautradio.la`.

- [ ] **Task 5: Pre-Flight Checks for SSL**
  - [ ] **DNS:** Use an online tool (like `dnschecker.org`) to confirm `ronautradio.la` and `www.ronautradio.la` both point to your server's IP address (`89.117.16.160`).
  - [ ] **Server Firewall:** Check the server's firewall status: `sudo ufw status`. If it's active, ensure ports 80 and 443 are allowed.
  - [ ] **Cloud Firewall:** Log in to your Contabo control panel and ensure there are no external firewall rules blocking inbound traffic on TCP ports 80 and 443.

---

### Phase 3: HTTPS Activation & Final Integration
*This phase secures the site with an SSL certificate and integrates the media player proxies.*

- [ ] **Task 6: Secure Site with Certbot**
  - [ ] Install Certbot and the Nginx plugin: `sudo apt install certbot python3-certbot-nginx -y`.
  - [ ] Run Certbot to automatically obtain a certificate and configure Nginx for HTTPS: `sudo certbot --nginx -d ronautradio.la -d www.ronautradio.la`.
  - [ ] Verify the site now loads correctly on `https://ronautradio.la`.

- [ ] **Task 7: Add Media Player Proxies to Nginx**
  - [ ] Edit the newly updated Nginx config: `sudo nano /etc/nginx/sites-available/ronautradio.la`.
  - [ ] Add the final `location` blocks for `/hls/` and `/now-playing` to the `server` block that listens on port 443.
  - [ ] Test Nginx configuration (`sudo nginx -t`) and restart the service (`sudo systemctl restart nginx`).

- [ ] **Task 8: Final Testing**
  - [ ] Open `https://ronautradio.la` in a browser.
  - [ ] Verify the media player loads and plays the stream without errors.
  - [ ] Check the browser's developer console for any "mixed content" warnings.
  - [ ] Verify the live chat is functional.

- [ ] **Task 9: Housekeeping**
    - [ ] Push the new `Architecture.md` and `task.md` files to the GitHub repository. 