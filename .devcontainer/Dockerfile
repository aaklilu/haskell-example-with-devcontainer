# Use the [Choice] comment to indicate option arguments that should appear in VS Code UX
#
# [Choice] Haskell version: 9, 8
ARG VARIANT=9
FROM haskell:${VARIANT}

# Use the [Option] comment to specify true/false arguments that should appear in VS Code UX
#
# [Option] Install zsh
ARG INSTALL_ZSH="true"
# [Option] Upgrade OS packages to their latest versions
ARG UPGRADE_PACKAGES="false"

# Install needed packages and setup non-root user. Use a separate RUN statement to add your own dependencies
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID
COPY library-scripts/*.sh /tmp/library-scripts/
RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && /bin/bash /tmp/library-scripts/common-debian.sh "${INSTALL_ZSH}" "${USERNAME}" "${USER_UID}" "${USER_GID}" "${UPGRADE_PACKAGES}" "true" "true"\
    && apt-get autoremove -y && apt-get clean -y && rm -rf /var/lib/apt/lists/* /tmp/library-scripts
    
# Install gpg keys
ARG GHCUP_GPG_KEY=7784930957807690A66EBDBE3786C5262ECB4A3F
RUN gpg --batch --keyserver keys.openpgp.org --recv-keys $GHCUP_GPG_KEY

# Install ghcup
RUN set -eux; \
    cd /tmp; \
    ARCH="$(dpkg-architecture --query DEB_BUILD_GNU_CPU)"; \
    case "$ARCH" in \
        'aarch64') \
            curl https://downloads.haskell.org/~ghcup/aarch64-linux-ghcup > /usr/bin/ghcup; \
            ;; \
        'x86_64') \
            curl https://downloads.haskell.org/~ghcup/x86_64-linux-ghcup > /usr/bin/ghcup; \
            ;; \
        *) echo >&2 "error: unsupported architecture '$ARCH'" ; exit 1 ;; \
    esac;

# GHC 9.0 requires LLVM version 9 - 12 on aarch64
ARG LLVM_VERSION=12
ARG LLVM_KEY=6084F3CF814B57C1CF12EFD515CF4D18AF4F7421

RUN set -eux; \
    if [ "$(dpkg-architecture --query DEB_BUILD_GNU_CPU)" = "aarch64" ]; then \
        GNUPGHOME="$(mktemp -d)"; export GNUPGHOME; \
        mkdir -p /usr/local/share/keyrings/; \
        gpg --batch --keyserver keyserver.ubuntu.com --recv-keys "$LLVM_KEY"; \
        gpg --batch --armor --export "$LLVM_KEY" > /usr/local/share/keyrings/apt.llvm.org.gpg.asc; \
        echo "deb [ signed-by=/usr/local/share/keyrings/apt.llvm.org.gpg.asc ] http://apt.llvm.org/buster/ llvm-toolchain-buster-$LLVM_VERSION main" > /etc/apt/sources.list.d/llvm.list; \
        apt-get update; \
        apt-get install -y --no-install-recommends llvm-$LLVM_VERSION; \
        gpgconf --kill all; \
        rm -rf "$GNUPGHOME" /var/lib/apt/lists/*; \
    fi

#Configure ghcup
RUN chmod +x /usr/bin/ghcup && \
    ghcup config set gpg-setting GPGStrict