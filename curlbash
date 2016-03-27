#!/bin/sh
#######################################################################
# Copyright (c) 2016 Scott Vokes <vokes.s@gmail.com>
# 
# Permission to use, copy, modify, and/or distribute this software for any
# purpose with or without fee is hereby granted, provided that the above
# copyright notice and this permission notice appear in all copies.
#
# THE SOFTWARE IS PROVIDED "AS IS" AND THE AUTHOR DISCLAIMS ALL WARRANTIES
# WITH REGARD TO THIS SOFTWARE INCLUDING ALL IMPLIED WARRANTIES OF
# MERCHANTABILITY AND FITNESS. IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR
# ANY SPECIAL, DIRECT, INDIRECT, OR CONSEQUENTIAL DAMAGES OR ANY DAMAGES
# WHATSOEVER RESULTING FROM LOSS OF USE, DATA OR PROFITS, WHETHER IN AN
# ACTION OF CONTRACT, NEGLIGENCE OR OTHER TORTIOUS ACTION, ARISING OUT OF
# OR IN CONNECTION WITH THE USE OR PERFORMANCE OF THIS SOFTWARE.
#######################################################################
# Curl a URL, allowing a chance to review/checksum it before executing.
#######################################################################

CURLBASH_VERSION="0.1.0"

# Defaults
VERBOSE=0
CONFIRM=1
USE_SUDO=0
EXPECTED_SHA256=0
FETCH_AND_EXIT=0
USE_CACHE=1
DELETE_AFTER_EXEC=1
PAGER=${PAGER:-less}
SUDO=${SUDO:-sudo}

usage() {
    echo "curlbash version ${CURLBASH_VERSION} by Scott Vokes"
    echo "Usage: curlbash [-c] [-d] [-e SHA256] [-f | -r] [-h] [-v] URL"
    echo "  -c           ignore cache"
    echo "  -d           don't delete cached downloads"
    echo "  -e SHA256    expect this SHA256, or exit"
    echo "  -f           fetch and exit"
    echo "  -h           print this help and exit"
    echo "  -r           run, without confirmation"
    #echo "  -s           run with sudo"
    echo "  -v           increase verbosity"
}

log() {
    if [ $VERBOSE -gt 0 ]; then
        echo "###" $*
    fi
}

fetch() {
    log "Saving $URL to $CACHE_FILE"
    curl --fail --silent --show-error --location "$1" -o "$2" || exit
    chmod u+x "$2"
}

run() {
    log "Running $1"
    if [ $USE_SUDO -gt 0 ]; then
        $SUDO "$1"
    else
        "$1"
    fi 
}

cleanup() {
    if [ $DELETE_AFTER_EXEC -gt 0 ]; then
        log "Deleting $1"
        rm "$1"
    else
        log "Skipping deletion"
    fi
}

parse_args() {
    args=`getopt cde:fhrsv $*`
    if [ $? != 0 ]; then
        usage
        exit 1
    fi

    set -- $args
    for i
    do
        case "$i"
        in
            -c)                 # ignore cache
                USE_CACHE=0
                shift;;
            -d)                 # don't delete download
                DELETE_AFTER_EXEC=0
                shift;;
            -e)                 # compare hash
                EXPECTED_SHA256="$2"
                shift
                shift;;
            -f)                 # fetch only
                FETCH_AND_EXIT=1
                shift;;
            -h)                 # print help
                usage
                exit 0;;
            -r)                 # just run
                CONFIRM=0
                shift;;
            -s)
                USE_SUDO=1
                shift;;
            -v)                 # verbose
                VERBOSE=1
                shift;;
            --)
                shift; break;;
        esac
    done
    URL="$1"
}

exists_in_path() {
    type $1 >/dev/null 2>/dev/null
}

# Try to find program to SHA256
if exists_in_path "sha256"; then
    SHA256="sha256"
elif exists_in_path "shasum"; then
    SHA256="shasum -p -a256 "
else
    echo "Error: Could not find a program for SHA256 checksums."
    echo "    Install shasum or sha256 and add it to your path."
fi

parse_args $*

if [ -z $URL ]; then
    usage
    exit 1
fi

HASH=$(echo -n $URL | ${SHA256} | awk '{print($1)}')
CACHE_FILE=${TMP:-/tmp}/$HASH

if [ -e $CACHE_FILE ] && [ $USE_CACHE = 1 ]; then
    log "Using existing cache at $CACHE_FILE"
else
    fetch "$URL" "$CACHE_FILE"
fi

if [ $FETCH_AND_EXIT = 1 ]; then
    exit 0
fi
   
if [ $EXPECTED_SHA256 != 0 ]; then
    log "Comparing checksums, expecting $EXPECTED_SHA256"
    CHECKSUM=$(${SHA256} "$CACHE_FILE" | awk '{print($1)}')
    log "Got checksum $CHECKSUM"
    if [ $EXPECTED_SHA256 != $CHECKSUM ]; then
        echo ""
        echo "### WARNING: Checksum mismatch!"
        echo "### EXP: $EXPECTED_SHA256"
        echo "### GOT: $CHECKSUM"
        exit 1
    fi
fi

if [ $CONFIRM = 1 ]; then
    $PAGER "${CACHE_FILE}"
    printf "Continue? [y/N] "
    read CONTINUE

    if [ ${CONTINUE:-n} = "y" ]; then
        run "${CACHE_FILE}"
        cleanup "${CACHE_FILE}"
    else
        log Exiting.
        exit 0
    fi
else
    run "${CACHE_FILE}" && cleanup "${CACHE_FILE}"
fi
