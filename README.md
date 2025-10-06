## macOS Packer Templates for Tart

Repository with Packer templates to build macOS [Tart](https://tart.run/) virtual machines to use with [Cirrus Runners](https://cirrus-runners.app/),
[Cirrus CI](https://cirrus-ci.org/guide/macOS/) or [any other automation](https://tart.run/integrations/cirrus-cli/).

The following image variants are currently available:

* `macos-{tahoe,sequoia,sonoma}-vanilla` — a vanilla macOS installation with helpful tweaks such as auto-login, but no additional software preinstalled
* `macos-{tahoe,sequoia,sonoma}-base` — based on `macos-{tahoe,sequoia,sonoma}-vanilla` image, it comes with `brew` and [other useful software](https://github.com/cirruslabs/macos-image-templates/blob/main/templates/base.pkr.hcl) pre-installed, but without Xcode
* `macos-{tahoe,sequoia,sonoma}-xcode:N` — based on `macos-{tahoe,sequoia,sonoma}-base` image and has `Xcode N` with [`Flutter`](https://flutter.dev/) pre-installed
* `macos-runner:{tahoe,sequoia,sonoma}` — a variant of `xcode:N` with several versions of `Xcode` pre-installed and [`xcodes` tool](https://github.com/XcodesOrg/xcodes) to switch between them.

See a full list of VMs available [here](https://github.com/orgs/cirruslabs/packages?tab=packages&q=macos-).

---

## 🧱 Building a Custom PHP Image (SimplePark)

The `xcode-php.pkr.hcl` template builds a macOS image based on Ciruss' base templates, with **Xcode, Flutter, PHP 8.4, and Composer** preinstalled.

### 🧩 Prerequisites

Make sure you have the following installed locally:

```bash
brew install hashicorp/tap/packer
brew install cirruslabs/tart/tart
```

Authenticate with GitHub Container Registry (GHCR) using a Personal Access Token (PAT) with `read:packages` and `write:packages` scopes:
```bash
export CR_PAT=<your_personal_access_token_with_write_packages>
echo "$CR_PAT" | tart login ghcr.io --username <your_github_username> --password-stdin
```

---

## ⚙️ Prepare the Xcode installer

Before building, you need a local cache of the Xcode .xip installer:

```bash
brew install xcodesorg/made/xcodes aria2
xcodes download 16.4 --directory ~/XcodesCache

# Move file to remove suffix
mv ~/XcodesCache/Xcode-16.4.0+16F6.xip ~/XcodesCache/Xcode_16.4.xip
```

---

### 🛠️ Build the macos-sequoia-xcode-php:16.4 image

Pull the base image

```bash
tart pull ghcr.io/cirruslabs/macos-sequoia-base:latest
```

Initialize the Packer environment (downloads the Tart plugin)

```bash
packer init templates/xcode.pkr.hcl
```

Runs the build for macOS Sequoia with Xcode 16.4:

```bash
packer build \
  -var macos_version="sequoia" \
  -var xcode_version='["16.4"]' \
  -var php_version="8.4" \
  templates/xcode-php.pkr.hcl
```

This will
* Pull the `macos-sequoia-base` image.
* Install Xcode 16.4 and related iOS components
* Install Flutter (stable), Android SDK, and developer tools.
* Install PHP 8.4 and Composer via Homebrew.
* Package everything into a new Tart image: `sequoia-xcode-php:16.4`

---

### 📦 Push the image to GitHub Container Registry

After the build finishes successfully:

```bash
tart tag sequoia-xcode-php:16.4 ghcr.io/simplepark-bv/macos-sequoia-xcode-php:16.4
tart push ghcr.io/simplepark-bv/macos-sequoia-xcode-php:16.4
```

To also tag it as the latest PHP build

```bash
tart push ghcr.io/simplepark-bv/macos-sequoia-xcode-php:16.4 ghcr.io/simplepark-bv/macos-sequoia-xcode-php:latest
```

---

### ✅ Verifying the Image

You can verify the image locally:

```bash
tart run ghcr.io/simplepark-bv/macos-sequoia-xcode-php:16.4
```

Then inside the VM:

```bash
php -v
composer -V
xcodebuild -version
flutter --version
```

---

## 🧾 License

This repository is based on the upstream [CirrusLabs macOS Image Templates](https://github.com/cirruslabs/macos-image-templates),  
licensed under the [MIT License](./LICENSE).

Modifications © 2025 SimplePark B.V.  
All changes and additions remain under the same MIT License.