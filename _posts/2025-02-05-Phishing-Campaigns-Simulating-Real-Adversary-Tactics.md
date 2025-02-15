---
layout: post
title: "Phishing Campaigns: Simulating Real Adversary Tactics"
date: 2025-02-05
categories: [phishing, redteam, gophish]
tags: [phishing, security, redteam, devops, gophish]
---

Nowadays, one of the most dangerous and effective attack vectors is the phishing campaigns. From an offensive perspective, there are several differences between a whaling campaign and a massive phishing campaign. However, there are also several common points that should be accomplished in both cases.

![](/img/2025-02-05.png)

At security professional, one of the goals is to help improve security in the industry. To achieve this target, annual phishing campaigns works well to enhance the detection and reporting rates of entities and companies.

The post will cover several phases of these engagements and provide some details on how we work to achieve our goals.

## Infrastructure

Starting with a Red Team Infrastructure model, team server and redirectors are used. Redirectors play a crucial role in Red Team engagements, as they are used to conceal the actual IP addresses and route traffic through intermediate points before reaching the target network. This obfuscation enhances the red team’s stealth during attacks, making it more challenging for defenders to detect and respond to their activities. Redirectors are essential for maintaining the security of client information on the team server, which is allocated in the company’s internal network.

The team server serves as a centralized command center that communicates with compromised systems or “agents” deployed on target networks. In the context of a phishing campaign, the team server is employed to run the applications and tools necessary for conducting the engagement.

To achieve this, we will be utilizing [Traefik](https://doc.traefik.io/traefik/) as a redirector. Traefik will handle SSL certificates and help route traffic securely. Additionally, [GoPhish](https://github.com/gophish/gophish) will be used to send emails and manage the campaign effectively. To establish seamless and secure connections between the elements, we will use [Headscale](https://github.com/juanfont/headscale), which is a self-hosted alternative compatible with [Tailscale](https://tailscale.com/). This combination of tools ensures that data is protected, and the engagement is conducted with utmost safety and confidentiality in mind.

## Redirector
As a redirector, a Linux host in the cloud was utilized. There are various trusted platforms available for this purpose, and the choice depends on individual preferences and requirements.

To run Traefik, Docker needs to be installed as it serves as the containerization platform for Traefik. According to the website, Traefik is an open-source Edge Router that simplifies the process of publishing your services. It acts as a request receiver on behalf of your system and determines which components are responsible for handling those requests. Developed in Golang, Traefik is known for being lightweight and faster than other commonly used options like Apache and Nginx.

Although, we will not go over the configuration files, in this case, traefik used two different files `traefik.toml`, where the server configuration, such as ports or logs paths, is stored. The second file is `traefik_dynamic.toml`, which is referenced in `traefik.toml`. This file is responsible for handling the SSL configuration and the behavior of the redirector.

By leveraging Docker and these configuration files, Traefik provides a user-friendly experience for managing and routing your services efficiently.

Additionally, Traefik has built-in capabilities to handle SSL certification. However, for our setup, we will generate SSL certificates using cerboot. The main goal of this post is to learn how to deploy a full phishing infrastructure. To reduce the detection rate, it should not use cerbotot. A self-signed SSL certificate in combination with Cloudflare is a better option.

To ensure all the software needed is installed in the host, the script ‘redirector.sh’ will install cerboot, docker, tailscale and other software required to set up our redirector correctly.

To ensure that all the necessary software is installed on the host, we have created a convenient script called `redirector.sh`. This script will install cerboot, Docker, Tailscale, and any other required software to set up our redirector correctly.

The SSL certificates generated with cerboot will be stored in the `cert/` folder, while all the log files will be found in `log/`. Additionally, the `system/` folder will store files required by Traefik in case you want to run Traefik as a service on the host.

```bash
dan1t0@cybertron:~/dinam1t0$ ls -1
cert/
log/
redirector.sh
systemd
traefik.sh
traefik.toml
traefik_dynamic.toml
```

`traefik.sh` allows to build the configuration of the redirector in case it was not run previously.

```bash
dan1t0@cybertron:~/dinam1t0$ sudo ./traefik.sh build hack.attacker.com
- Genereting SSL certs with certboot...
  -> Checking IPs
  -> Local IP: X.X.X.X (hidden)
  -> hack.attacker.com point to X.X.X.X (hidden)
cerbot command -> certbot certonly --standalone -d hack2.attacker.com --staple-ocsp --agree-tos --register-unsafely-without-email

- SSL certificates generated correctly
- Certificates found
- Running docker containers
  -> Cleaning previous version
     docker rm -f traefik
  -> Deploying docker
docker run -d -v /var/run/docker.sock:/var/run/docker.sock -v /home/dan1t0/dinam1t0/log:/log -v /home/dan1t0/dinam1t0/cert:/cert -v /home/dan1t0/dinam1t0/traefik.toml:/traefik.toml -v /home/dan1t0/dinam1t0/traefik_dynamic.toml:/traefik_dynamic.toml -p 80:80 -p 443:443 --name traefik --hostname traefik traefik:v2.10
traefik
fbc5ed8f95e4ac284ce0a8f4e9749946d7b22531646993f50b86a280fabdbff5

dan1t0@cybertron:~/dinam1t0$ sudo docker ps
CONTAINER ID   IMAGE           COMMAND                  CREATED         STATUS         PORTS                                                                      NAMES
fbc5ed8f95e4   traefik:v2.10   "/entrypoint.sh trae…"   9 seconds ago   Up 6 seconds   0.0.0.0:80->80/tcp, :::80->80/tcp, 0.0.0.0:443->443/tcp, :::443->443/tcp   traefik
```

While the script can be executed with just one domain name, it also accepts multiple domain names as input.

Although, this blogpost is not a tutorial about how traefik works, we will focus on discussing two of the most crucial aspects of the configuration in `traefik_dynamic.toml`:

```toml
[http.routers.gophish]
  rule = "Host(`hack.attacker.com`) && Query(`ogt=`) && !HeadersRegexp(`User-Agent`, `(?i:wget|curl|HTTrack|crawl|google|bot|b-o-t|spider|baidu)`)"
  entrypoints = ["websecure"] # ["web"]
  service = "gophish"
  priority = 30
  [http.routers.gophish.tls]

[http.services.gophish]
  [http.services.gophish.loadBalancer]
    [[http.services.gophish.loadBalancer.servers]]
      url = "http://gophish:8080/"
```

`[http.routers.gophish]` is the section in the configuration where the redirector rules are defined. Within this section, various options supported by Traefik can be utilized, such as filtering requests based on user-agent or adding special strings to the URL, in this example the string ogt is used. The explanation about the string value will be explained below.

Filtering traffic by the User-Agent header in a phishing campaign is an important defensive measure to detect and potentially block or limit the Blue Team activities. The User-Agent header is a part of the HTTP request sent by web browsers, applications, or automated scripts when they communicate with a web server. It contains information about the client’s identity, such as the type of browser or application and its version. In addition to Blue Teams, it is recommended to add user-agent of bots to avoid be flagged easily.

`[http.services.gophish]` sets the internal path where the request that meet our requirements are sent. In our case, it will be sent to `http://gophish:8080/` where our Gophish instance is running internally. The hostname is configured during the Tailscale configuration process.

## GoPhish
GoPhish is a well-known tool for conducting phishing campaigns. However, it is recommended to add some modifications in the original source to remove IOCs.

These IOCs are potential signs of a security breach or malicious activity that could be used by defenders or security tools to detect and respond to phishing attempts.

By removing IOCs from the source code, the phishing campaigns become more challenging for defenders to detect, allowing the red team to maintain a higher level of stealth and effectiveness during engagements.

A good starting point to remove the GoPhish IOCs is [Sneaky GoPhish](https://github.com/puzzlepeaches/sneaky_gophish) where the following lines are modified:

```bash
# Stripping X-Gophish 
sed -i 's/X-Gophish-Contact/X-Contact/g' models/email_request_test.go
sed -i 's/X-Gophish-Contact/X-Contact/g' models/maillog.go
sed -i 's/X-Gophish-Contact/X-Contact/g' models/maillog_test.go
sed -i 's/X-Gophish-Contact/X-Contact/g' models/email_request.go

# Stripping X-Gophish-Signature
sed -i 's/X-Gophish-Signature/X-Signature/g' webhook/webhook.go

# Changing servername
sed -i 's/const ServerName = "gophish"/const ServerName = "IGNORE"/' config/config.go

# Changing rid value
sed -i 's/const RecipientParameter = "rid"/const RecipientParameter = "ogt"/g' models/campaign.go

```

The parameter `rid` is added to the URL of the link sent to the targeted user. In the previous example, it was changed to the highlighted string `ogt`.

This string is added in the configuration of the redirector in the traefik configuration files. As a result, the URL of the cloned portal location and the pixel tracking sent to the target will be modified by this `http://hack.attacker.com/?ogt={randomToken}`.

Once the changes are made, gophish source code is ready to be built by executing `go build`.

To boost our phishing infrastructure, we can monitor Gophish using additional scripts running in our internal server. These scripts will enable us to receive real-time alerts every time a user submits their credentials. We can achieve this by leveraging tools like Pushover, which will provide instant notifications.

Another effective strategy is to use Sendinblue or Mailgun over SMTP or explore other alternatives to get instant karma in our emails and save time that we can use to improve our ruses.

In the future, we will share some Gophish tricks to further enhance the efficiency and effectiveness of our campaigns.
