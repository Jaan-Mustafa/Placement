# IP Address

An IP address identifies any device on a network — server, router, computer, or phone.

## How a Website Request Works

1. You type a domain (e.g., `google.com`)
2. DNS resolves it to an **IP address** (e.g., `142.250.80.46`)
3. Your browser connects to **that IP** — the server hosting the website
4. The server sends back the webpage

So when a website/app makes a request, the IP address it hits **is the address of the destination server**.

## Key Points

- One server can have **multiple IP addresses**
- One IP address can serve **multiple websites** (shared hosting / virtual hosts)
- Large services sit behind **load balancers** — the IP points to the balancer, not a single machine
- **Your own machine also has an IP** — the server uses it to send the response back
