## Part A - Economical Evaluation of Kamal

Managed hosting solutions (like Heroku, Render, Fly.io, and also managed databases) hide the complexity needed to deploy and maintain a web application.
For that, they charge a premium - usually per container/app/service without much of a quantity discount.
Hence, your bill grows linearly with the number of services you deploy.

At some level, the combined "management costs" for all services becomes larger than if you invested your own time(=money) to manage the infrastructure.
If some new tool now helps you to deploy to unmanaged servers more easily, this level is reached earlier.
Kamal does exactly that.

Read [their vision](https://kamal-deploy.org/) to learn more about the benefits of Kamal.
Here, however, I focus on **when not to use Kamal**:

- I said Kamal **lowered** the break-even point to justify your own infrastructure management. However, it does not **eliminate** it.
If you deploy only a small number of services, the overhead of managing your own infrastructure might still be too high to break even.
 In this case, you should stick to managed services (from an economic perspective).
- For higly complex deployments, Kamal might not be the right tool. It was designed with Rails apps with an off-the-shelf database in mind.

For me, personally, Kamal is not the right tool because I manage only a few, low-traffic services.
So I rather enjoy a 7$ managed service on render.com, than spending 4$ on a VPS (doesn't matter) plus hours of setting up the VPS, the SSH keys, the Docker containers, the health checks, the backups, the monitoring, the scaling, etc.
Of course, the decision would look different if I had to decide between 100 containers on render (700$) versus way less containers on a VPS (because I could place multiple services on one VPS).

## Part B - Personal Development Journal

> These are my personal notes. I wrote this **while** I was working on it. 
> I do not expect anyone to read this.

I desperately wanted a free tool to deploy on cheaper (more dumb) servers compared to managed services like Heroku, Render, Fly.io.
Kamal promised this.

I knew from the beginning that I'd had to learn Docker.
No problem, this is something I should know anyway I guess.
[This official Docker tutorial](https://docs.docker.com/language/nodejs/containerize/) shows how to containerize a Node.js app.

Then came Kamal. Therefore I had to SSH into the server. 
1Password's ssh key implementation is great but doesn't work with the ssh client that Kamal uses.
So I fell back to the old-school way of saving the private key in a file and modified `./ssh/config` to use it.

```
Host <custom-name>
    HostName <ip address>
    User root
    IdentityFile <path/to/private_key_file>
```

Then, Kamal was able to start the setup process but failed to build the Docker image.
The reason was that Kamal is currently not compatible with Windows.

I tried GitBash under Windows - didn't work. 
I spent hours installing and configuring WSL2 - even more problems occured.

Then I converted the docker commands Kamal provides [in the documentation](https://kamal-deploy.org/docs/installation/) to a windows compatible format:

```bash
docker run -it --rm  -v "%cd%:/workdir" -v "C:\Users\kogli\.ssh:/root/.ssh" -v /var/run/docker.sock:/var/run/docker.sock ghcr.io/basecamp/kamal:latest setup
```

- `docker run -it --rm` stays the same.
- The working directory is mounted with `-v "%cd%:/workdir"` (windows cannot handle `${PWD}`).
- To use ssh keys, I simpley mounted the `.ssh` directory into the container. 
- The docker socket is mounted the same way as in the documentation.

Next problem: Kamal's health checks are somewhat implicit.
By default, it checks if app containers on the server are up and running by executing `curl -f http://localhost:3000/up` **within the container**.
So when you don't modify the health check

a) your app container must expose such an endpoint and
b) the container must have `curl` installed. **Spoiler:** In node-alpine images, `curl` is not installed by default.

### Gotchas while adding accessories

Note that accessories are NOT automatically added, updated, or removed when you run `kamal deploy`.
Instead, to add an accessory, you must first add it in the config file and then run `kamal accessory boot <my-accessory>`.

In contrast to docker compose, the accessories' hosts must be ip addresses, not service names.

### Open questions

To be honest, how can Kamal be the best open-source tool that we have?
It solves a problem that is shared by every single developer on this planet.
There should be more widespread solutions to this problem.
Paying managed services 10$ per month is okay, but not for every container (backend, frontend, database, redis, etc.).

How can I deploy a static frontend together with this backend service?
At first I thought Kamal would work almost exactly like Docker Compose.
However, the big difference is that Kamal is intended(/just capable?) to deploy to multiple servers.

<details>
    <summary>My failed experiment to add Docker-like networks</summary>

[This article](https://guillaumebriday.fr/how-to-deploy-rails-with-kamal-postgresql-sidekiq-and-backups-on-a-single-host) mentions the network option like in Docker Compose (Docker Compose creates a shared network for all services by default).
I am trying this now. I receive a Docker error: "Network not found".
[This blog post](https://www.fromthekeyboard.com/single-server-kamal-deployments-with-a-docker-network/)
reassures that Kamal is not made for that purpose.
So I stop this experiment here.
</details>