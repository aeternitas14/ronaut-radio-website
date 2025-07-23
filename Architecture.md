# Ronaut Radio - Deployment Architecture

This document outlines the technical architecture for deploying the Ronaut Radio website on a Contabo Cloud VPS. The primary goal is to create a robust, secure, and scalable environment that can serve the website, a live chat, and a proxied media stream flawlessly.

## Core Components

The deployment consists of four main components running on a single Ubuntu server.

### 1. The Server (Contabo VPS)
- **Provider:** Contabo
- **Operating System:** Ubuntu 22.04 LTS
- **Role:** The foundational layer. It hosts all other components and provides us with the necessary root access to install software and configure the environment.

### 2. Nginx (The Web Server & Reverse Proxy)
- **Role:** The public-facing "front door" for all web traffic to `ronautradio.la`. It is the most critical piece of the puzzle.
- **Responsibilities:**
    - **SSL Termination:** It will handle all HTTPS traffic, decrypting it using a valid Let's Encrypt certificate before passing it to the correct backend service.
    - **Static File Serving:** It will directly serve the static assets of the website (HTML, CSS, JavaScript, images) for maximum performance.
    - **Reverse Proxy for Chat:** It will receive requests for the live chat and forward them to the background Node.js application running on `localhost:3000`. This includes managing the WebSocket connection for real-time messages.
    - **Reverse Proxy for Media:** It will intercept requests for `/hls/stream.m3u8` and `/now-playing`, fetch the content from the original `http://` sources in the background, and then serve that content to the user securely over HTTPS. This is the key to solving the mixed-content error and making the player work.

### 3. Node.js Application (The Chat Server)
- **Runtime:** Node.js
- **Framework:** Express.js with Socket.IO
- **Process Manager:** `pm2`
- **Role:** To provide the real-time backend logic for the live chat feature.
- **Responsibilities:**
    - It listens internally on a port (e.g., 3000), accepting connections only from the Nginx reverse proxy.
    - It uses Socket.IO to manage user connections, usernames, and the broadcasting of chat messages to all connected clients.
    - `pm2` ensures that the Node.js application runs continuously and restarts automatically if it ever crashes.

### 4. Certbot (The SSL Certificate Manager)
- **Provider:** Let's Encrypt
- **Role:** To automate the process of obtaining and renewing a free, trusted SSL/TLS certificate.
- **Responsibilities:**
    - It interfaces with Nginx to automatically prove ownership of the `ronautradio.la` domain to Let's Encrypt.
    - It configures Nginx to use the generated certificate, enabling HTTPS.
    - It sets up a cron job to automatically renew the certificate before it expires, ensuring the site remains secure without manual intervention.

## Traffic Flow Diagram

```mermaid
graph TD
    subgraph "User's Browser"
        A[User visits https://ronautradio.la]
    end

    subgraph "Internet"
        B(Firewall)
    end
    
    subgraph "Contabo VPS (Your Server)"
        C(Nginx)
        D[Node.js Chat Server <br/> (pm2 on localhost:3000)]
        E[HLS Stream Source <br/> (http://...)]
        F[Now Playing API <br/> (http://...:5050)]
    end

    A --> B
    B --> C

    C -- Serves static files --> A
    C -- Proxies chat connection --> D
    D -- Real-time messages --> C

    C -- Proxies request for /hls/ --> E
    E -- Returns stream data --> C
    C -- Serves secure stream --> A

    C -- Proxies request for /now-playing --> F
    F -- Returns track info --> C
    C -- Serves secure track info --> A
``` 