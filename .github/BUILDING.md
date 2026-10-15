# Build and publish your extension store

`Build and publish extensions` builds signed release APKs and JARs, uploads them
to GitHub Releases in `Shogun05/extensions`, and updates its `repo` branch with
the matching Mihon catalogue. APKs are release assets, not committed files.

The first run rebuilds every extension (currently 1,431), including MangaRaw.
Builds run in batches of 40 with at most four batches running concurrently.
Subsequent pushes to `main` or `add-mangaraw` rebuild affected extensions.
The manual **Run workflow** action defaults to a full rebuild.

## One-time setup

Create a persistent signing key on your computer. Use the same password for the
keystore and key; retain the file and password for future updates.

```bash
mkdir -p ~/.config/shogun-extensions
keytool -genkeypair \
  -keystore ~/.config/shogun-extensions/extensions.jks \
  -storetype PKCS12 \
  -alias extensions \
  -keyalg RSA -keysize 2048 -validity 10000 \
  -dname "CN=Shogun05 Extensions"
```

In **Shogun05/extensions-source → Settings → Secrets and variables → Actions**,
create these repository secrets:

| Secret | Value |
| --- | --- |
| `SIGNING_KEY` | Base64 contents of the keystore, from the command below |
| `KEY_STORE_PASSWORD` | The password entered when creating the keystore |
| `EXTENSIONS_TOKEN` | A fine-grained GitHub personal access token with **Contents: Read and write** for **Shogun05/extensions** |

```bash
base64 -w0 ~/.config/shogun-extensions/extensions.jks
```

Paste that command's output into `SIGNING_KEY`. Do not commit the keystore or
password. `EXTENSIONS_TOKEN` is needed because the source workflow publishes to
a separate repository; its built-in GitHub token only covers the source repo.

Keep the existing `repo` branch in `Shogun05/extensions`.

## Run and download

Push the workflow and source changes to `add-mangaraw`, or merge them into `main`.
Enable Actions if GitHub prompts you. Open **Actions → Build and publish
extensions**. The workflow checks the credentials before starting build jobs.
For later manual runs, select the branch containing the workflow and enable
**Rebuild and publish every extension** when you want a full rebuild. GitHub's
manual-run UI requires the workflow to also exist on the default branch.

After all batches succeed:

- Each build job exposes an `extension-batch-N` Actions artifact.
- The publish job uploads APKs and JARs under
  <https://github.com/Shogun05/extensions/releases>.
- Mihon can install the published extensions using this store URL:

  ```text
  https://raw.githubusercontent.com/Shogun05/extensions/repo/index.pb
  ```

Use that branch URL rather than an older commit-pinned catalogue URL, so Mihon
can see updates. The catalogue stays unchanged if a build batch fails. The
publisher also verifies that every requested module has matching build artifacts.

The separate **Build MangaRaw APK** workflow produces a development APK artifact
only. It does not publish to the store. If that development APK was installed,
uninstall it before installing the release signed with your new persistent key.
