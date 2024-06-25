# Personal Development Journal

_These are my personal notes. I do not expect anyone to read them. I write them for myself._

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