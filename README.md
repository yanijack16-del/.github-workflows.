name: Build OpenPLi MultiboxPro

on:
  workflow_dispatch:

permissions:
  contents: read

jobs:
  build:
    runs-on: ubuntu-24.04
    timeout-minutes: 360
    steps:
      - name: Checkout workflow repository
        uses: actions/checkout@v4

      - name: Free runner space
        run: |
          sudo rm -rf /usr/local/lib/android /usr/share/dotnet /opt/ghc
          df -h

      - name: Install build prerequisites
        run: |
        sudo apt-get update
          sudo apt-get install -y \
            subversion \
            gettext \
            build-essential \
            chrpath \
            cpio \
            debianutils \
            diffstat \
            file \
            gawk \
            gcc \
            git \
            iputils-ping \
            libacl1 \
            liblz4-tool \
            locales \
            python3 \
            python3-git \
            python3-jinja2 \
            python3-pexpect \
            python3-pip \
            python3-subunit \
            socat \
            texinfo \
            unzip \
            wget \
            xz-utils \
            zstd \
            lz4
          command -v svn
          command -v msgfmt

      - name: Clone OpenPLi Scarthgap
        run: |
          git clone --depth 1 --branch scarthgap https://github.com/OpenPLi/openpli-oe-core.git openpli-oe-core

      - name: Initialize build environment
        working-directory: openpli-oe-core
        run: |
          make init
          make update

      - name: Build MultiboxPro image
        working-directory: openpli-oe-core
        env:
          MACHINE: multiboxpro
        run: |
          MACHINE=multiboxpro make image

      - name: List generated images
        if: always()
        run: |
          find openpli-oe-core/build/tmp/deploy/images -type f \( -name '*.zip' -o -name '*.tar.gz' -o -name '*.bin' -o -name '*.img' \) -print 2>/dev/null || true

      - name: Upload image artifacts
        if: success()
        uses: actions/upload-artifact@v4
        with:
          name: Novaler-MultiboxPro-OpenPLi
          path: openpli-oe-core/build/tmp/deploy/images/multiboxpro/
          if-no-files-found: error
          compression-level: 0
