# How to Build MV3 CinderBlock

Instructions for reviewers.

The following assumes a linux environment.

1. Open Bash console
2. `git clone https://github.com/SabeeirSharrma/CinderBlock.git`
3. `cd CinderBlock`
4. `git submodule init`
5. `git submodule update`
6. `make mv3-[platform]`, where `[platform]` is either `chromium`, `edge`, `firefox`, or `safari`
7. This will fully build CinderBlock MV3, and during the process filter lists will be downloaded from their respective remote servers

Upon completion of the script, the resulting extension package will be present in:

- Chromium: `dist/build/cinderblock.mv3.chromium`
- Edge: `dist/build/cinderblock.mv3.edge`
- Firefox: `dist/build/cinderblock.mv3.firefox`
- Safari: `dist/build/cinderblock.mv3.safari`

The folder `dist/build/mv3-data` will cache data fetched from remote servers, so as to avoid fetching repeatedly from remote servers with repeated build commands. Use `make cleanassets` to remove all locally cached filter lists if you want to build with latest versions of filter lists.

The file `dist/build/cinderblock.mv3.[platform]/log.txt` will contain information about what happened during the build process.

The entry in the `Makefile` which implements the build process is `tools/make-mv3.sh [platform]`. This Bash script copies various files from CinderBlock's main branch and MV3-specific branch into a single folder which will be the final extension package.

Notably, `tools/make-mv3.sh [platform]` calls a Node.js script which purpose is to convert the filter lists into various rulesets to be used in a declarative way. The Node.js version required is 17.5.0 or above.

All the final rulesets are present in the `dist/build/cinderblock.mv3.[platform]/rulesets` in the final extension package.
