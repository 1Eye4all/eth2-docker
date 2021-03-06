# eth2-docker v0.1.7.5
Unofficial and experimental docker build instructions for eth2 clients

## Acknowledgements

Parts of this guide are based on the Linux [guides](https://medium.com/@SomerEsat) written by [Somer Esat](https://twitter.com/SomerEsat).

Without their previous work, this project would not exist.

## Supported clients

This project builds clients from source. A similar workflow for
binary images is a TODO, as long as it does not duplicate work
by client teams.

Currently supported clients:
- Lighthouse
- Prysm
- Teku
- Nimbus

Currently supported optional components:
- geth, local eth1 node. Use this or a 3rd-party provider of eth1 chain data to "feed"
  your eth2 beacon node, so you can [propose](https://ethos.dev/beacon-chain/) blocks.
- Grafana dashboard

**Note**: As of 10/09/2020, Nimbus requires an "archive" source of eth1 chain data.
This can be a local geth in archive mode via ws://, or a 3rd-party provider via wss://. 
Please see [SETUP](SETUP.md) for details.

Please see [WEB](WEB.md) for experimental Web UI support on Prysm, and use the Web instead
of validator-import to import keys.

# USAGE

## Before you start

Warnings about the dangers of running eth2 validators are in [RECOMMENDATIONS.md](RECOMMENDATIONS.md).
That file also contains a link to SomerEsat's [guide on host security](https://medium.com/@SomerEsat/guide-to-staking-on-ethereum-2-0-ubuntu-medalla-nimbus-5f4b2b0f2d7c), and comments on key security.
Please take a look.

## Steps to bring an eth2 node up

1. Install prerequisites
2. Choose a client and do initial setup.
3. Build the client
4. Generate deposit files and an eth2 wallet. This can be done within this project, or outside of it
5. Import the validator keystore files generated in the previous step
6. Run the client
7. Finalize the deposit. This is not done within this project
8. Set up Grafana dashboards (optional)
9. Configure your system to start the eth2 node on boot (optional)

## Step 1: Install prerequisites

You will need git, docker, and docker-compose. This should work on Linux, possibly MacOS.
Running a node on Windows 10 is highly discouraged because of time synchronization concerns.
Please see [prerequisites](PREREQUISITES.md) and then come back here.

## Step 2: Choose a client, initial setup

Next, choose a client and configure this project to use it.
Please see [setup instructions](SETUP.md) and then come back here.

## Step 3: Build the client

> **Important** Before you build, verify once more that `LOCAL_UID` in `.env`
> is the UID of your user (check with `echo $UID`), and that the file `.env`
> (dot env) is called exactly that, and contains the parameters you expect.
> You will get errors about missing permissions, during Step 4, if the UID is wrong.

Build all required images from source. This will take 20-30 minutes. Assuming
you cloned the project into `eth2-docker`:
```
cd ~/eth2-docker
sudo docker-compose build
```

## Step 4: Create an eth2 wallet and validator keystore and deposit files

You will deposit eth to the deposit contract, and receive locked eth2 in turn.<br />
> **Vital** [RECOMMENDATIONS.md](RECOMMENDATIONS.md) has comments on key security. If you haven't
read these yet, please do so now. You need to know how to guard your keystore password
and your seed phrase (mnemonic). **Without the mnemonic, you will be unable to withdraw your funds
in phase 2. You need the seed phrase or your eth is gone forever.**

Make sure you're in the project directory, `cd ~/eth2-docker` by default.

Edit the `.env` file to set the number of validators you wish to run. The default
is just one (1) validator.

This command will get you ready to deposit eth:<br />
`sudo docker-compose run deposit-cli`

The created files will be in the directory `.eth2/validator_keys` in this project.
This is also where you'd place your own keystore files if you already have some for import.

### You brought your own keys

They go into `.eth2/validator_keys` in this project directory, not directly under `$HOME`.

## Step 5: Create a validator wallet by importing validator keys

**Warning** Import your validator key(s) to only *one* client.

> If you want to use the experimental [Prysm Web UI](WEB.md), use it to
> import keys and not this command-line process.

Import the validator key(s) to the validator client:

`sudo docker-compose run validator-import`

> #### Prysm-specific
> - You will be asked to provide a wallet directory. Use `/var/lib/prysm`.
> - You will be asked to provide a "New wallet password", independent of the
>   keystore password. 
> - If you choose not to store the wallet password with the validator,
>   you will need to edit `prysm-base.yml` and comment out the wallet-password-file
>   parameter

If you choose to save the password during import, it'll be available to the client every
time it starts. If you do not, you'll need to be present to start the
validator and start it interactively. Determine your own risk profile.

## Step 6: Start the client

To start the client:
```
sudo docker-compose up -d eth2
```
> **Nimbus and Teku**: Beacon and validator run in the same process, there is only one container for both

If, however, you chose not to store the wallet password with the validator, you will need
to bring the beacon and, if in use, geth, up individually instead, then "run"
the validator so it can prompt you for input:

```
sudo docker-compose up -d geth beacon
sudo docker-compose run validator
```

After providing the wallet password, use the key sequence Ctrl-p Ctrl-q to detach
from the running container.

## Step 7: Depositing

Optional: You may wish to wait until the beacon node is fully synchronized before you deposit. Check
its logs with `sudo docker-compose logs -f beacon`. This safe-guards against the validator being
marked offline if your validator is activated before the beacon syncs.

Once you are ready, you can send eth to the deposit contract by using
the `.eth2/validator_keys/deposit_data-TIMESTAMP.json` file at the [Medalla launchpad](https://medalla.launchpad.ethereum.org/).

## Step 8: Grafana Dashboards

I'll repeat /u/SomerEsat's instructions on how to set up Grafana. 
- Connect to http://YOURSERVERIP:3000/, log in as admin/admin, set a new password
- Click on the gear icon on the left, choose "Data Sources", and "Add Data Source". Choose Prometheus, use `http://prometheus:9090` as the URL, then click "Save and Test".
- Import a Dashboard. Click on the + icon on the left, choose "Import". Copy/paste JSON code from one of the client dashboard links below (click anywhere inside the page
the link gets you to, use Ctrl-a to select all and Ctrl-C to copy), click "Load", choose the "prometheus" data source you just configured, click "Import".
- For Teku, you can use the grafana.com URL instead of raw JSON.

- [Lighthouse Dashboard JSON](https://raw.githubusercontent.com/sigp/lighthouse-metrics/master/dashboards/Summary.json)
- [Metanull's Prysm Dashboard JSON](https://raw.githubusercontent.com/metanull-operator/eth2-grafana/master/eth2-grafana-dashboard-single-source.json)
- [Prysm Dashboard JSON](https://raw.githubusercontent.com/GuillaumeMiralles/prysm-grafana-dashboard/master/less_10_validators.json)
- [Prysm Dashboard JSON for more than 10 validators](https://raw.githubusercontent.com/GuillaumeMiralles/prysm-grafana-dashboard/master/more_10_validators.json)
- [Nimbus Dashboard JSON](https://raw.githubusercontent.com/SomerEsat/ethereum-staking-guide/master/NimbusGrafana.json)
- [Teku Dashboard](https://grafana.com/grafana/dashboards/12199)

## Step 9: Autostart the client on boot

If you are using the "unless-stopped" restart policy, docker will start the 
client for you. That said, you may wish to make sure this happens on
startup no matter what.

For Linux systems that use systemd, e.g. Ubuntu, you'd create a systemd
service. 

- Copy the file: `sudo cp sample-systemd /etc/systemd/system/eth2.service`
- Edit the file: `sudo nano /etc/systemd/system/eth2.service`
- Adjust the `WorkingDirectory` to the directory you stored the project in.
- Adjust the path to `docker-compose` to be right for your system, see `which docker-compose`
- Test the service: From within the project directory, `sudo docker-compose down` to shut
  it down, then `sudo systemctl daemon-reload`, `sudo systemctl start eth2` to bring it back up,
  and check `sudo docker ps` to see all expected containers are up.
- Enable the service: `sudo systemctl enable eth2`

## Addendum: Monitor the client

Monitoring the logs of the client is useful for troubleshooting
and to judge the amount of time left before the beacon and geth nodes
are fully synchronized.

To see a list of running containers:

```
sudo docker ps
```

To see the logs of a container:

```
sudo docker logs -f CONTAINERNAME
```

or

```
sudo docker-compose logs -f SERVICENAME
```

## Addendum: Update the software

This project does not monitor client versions. It is up to you to decide that you
are going to update a component. When you are ready to do so, the below instructions
show you how to.

Please see the [changelog](CHANGELOG.md) to see what changed between versions.

### The project itself

Inside the project directory, run:<br />
`git pull`

Then `cp .env .env.bak` and `cp default.env .env`, and set variables inside `.env`
the way you need them, with `.env.bak` as a guide.

### Geth

Run:<br />
`sudo docker-compose build --no-cache geth`

Then stop, remove and start geth:<br />
`sudo docker-compose stop geth && sudo docker-compose rm geth`<br />
`sudo docker-compose up -d geth`

### Client

Beacon and validator share the same image, we only need to rebuild one.

Run:<br />
`sudo docker-compose build --no-cache beacon`

Then restart the client:<br />
`sudo docker-compose down && sudo docker-compose up -d eth2`

If you did not store the wallet password with the validator, come up 
[more manually](#start-the-client) instead.

## Addendum: Voluntary client exit

Ethereum 2.0 has a concept of "voluntary client exit", which will remove the
validator from attesting duties. Locked eth could be withdrawn in phase 2,
and not sooner.

Currently, Prysm supports voluntary exit. This requires a fully synced
Prysm client.

To exit, run `sudo docker-compose run validator-voluntary-exit` and follow the
prompts.

If you wish to exit validators that were running on other clients, you can do this
as follows:

- Stop the other client(s), and wait 20 minutes. This is so you won't have
  a validator attest in the same epoch twice.
- Copy all `keystore-m` JSON files into `.eth2/validator_keys` in this project
  directory.
- Stop the Prysm client in this project, `sudo docker-compose down`
- Import the new keys via `sudo docker-compose run validator-import`. Note
  that Prysm assumes they all have the same password. If that's not the case,
  maybe work in batches.
- Verify once more that the old client is down, has been for 20 minutes, and
  can't come back up. **If both the old client and this Prysm run at the same time,
  you will slash yourself**
- Bring the Prysm client up: `sudo docker-compose up -d eth2`
- Check logs until the beacon is synced: `sudo docker-compose logs -f beacon`
- Initiate voluntary exit and follow the prompts: `sudo docker-compose run validator-voluntary-exit`

> Note you will need to continue running your validator until the exit
> has been processed by the chain, if you wish to avoid incurring offline
> penalties. You can check the status of your validator with tools such
> as beaconcha.in and beaconscan.

## Addendum: Troubleshooting

A few useful commands if you run into issues. As always, `sudo` is a Linux-ism and may not be needed on MacOS.

`sudo docker-compose stop servicename` brings a service down, for example `docker-compose stop validator`.<br />
`sudo docker-compose down` will stop the entire stack.<br />
`sudo docker-compose up -d servicename` starts a single service, for example `sudo docker-compose up -d validator`.
The `-d` means "detached", not connected to your input session.<br />
`sudo docker-compose run servicename` starts a single service and connects your input session to it. Use the Ctrl-p Ctrl-q
key sequence to detach from it again.

`sudo docker ps` lists all running services, with the container name to the right.<br />
`sudo docker logs containername` shows logs for a container, `sudo docker logs -f containername` scrolls them.<br />
`sudo docker-compose logs servicename` shows logs for a service, `sudo docker-compose logs -f servicename` scrolls them.<br />
`sudo docker exec -it containername /bin/bash` will connect you to a running service in a bash shell. The geth service doesn't have a shell.<br />

You may start a service with `sudo docker-compose up -d servicename` and then find it's not in `sudo docker ps`. That means it terminated while
trying to start. To investigate, you could leave the `-d` off so you see logs on command line:<br />
`sudo docker-compose up beacon`, for example.<br />
You could also run `sudo docker-compose logs beacon` to see the last logs of that service and the reason it terminated.<br />

If a service is not starting and you want to bring up its container manually, so you can investigate, first bring everything down:<br />
`sudo docker-compose down`, tear down everything first.<br />
`sudo docker ps`, make sure everything is down.<br />

If you need to see the files that are being stored by services such as beacon, validator, eth1 node, grafana, &c, in Ubuntu Linux you can find
those in /var/lib/docker/volumes. `sudo bash` to invoke a shell that has access.

**HERE BE DRAGONS** You can totally run N copies of an image manually and then successfully start a validator in each and get yourself slashed.
Take extreme care.

Once your stack is down, to run an image and get into a shell, without executing the client automatically:<br />
`sudo docker run -it --entrypoint=/bin/bash imagename`, for example `sudo docker run -it --entrypoint=/bin/bash lighthouse`.<br />
You'd then run Linux commands manually in there, you could start components of the client manually. There is one image per client.<br />
`sudo docker images` will show you all images.

# Guiding principles:

- Reduce the attack surface of the client as much as feasible.
  None of the eth2 clients lend themselves to be statically compiled and running
  in "scratch" containers, alas.
- Guide users to good key management as much as possible
- Create something that makes for a good user experience and guides people new to docker and Linux as much as feasible

LICENSE: MIT
