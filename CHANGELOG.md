# Changelog for eth2-docker project

## Updating the project

To update the components of the project, run from within the project
directoy (`cd ~/eth2-docker` by default):

* `git pull`
* `cp .env .env.bak && cp default.env .env`
* Adjust contents of new `.env`, use `.env.bak` for guidance (LOCAL_UID
  and COMPOSE_FILE are the most common variables that may need to be adjusted)
* `sudo docker-compose build --no-cache beacon` (go for a 30 minute walk)
* `sudo docker-compose down`
* !! If coming from Lighthouse v0.2.x, make changes as per notes for [v0.1.6](#v016-2020-10-09)
* !! If coming from Prysm alpha.29 or earlier, make changes as per notes for [v0.1.7](#v017-2020-10-15)
* `sudo docker-compose up -d eth2`

## v0.1.7.5 2020-10-29

* validator-import for Teku now understands Prysm export

## v0.1.7.4 2020-10-29

* Support experimental Prysm Web UI

## v0.1.7.3 2020-10-27

* Prysm change to remove creation of new protection DB, Prysm no longer has this flag

## v0.1.7.2 2020-10-23

* Prysm changes to allow creation of new protection DB and remove experimental web support while it is in flux

## v0.1.7.1 2020-10-16

* Prysm renamed `accounts-v2` to `accounts`, keeping pace with it

## v0.1.7 2020-10-15

* Added "validator-voluntary-exit" to Prysm, see [readme](README.md#addendum-voluntary-client-exit)
* Default restart policy is now "unless-stopped" and can be changed via `.env`
* Preliminary work to support Prysm Web UI, not yet functional
* Changed testnet parameter for Prysm to conform with alpha.29
* Use `--blst` with Prysm by default for faster sync speed
* Handles Terms Of Service for Prysm, user is prompted during validator-import, and choice is remembered
* If you are upgrading this project and you are using Prysm, please run `sudo docker-compose run validator`
  and accept the terms of use. You can then Ctrl-C that process and start up normally again. This step
  is not necessary if you are starting from scratch.

## v0.1.6 2020-10-09

* Support for Lighthouse v0.3.0, drop support for v0.2.x
  * Please note that Lighthouse v0.3.x makes a breaking change to the beacon
    db. You will need to sync again from scratch, after building the new v0.3.0
    beacon image. You can force this with 
    `sudo docker-compose down`, `sudo docker volume rm eth2-docker_lhbeacon-data`
    (adjust to your directory path if you are not in `eth2-docker`, see
    `sudo docker volume ls` for a list).
  * Likewise, the location of the validator keystore has changed. The fastest way
    to resolve this involves importing the keystore from scratch:
    `sudo docker volume rm eth2-docker_lhvalidator-data` (as before, adjust for
    your directory), and then import the keystore(s) again with
    `sudo docker-compose run validator-import`. Your keystore(s) need to be in
    `.eth2/validator_keys` inside the project directory for that.
  * When you have completed the above steps, bring up Lighthouse with
    `sudo docker-compose up -d eth2` and verify that the beacon started syncing
    and the validator found its public key(s) by observing logs:<br />
    `sudo docker-compose logs -f beacon` and `sudo docker-compose logs validator | head -30`,
    and if you wish to see ongoing validator logs, `sudo docker-compose logs -f validator`.
  * The beacon will sync from scratch, which will take about half a day. Your
    validator will be marked offline for that duration.
