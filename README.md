# dosxer-sync

Dosxer sync aims to handle the problems currently exisiting with Docker for Mac and its slow `osxfs` implementation.

## Installation

Run `dosxer_sync_setup`. This will install all needed prerequisites.

### What it does

* Checks if `brew` is available (see http://brew.sh/ for installation instructions)
* Installs `unison` via `brew` for the two-way-sync (see https://github.com/bcpierce00/unison)
* Installs `terminal-notifier` via `brew` for Desktop notifications (see https://github.com/julienXX/terminal-notifier)
* Installs `MacFSEvents` via `pip` for filesystem event handling (might require `sudo` privileges when using stock Python; then please manually install it) (see https://github.com/malthe/macfsevents)
* Installs `unox` as glue for `unison` and filesystem events (see https://github.com/hnsl/unox)
* Install `dosxer_sync` to `/usr/local/bin` (optional)

## Usage

Basically you only need to add an additional `docker-compose` file that defines the extra container providing the faster volume.

```
  app:
    volumes_from:
      - dosxer
  dosxer:
    container_name: project_dosxer
    image: onnimonni/unison:latest
    environment:
      - UNISON_DIR=/app
      - UNISON_UID=501 # Default UID of the OSX user; might be overwritten with an .env file
    volumes:
      - /app
    ports:  
      - "5000"
```

If you bind also the host port please ensure that it is free. `dosxer-sync` normally handles this for you.

Currently the convention is that the file needs to be named `docker-compose-dev.yml` and the base file must read `docker-compose.yml`.
The sync container name also has to contain `dosxer` so it can be detected to extract the local port assignement.

After that you can just run the `dosxer_sync` script from within the directory you have setup the `docker-compose` stack.

>Note: If you want you can also link the script to a folder in your `PATH` for global usage, e.g. `ln -s $(pwd)/dosxer_sync /usr/local/bin/dosxer_sync`.

### What it does

`unsion` can be seen as a two-way `rsync`. It is very stable and mature.

`dosxer-sync` uses an intermediary container that abstracts away `unison` on the Docker side. Locally `unison` is run in client mode (connect via socket to the sync container) and propagates the changes to the local filesystem (of the folder defined to sync) as well as syncs the changes from the sync container back to the local filesystem.

As the sync is performed via `unsion` the volume exposed by the sync container is a native linux volume and hence very fast. `unison` itself is also very fast.

### Files generated

`dosxer-sync` creates some files to handling the local sync. These are:

* `.dosxer.pid` to hold the local unison process id
* `.dosxer_init` to mark a stack to be initially synced
* `dosxer.log` to hold the log for the local sync

## Known issues & limitations

* Sometimes if a a lot of files need to be synced the sync process hangs. Workaround is to restart the stack (via the `docker_sync` command). This likely is caused by a large amount of files (e.g. an Angular project with a lot of `node_modules`). Eventually it will sync properly.
* There is currently no configuration possible.
* There are some conventions that need to be followed for now.
* It's not very battle-proof yet.

## Alternatives

Yes, there are quite some alternatives available which all didn't fully fit as of what I thought is needed. Either they are a bit bloated or insufficient or I didn't get them working.

A non exhaustive list:

* https://github.com/EugenMayer/docker-sync : Offers also rsync and one-sided unison as strategies but has quite some dependencies
* https://github.com/onnimonni/docker-unison : Is basically what `dosxer-sync` automates and is also the image being used by `dosxer-sync`
* https://github.com/leighmcculloch/docker-unison : The base for the above project
* https://github.com/mickaelperrin/docker-magic-sync : This is in most parts incorporated in `docker-sync`
* https://github.com/brikis98/docker-osx-dev : Works only with `docker-machine` IIRC

## Credits

`dosxer-sync` is as written in alternatives heavily inspired by those projects. Namely `docker-unison` by [onnimonni](https://github.com/onnimonni) and `docker-sync` by [EugenMayer](https://github.com/EugenMayer) and [mickaelperrin](https://github.com/mickaelperrin).
