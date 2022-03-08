# Logseq Flatpak

## How to update to a new version

Set the VERSION environment variable

```shell
export VERSION=0.5.7
```

Download the latest release from <https://github.com/logseq/logseq/releases>

```shell
curl -L -o logseq-$VERSION.tar.gz https://github.com/logseq/logseq/archive/refs/tags/$VERSION.tar.gz
```

Generate the sha256sum

```shell
sha256sum logseq-$VERSION.tar.gz
```

Update the release url and the sha257sum in the `com.logseq.Logseq.json` file.

Uncompress the file

```shell
tar xf logseq-$VERSION.tar.gz
```

Generate `generated-sources.json` use the `flatpak-node-generator.py` script from
[flatpak-builder-tools](https://github.com/flatpak/flatpak-builder-tools)

```shell
python3 ~/r/flatpak-builder-tools/node/flatpak-node-generator.py -r yarn \
  logseq-$VERSION/yarn.lock --xdg-layout -o generated-sources.json
```

Finally we may also need to update the clojure dependencies.

First we need to build the new release to force download dependencies.

```shell
rm -rf ~/.m2/repository
cd logseq-$VERSION
yarn
yarn gulp:build && yarn cljs:release-electron
```

Then update the `maven-sources.json` file

```shell
python3 flatpak-clj-generator-from-cache.py > maven-sources.json
```

Finally, test the build

```shell
flatpak-builder --user --install --force-clean build-dir/ com.logseq.Logseq.json
```

If all goes well, we can commit and push a new release.
