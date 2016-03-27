curlbash: locally save and checksum/review before curl|bash-ing installers

# curlbash

Installing things by curling them to a shell probably isn't a great
practice security-wise, but several things can make it a little better.
This script caches the curl'd installer, can optionally verify a
checksum, and give the user a chance to review what was downloaded
before executing it. Consider it harm reduction for the command line.

curlbash is distributed under the ISC license.


# Dependencies

curlbash uses SHA-256 for hashing. It will check for `sha256` or
`shasum`, and use whichever is in the path. It also requires `curl`.


# Usage

    Usage: curlbash [-c] [-e SHA256] [-f | -r] [-h] [-v] URL
      -c           ignore cache
      -e SHA256    expect this SHA256, or exit
      -f           fetch and exit
      -h           print this help and exit
      -r           run, without confirmation
      -v           increase verbosity


# Examples

    $ curlbash -h

Print help.

    $ curlbash -f URL

Fetch `URL` (if not cached), cache it locally, and exit. It will be
saved under `/tmp`, with a filename based on the hash of the URL.

    $ curlbash -r -e HASH URL

Download and execute `URL`, but only if the SHA-256 hash for the
downloaded file matches `HASH`.

    $ curlbash -c URL

Re-fetch `URL` (ignoring a locally cached copy), display what was
downloaded, and prompt for confirmation before executing.


# Portability

Despite the name, curlbash is targeted at `/bin/sh` for portability
reasons. A reasonable effort is made to test it with bash, mksh, pdksh,
dash, and other common shells that attempt to be POSIX-compliant /
compatible with the Bourne shell. (In other words, bash-isms are bugs.)
