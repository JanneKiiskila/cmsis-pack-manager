# cmsis-pack-manager
cmsis-pack-manager is a python module, Rust crate and command line utility for managing current device information that is stored in many CMSIS PACKs. Users of cmsis-pack-manager may query for information such as processor type, flash algorithm and memory layout information in a python program or through the command line utility, `pack-manager`, provided as part of this module.

# CI Status
[![Windows Build status](https://ci.appveyor.com/api/projects/status/tltovxvu20y4pma8?svg=true)](https://ci.appveyor.com/project/theotherjimmy/cmsis-pack-manager) [![Mac and Linux Build Status](https://travis-ci.org/ARMmbed/cmsis-pack-manager.svg?branch=master)](https://travis-ci.org/ARMmbed/cmsis-pack-manager)

## Wheels

The last step of CI uploads binary wheels to [this S3 bucket.](http://mbed-os.s3-website-eu-west-1.amazonaws.com/?prefix=builds/cmsis-pack-manager/dist/)

# DOCS!

They live here: https://armmbed.github.io/cmsis-pack-manager/

# Building

To build cmsis-pack-manager locally, Install a stable rust compiler.
See https://rustup.rs/ for details on installing `rustup`, the rust
toolchain updater. Afterwards, run `rustup update stable` to get the
most recent stable rust toolchain and build system.

After installing the rust toolchain and downloading a stable compiler,
run `python2 setup.py bdist_wheel` from the root of this repo to
generate a binary wheel (`.whl` file) in the same way as we release.

For testing purposes, there is a CLI written in Rust within the rust
workspace as the package `cmsis-cli`. For example From the `rust`
directory, `cargo run -p cmsis-cli -- update` builds this testing
CLI and runs the update command, for example.

# Updating the CMSIS-packs in Mbed OS

Mbed OS has a very large [index.json](https://github.com/ARMmbed/mbed-os/blob/master/tools/arm_pack_manager/index.json) file, which contains the CMSIS-pack data.

Please note that file is maintained manually - the TI CC information has been manually typed in, so you have to do the updates manually, too.

The update process is slightly complicated due to the CMSIS-pack-manager issue [#121](https://github.com/ARMmbed/cmsis-pack-manager/issues/121) and requires currently you to use a special hack version of the CMSIS-pack-manager to avoid the TLS handshake issue which prevents the download of some CMSIS-packs from Azure.

The process is following:
1. Set up a [virtual environment](https://www.geeksforgeeks.org/python-virtual-environment/) Python.
2. Create a new or activate a suitable virtual environment.
    1. `virtualenv cmsis` (Create a new one).
    1. `source ~/venv/cmsis/bin/activate` (Activate the new env).
    1. `pip install -r mbed-os/requirements.txt` (Install basic Mbed OS tooling).
1. Get the fork of cmsis-pack-manager, i.e. `git clone https://github.com/JanneKiiskila/cmsis-pack-manager -b Rustless-fix`. You must use that branch, master of the official repo does not have the TLS workaround.
1. Install [`rust`](https://www.rust-lang.org/), if you do not have it yet.
1. Build and install the forked cmsis-pack-manager.
    1. `cd cmsis-pack-manager`
    1. `rustup update stable` (update Rust itself).
    1. Run `python2 setup.py bdist_wheel` (build the wheels aka binaries for PyPi).
    1. Install the custom PyPi package - `pip install -e .`.
    1. Verify PyPi package went in, `pip list |grep cmsis` and see it matches.
1. Move over to your Mbed OS repository, `cd ../mbed-os/tools`.
1. Run `python project.py --update-packs` to update the `index.json` file. **Note! This takes a long time**, as it will download all of the packs.
1. Isolate the changes you are interested in getting merged in. Find the start and end-point, copy paste that section somewhere safe and reset the index.json file back to what it was (git checkout index.json).
1. Locate a suitable spot where to add the CMSIS-pack changed you want, copy paste it there and save the file. Try to keep the order the same, so that delta is minimised.
1. Do a commit to a development branch:
    1. `git checkout -b cmsis-update-xxxx` (xxxx should be descriptive name).
    1. `git add index.json`.
    1. `git commit`.
1. Test your changes.
1. Do `git push` via your fork and prepare a normal PR to the Mbed OS repository.
1. Wait for maintainers to review and merge the change back in.

