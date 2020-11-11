FROM github.com/netlify/toolchain-packages:main+repository
ARG DEB_BUILD_PROFILES=debug
ARG GITHUB_WORKSPACE=/root/upstream
ARG GITHUB_BASE_REF=7.1.x-netlify
ARG GITHUB_HEAD_REF=HEAD

ENV DEBFULLNAME github-actions[bot]
ENV DEBEMAIL 41898282+github-actions[bot]@users.noreply.github.com
ENV DPKG_COLORS always
ENV TZ UTC

RUN apt-get update
RUN apt-get install git-buildpackage build-essential devscripts debhelper equivs

GIT CLONE --branch ${GITHUB_BASE_REF} github.com/netlify/trafficserver upstream
COPY --dir debian upstream/

WORKDIR upstream

RUN git switch ${GITHUB_BASE_REF}

update-debian-changelog:
  ARG DEB_BUILD_PROFILES=debug
  RUN debian/scripts/update-debian-changelog \
    --distribution=pull-request \
    --version=$(date --utc +%Y.%m.%d) \
    --revision=$(date --date "1970-01-01 $(date +%T)" +%s) \
    --profile=${DEB_BUILD_PROFILES} \
    --package=netlify-trafficserver-dev \
    --package=netlify-trafficserver

build:
  FROM +update-debian-changelog
  ARG GITHUB_WORKSPACE=/root/upstream
  ARG DEB_BUILD_PROFILES=debug
  ARG DEB_BUILD_OPTIONS=-u${GITHUB_WORKSPACE}
  RUN mk-build-deps --build-dep --install --remove --tool \
    'apt-get --yes -o Debug::pkgProblemResolver=yes --no-install-recommends'
  RUN debuild \
    --preserve-envvar=NETLIFY_* \
    --preserve-envvar=GITHUB_* \
    --preserve-envvar=CCACHE_* \
    --preserve-envvar=DPKG_* \
    --prepend-path=/usr/lib/ccache \
    --source-option=--diff-ignore=debian/.* \
    --buildinfo-option=-u/root/upstream \
    --changes-option=-u/root/upstream \
    --build=binary \
    --no-sign --no-check-builddeps
  SAVE ARTIFACT *.ddeb
  SAVE ARTIFACT *.deb

packages:
  FROM ubuntu:20.04
  COPY +build/* /srv/deb/
  SAVE ARTIFACT /srv/deb/* AS LOCAL ./
