aria2c(1)
=========

SYNOPSIS
--------
**aria2c** [<OPTIONS>] [<URI>|<MAGNET>|<TORRENT_FILE>|<METALINK_FILE>] ...

DESCRIPTION
-----------

aria2 is a utility for downloading files. The supported protocols are
HTTP(S), FTP, BitTorrent, and Metalink. aria2 can download a file from
multiple sources/protocols and tries to utilize your maximum download
bandwidth. It supports downloading a file from HTTP(S)/FTP and
BitTorrent at the same time, while the data downloaded from
HTTP(S)/FTP is uploaded to the BitTorrent swarm. Using Metalink's
chunk checksums, aria2 automatically validates chunks of data while
downloading a file like BitTorrent.

OPTIONS
-------

Basic Options
~~~~~~~~~~~~~
.. option:: -d, --dir=<DIR>

  The directory to store the downloaded file.

.. option:: -i, --input-file=<FILE>

  Downloads URIs found in FILE. You can specify multiple URIs for a single
  entity: separate URIs on a single line using the TAB character.
  Reads input from stdin when ``-`` is specified.
  Additionally, options can be specified after each line of
  URI. This optional line must start with one or more white spaces and have
  one option per single line.
  See `Input File`_ subsection for details.
  See also :option:`--deferred-input` option.

.. option:: -l, --log=<LOG>

  The file name of the log file. If ``-`` is specified, log is written to
  stdout. If empty string("") is specified, log is not written to file.

.. option:: -j, --max-concurrent-downloads=<N>

  Set maximum number of parallel downloads for every static (HTTP/FTP) URI,
  torrent and metalink. See also :option:`--split <-s>` option.
  Default: ``5``

.. option:: -V, --check-integrity[=true|false]

  Check file integrity by validating piece hashes or a hash of entire
  file.  This option has effect only in BitTorrent, Metalink downloads
  with checksums or HTTP(S)/FTP downloads with
  :option:`--checksum` option.  If
  piece hashes are provided, this option can detect damaged portions
  of a file and re-download them.  If a hash of entire file is
  provided, hash check is only done when file has been already
  download. This is determined by file length. If hash check fails,
  file is re-downloaded from scratch.  If both piece hashes and a hash
  of entire file are provided, only piece hashes are used. Default:
  ``false``

.. option:: -c, --continue[=true|false]

   Continue downloading a partially downloaded file.
   Use this option to resume a download started by a web browser or another
   program which downloads files sequentially from the beginning.
   Currently this option is only applicable to HTTP(S)/FTP downloads.

.. option:: -h, --help[=<TAG>|<KEYWORD>]

   The help messages are classified with tags. A tag starts with
   ``#``. For example, type ``--help=#http`` to get the usage for the
   options tagged with ``#http``. If non-tag word is given, print the
   usage for the options whose name includes that word.  Available
   Values: ``#basic``, ``#advanced``, ``#http``, ``#https``, ``#ftp``,
   ``#metalink``, ``#bittorrent``, ``#cookie``, ``#hook``, ``#file``, ``#rpc``,
   ``#checksum``, ``#experimental``, ``#deprecated``, ``#help``, ``#all``
   Default: ``#basic``

HTTP/FTP Options
~~~~~~~~~~~~~~~~
.. option:: --all-proxy=<PROXY>

  Use this proxy server for all protocols.  To erase previously
  defined proxy, use "".  You can override this setting and specify a
  proxy server for a particular protocol using :option:`--http-proxy`,
  :option:`--https-proxy` and :option:`--ftp-proxy` options.  This affects all URIs.
  The format of PROXY is ``[http://][USER:PASSWORD@]HOST[:PORT]``.
  See also `ENVIRONMENT`_ section.

  .. note::
    
    If user and password are embedded in proxy URI and they are also
    specified by *--{http,https,ftp,all}-proxy-{user,passwd}* options,
    those appeared later have precedence. For example, you have
    ``http-proxy-user=myname``, ``http-proxy-passwd=mypass`` in aria2.conf and
    you specify ``--http-proxy="http://proxy"`` in command-line, then you get
    HTTP proxy ``http://proxy`` with user ``myname`` and password
    ``mypass``.

    Another example: if you specified in command-line
    ``--http-proxy="http://user:pass@proxy" --http-proxy-user="myname"
    --http-proxy-passwd="mypass"``, then you will get HTTP proxy
    ``http://proxy`` with user ``myname`` and password ``mypass``.

    One more example: if you specified in command-line ``--http-proxy-user="myname"
    --http-proxy-passwd="mypass" --http-proxy="http://user:pass@proxy"``,
    then you get HTTP proxy ``http://proxy`` with user ``user`` and password
    ``pass``.

.. option:: --all-proxy-passwd=<PASSWD>

  Set password for :option:`--all-proxy` option.

.. option:: --all-proxy-user=<USER>

  Set user for :option:`--all-proxy` option.

.. option:: --checksum=<TYPE>=<DIGEST>

  Set checksum. TYPE is hash type. The supported hash type is listed
  in ``Hash Algorithms`` in ``aria2c -v``. DIGEST is hex digest.  For
  example, setting sha-1 digest looks like this:
  ``sha-1=0192ba11326fe2298c8cb4de616f4d4140213838`` This option applies
  only to HTTP(S)/FTP downloads.

.. option:: --connect-timeout=<SEC>

  Set the connect timeout in seconds to establish connection to
  HTTP/FTP/proxy server. After the connection is established, this
  option makes no effect and :option:`--timeout <-t>` option is used instead.
  Default: ``60``

.. option:: --dry-run[=true|false]

  If ``true`` is given, aria2 just checks whether the remote file is
  available and doesn't download data. This option has effect on
  HTTP/FTP download.  BitTorrent downloads are canceled if ``true`` is
  specified.  Default: ``false``

.. option:: --lowest-speed-limit=<SPEED>

  Close connection if download speed is lower than or equal to this
  value(bytes per sec).
  ``0`` means aria2 does not have a lowest speed limit.
  You can append ``K`` or ``M`` (1K = 1024, 1M = 1024K).
  This option does not affect BitTorrent downloads.
  Default: ``0``

.. option:: -x, --max-connection-per-server=<NUM>

  The maximum number of connections to one server for each download.
  Default: ``1``

.. option:: --max-file-not-found=<NUM>

  If aria2 receives "file not found" status from the remote HTTP/FTP
  servers NUM times without getting a single byte, then force the
  download to fail. Specify ``0`` to disable this option. This options is
  effective only when using HTTP/FTP servers.
  Default: ``0``

.. option:: -m, --max-tries=<N>

  Set number of tries. ``0`` means unlimited.
  See also :option:`--retry-wait`.
  Default: ``5``

.. option:: -k, --min-split-size=<SIZE>

  aria2 does not split less than 2*SIZE byte range.  For example,
  let's consider downloading 20MiB file. If SIZE is 10M, aria2 can
  split file into 2 range [0-10MiB) and [10MiB-20MiB) and download it
  using 2 sources(if :option:`--split <-s>` >= 2, of course).  If SIZE is 15M,
  since 2*15M > 20MiB, aria2 does not split file and download it using
  1 source.  You can append ``K`` or ``M`` (1K = 1024, 1M = 1024K).
  Possible Values: ``1M`` -``1024M`` Default: ``20M``

.. option:: -n, --no-netrc[=true|false]

  Disables netrc support. netrc support is enabled by default.

  .. note::
    
    netrc file is only read at the startup if
    :option:`--no-netrc <-n>` is ``false``.
    So if :option:`--no-netrc <-n>` is ``true`` at the startup,
    no netrc is available throughout the session.
    You cannot get netrc enabled even if you send
    :option:`--no-netrc=false <-n>` using
    :func:`aria2.changeGlobalOption`.

.. option:: --no-proxy=<DOMAINS>

  Specify comma separated hostnames, domains and network address with
  or without CIDR block where proxy should not be used.

  .. note::

    For network address with CIDR block, both IPv4 and IPv6 addresses work. Current
    implementation does not resolve hostname in URI to compare network
    address specified in :option:`--no-proxy`. So it is only effecive if URI has
    numeric IP addresses.

.. option:: -o, --out=<FILE>

  The file name of the downloaded file. When :option:`--force-sequential <-Z>` option is used, this
  option is ignored.

  .. note::

    In Metalink or BitTorrent download you cannot specify file name.
    The file name specified here is only used when the URIs fed to aria2
    are done by command line without :option:`--input-file <-i>`, :option:`--force-sequential <-Z>` option. For example:

    .. code-block:: console

      $ aria2c -o myfile.zip "http://mirror1/file.zip" "http://mirror2/file.zip"

.. option:: --proxy-method=<METHOD>

  Set the method to use in proxy request.  METHOD is either ``get`` or
  ``tunnel``. HTTPS downloads always use ``tunnel`` regardless of this
  option.
  Default: ``get``

.. option:: -R, --remote-time[=true|false]

  Retrieve timestamp of the remote file from the remote HTTP/FTP
  server and if it is available, apply it to the local file.
  Default: ``false``

.. option:: --reuse-uri[=true|false]

  Reuse already used URIs if no unused URIs are left.
  Default: ``true``

.. option:: --retry-wait=<SEC>

  Set the seconds to wait between retries. With SEC > 0, aria2 will
  retry download when the HTTP server returns 503 response. Default:
  ``0``

.. option:: --server-stat-of=<FILE>

  Specify the filename to which performance profile of the servers is
  saved. You can load saved data using :option:`--server-stat-if` option. See
  `Server Performance Profile`_
  subsection below for file format.

.. option:: --server-stat-if=<FILE>

  Specify the filename to load performance profile of the servers. The
  loaded data will be used in some URI selector such as ``feedback``.
  See also :option:`--uri-selector` option. See
  `Server Performance Profile`_
  subsection below for file format.

.. option:: --server-stat-timeout=<SEC>

  Specifies timeout in seconds to invalidate performance profile of
  the servers since the last contact to them.
  Default: ``86400`` (24hours)

.. option:: -s, --split=<N>

  Download a file using N connections.  If more than N URIs are given,
  first N URIs are used and remaining URIs are used for backup.  If
  less than N URIs are given, those URIs are used more than once so
  that N connections total are made simultaneously.  The number of
  connections to the same host is restricted by
  :option:`--max-connection-per-server <-x>` option.
  See also :option:`--min-split-size <-k>` option.
  Default: ``5``

  .. note::
    
    Some Metalinks regulate the number of servers to connect.  aria2
    strictly respects them.  This means that if Metalink defines the
    maxconnections attribute lower than N, then aria2 uses the
    value of maxconnections attribute instead of N.

.. option:: --stream-piece-selector=<SELECTOR>

  Specify piece selection algorithm used in HTTP/FTP download. Piece
  means fixed length segment which is downloaded in parallel in
  segmented download. If ``default`` is given, aria2 selects piece so
  that it reduces the number of establishing connection. This is
  reasonable default behaviour because establishing connection is an
  expensive operation.  If ``inorder`` is given, aria2 selects piece
  which has minimum index. Index=0 means first of the file. This will
  be useful to view movie while downloading it.
  :option:`--enable-http-pipelining` option may
  be useful to reduce reconnection overhead.  Please note that aria2
  honors
  :option:`--min-split-size <-k>` option,
  so it will be necessary to specify a reasonable value to
  :option:`--min-split-size <-k>` option.
  If ``geom`` is given, at the beginning aria2 selects piece which has
  minimum index like ``inorder``, but it exponentially increasingly
  keeps space from previously selected piece. This will reduce the
  number of establishing connection and at the same time it will
  download the beginning part of the file first. This will be useful
  to view movie while downloading it.
  Default: ``default``

.. option:: -t, --timeout=<SEC>

  Set timeout in seconds.
  Default: ``60``

.. option:: --uri-selector=<SELECTOR>

  Specify URI selection algorithm. The possible values are ``inorder``,
  ``feedback`` and ``adaptive``.  If ``inorder`` is given, URI is tried in
  the order appeared in the URI list.  If ``feedback`` is given, aria2
  uses download speed observed in the previous downloads and choose
  fastest server in the URI list. This also effectively skips dead
  mirrors. The observed download speed is a part of performance
  profile of servers mentioned in :option:`--server-stat-of` and
  :option:`--server-stat-if` options.  If ``adaptive`` is given, selects one of
  the best mirrors for the first and reserved connections.  For
  supplementary ones, it returns mirrors which has not been tested
  yet, and if each of them has already been tested, returns mirrors
  which has to be tested again. Otherwise, it doesn't select anymore
  mirrors. Like ``feedback``, it uses a performance profile of servers.
  Default: ``feedback``

HTTP Specific Options
~~~~~~~~~~~~~~~~~~~~~
.. option:: --ca-certificate=<FILE>

  Use the certificate authorities in FILE to verify the peers.
  The certificate file must be in PEM format and can contain multiple CA
  certificates.
  Use :option:`--check-certificate` option to enable verification.

.. option:: --certificate=<FILE>

  Use the client certificate in FILE.
  The certificate must be in PEM format.
  You may use :option:`--private-key` option to specify the private key.

.. option:: --check-certificate[=true|false]

  Verify the peer using certificates specified in :option:`--ca-certificate` option.
  Default: ``true``

.. option:: --http-accept-gzip[=true|false]

  Send ``Accept: deflate, gzip`` request header and inflate response if
  remote server responds with ``Content-Encoding: gzip`` or
  ``Content-Encoding: deflate``.  Default: ``false``

  .. note::
    
    Some server responds with ``Content-Encoding: gzip`` for files which
    itself is gzipped file. aria2 inflates them anyway because of the
    response header.

.. option:: --http-auth-challenge[=true|false]

  Send HTTP authorization header only when it is requested by the
  server. If ``false`` is set, then authorization header is always sent
  to the server.  There is an exception: if username and password are
  embedded in URI, authorization header is always sent to the server
  regardless of this option.  Default: ``false``

.. option:: --http-no-cache[=true|false]

   Send Cache-Control: no-cache and Pragma: no-cache header to avoid
   cached content.  If ``false`` is given, these headers are not sent
   and you can add Cache-Control header with a directive you like
   using :option:`--header` option. Default: ``true``

.. option:: --http-user=<USER>

  Set HTTP user. This affects all URIs.

.. option:: --http-passwd=<PASSWD>

  Set HTTP password. This affects all URIs.

.. option:: --http-proxy=<PROXY>

  Use this proxy server for HTTP.  To erase previously defined proxy,
  use "".  See also :option:`--all-proxy` option.  This affects all URIs.  The
  format of PROXY is ``[http://][USER:PASSWORD@]HOST[:PORT]``

.. option:: --http-proxy-passwd=<PASSWD>

  Set password for :option:`--http-proxy` option.

.. option:: --http-proxy-user=<USER>

  Set user for :option:`--http-proxy` option.

.. option:: --https-proxy=<PROXY>

  Use this proxy server for HTTPS. To erase previously defined proxy,
  use "". See also :option:`--all-proxy` option.  This affects all URIs.  The
  format of PROXY is ``[http://][USER:PASSWORD@]HOST[:PORT]``

.. option:: --https-proxy-passwd=<PASSWD>

  Set password for :option:`--https-proxy` option.

.. option:: --https-proxy-user=<USER>

  Set user for :option:`--https-proxy` option.

.. option:: --private-key=<FILE>

  Use the private key in FILE.
  The private key must be decrypted and in PEM format.
  The behavior when encrypted one is given is undefined.
  See also :option:`--certificate` option.

.. option:: --referer=<REFERER>

  Set Referer. This affects all URIs.

.. option:: --enable-http-keep-alive[=true|false]

  Enable HTTP/1.1 persistent connection.
  Default: ``true``

.. option:: --enable-http-pipelining[=true|false]

  Enable HTTP/1.1 pipelining.
  Default: ``false``

  .. note::
    
    In performance perspective, there is usually no advantage to enable
    this option.

.. option:: --header=<HEADER>

  Append HEADER to HTTP request header.
  You can use this option repeatedly to specify more than one header:

  .. code-block:: console

        $ aria2c --header="X-A: b78" --header="X-B: 9J1" "http://host/file"

.. option:: --load-cookies=<FILE>

  Load Cookies from FILE using the Firefox3 format (SQLite3),
  Chromium/Google Chrome (SQLite3) and the
  Mozilla/Firefox(1.x/2.x)/Netscape format.

  .. note::

    If aria2 is built without libsqlite3, then it doesn't support Firefox3
    and Chromium/Google Chrome cookie format.

.. option:: --save-cookies=<FILE>

  Save Cookies to FILE in Mozilla/Firefox(1.x/2.x)/ Netscape
  format. If FILE already exists, it is overwritten. Session Cookies
  are also saved and their expiry values are treated as 0.  Possible
  Values: ``/path/to/file``

.. option:: --use-head[=true|false]

  Use HEAD method for the first request to the HTTP server.
  Default: ``false``


.. option:: -U, --user-agent=<USER_AGENT>

  Set user agent for HTTP(S) downloads.
  Default: ``aria2/$VERSION``, $VERSION is replaced by package version.

FTP Specific Options
~~~~~~~~~~~~~~~~~~~~
.. option:: --ftp-user=<USER>

  Set FTP user. This affects all URIs.
  Default: ``anonymous``

.. option:: --ftp-passwd=<PASSWD>

  Set FTP password. This affects all URIs.
  If user name is embedded but password is missing in URI, aria2 tries
  to resolve password using .netrc. If password is found in .netrc,
  then use it as password. If not, use the password specified in this
  option.
  Default: ``ARIA2USER@``

.. option:: -p, --ftp-pasv[=true|false]

  Use the passive mode in FTP.
  If ``false`` is given, the active mode will be used.
  Default: ``true``

.. option:: --ftp-proxy=<PROXY>

  Use this proxy server for FTP.  To erase previously defined proxy,
  use "".  See also :option:`--all-proxy` option.  This affects all URIs.  The
  format of PROXY is ``[http://][USER:PASSWORD@]HOST[:PORT]``

.. option:: --ftp-proxy-passwd=<PASSWD>

  Set password for :option:`--ftp-proxy` option.

.. option:: --ftp-proxy-user=<USER>

  Set user for :option:`--ftp-proxy` option.

.. option:: --ftp-type=<TYPE>

  Set FTP transfer type. TYPE is either ``binary`` or ``ascii``.
  Default: ``binary``

.. option:: --ftp-reuse-connection[=true|false]

  Reuse connection in FTP.
  Default: ``true``

BitTorrent/Metalink Options
~~~~~~~~~~~~~~~~~~~~~~~~~~~
.. option:: --select-file=<INDEX>...

  Set file to download by specifying its index.
  You can find the file index using the :option:`--show-files <-S>` option.
  Multiple indexes can be specified by using ``,``, for example: ``3,6``.
  You can also use ``-`` to specify a range: ``1-5``.
  ``,`` and ``-`` can be used together: ``1-5,8,9``.
  When used with the -M option, index may vary depending on the query
  (see *--metalink-\** options).

  .. note::

    In multi file torrent, the adjacent files specified by this option may
    also be downloaded. This is by design, not a bug.
    A single piece may include several files or part of files, and aria2
    writes the piece to the appropriate files.

.. option:: -S, --show-files[=true|false]

  Print file listing of .torrent, .meta4 and .metalink file and exit.
  In case of .torrent file, additional information
  (infohash, piece length, etc) is also printed.

BitTorrent Specific Options
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. option:: --bt-enable-lpd[=true|false]

  Enable Local Peer Discovery.  If a private flag is set in a torrent,
  aria2 doesn't use this feature for that download even if ``true`` is
  given.  Default: ``false``

.. option:: --bt-exclude-tracker=<URI>[,...]

  Comma separated list of BitTorrent tracker's announce URI to
  remove. You can use special value ``*`` which matches all URIs, thus
  removes all announce URIs. When specifying ``*`` in shell
  command-line, don't forget to escape or quote it.  See also
  :option:`--bt-tracker` option.

.. option:: --bt-external-ip=<IPADDRESS>

  Specify the external IP address to report to a BitTorrent
  tracker. Although this function is named ``external``, it can accept
  any kind of IP addresses. IPADDRESS must be a numeric IP address.

.. option:: --bt-hash-check-seed[=true|false]

 If ``true`` is given, after hash check using :option:`--check-integrity <-V>` option and
 file is complete, continue to seed file. If you want to check file
 and download it only when it is damaged or incomplete, set this
 option to ``false``.  This option has effect only on BitTorrent download.
 Default: ``true``

.. option:: --bt-lpd-interface=<INTERFACE>

  Use given interface for Local Peer Discovery. If this option is not
  specified, the default interface is chosen. You can specify
  interface name and IP address.  Possible Values: interface, IP
  addres

.. option:: --bt-max-open-files=<NUM>

  Specify maximum number of files to open in each BitTorrent download.
  Default: ``100``

.. option:: --bt-max-peers=<NUM>

  Specify the maximum number of peers per torrent.  ``0`` means
  unlimited.  See also :option:`--bt-request-peer-speed-limit` option.
  Default: ``55``

.. option:: --bt-metadata-only[=true|false]

  Download metadata only. The file(s) described in metadata will not
  be downloaded. This option has effect only when BitTorrent Magnet
  URI is used. See also :option:`--bt-save-metadata` option.  Default: ``false``

.. option:: --bt-min-crypto-level=plain|arc4

  Set minimum level of encryption method.
  If several encryption methods are provided by a peer, aria2 chooses the lowest
  one which satisfies the given level.
  Default: ``plain``

.. option:: --bt-prioritize-piece=head[=<SIZE>],tail[=<SIZE>]

  Try to download first and last pieces of each file first. This is
  useful for previewing files. The argument can contain 2 keywords:
  ``head`` and ``tail``. To include both keywords, they must be separated
  by comma. These keywords can take one parameter, SIZE. For example,
  if ``head=<SIZE>`` is specified, pieces in the range of first SIZE bytes
  of each file get higher priority.  ``tail=<SIZE>`` means the range of
  last SIZE bytes of each file. SIZE can include ``K`` or ``M`` (1K = 1024,
  1M = 1024K). If SIZE is omitted, SIZE=1M is used.

.. option:: --bt-remove-unselected-file[=true|false]

   Removes the unselected files when download is completed in
   BitTorrent. To selecting files, use
   :option:`--select-file` option. If it is
   not used, all files are assumed to be selected. Please use this
   option with care because it will actually remove files from your
   disk.
   Default: ``false``

.. option:: --bt-require-crypto[=true|false]

  If true is given, aria2 doesn't accept and establish connection with legacy
  BitTorrent handshake(\19BitTorrent protocol).
  Thus aria2 always uses Obfuscation handshake.
  Default: ``false`` 

.. option:: --bt-request-peer-speed-limit=<SPEED>

  If the whole download speed of every torrent is lower than SPEED,
  aria2 temporarily increases the number of peers to try for more
  download speed. Configuring this option with your preferred download
  speed can increase your download speed in some cases.
  You can append ``K`` or ``M`` (1K = 1024, 1M = 1024K).
  Default: ``50K``

.. option:: --bt-save-metadata[=true|false]

  Save metadata as .torrent file. This option has effect only when
  BitTorrent Magnet URI is used.  The filename is hex encoded info
  hash with suffix .torrent. The directory to be saved is the same
  directory where download file is saved. If the same file already
  exists, metadata is not saved. See also :option:`--bt-metadata-only`
  option. Default: ``false``

.. option:: --bt-seed-unverified[=true|false]

  Seed previously downloaded files without verifying piece hashes.
  Default: ``false``

.. option:: --bt-stop-timeout=<SEC>

  Stop BitTorrent download if download speed is 0 in consecutive SEC
  seconds. If ``0`` is given, this feature is disabled.  Default: ``0``

.. option:: --bt-tracker=<URI>[,...]

  Comma separated list of additional BitTorrent tracker's announce
  URI. These URIs are not affected by :option:`--bt-exclude-tracker` option
  because they are added after URIs in :option:`--bt-exclude-tracker` option are
  removed.

.. option:: --bt-tracker-connect-timeout=<SEC>

  Set the connect timeout in seconds to establish connection to
  tracker. After the connection is established, this option makes no
  effect and :option:`--bt-tracker-timeout` option is used instead.  Default:
  ``60``

.. option:: --bt-tracker-interval=<SEC>

  Set the interval in seconds between tracker requests. This
  completely overrides interval value and aria2 just uses this value
  and ignores the min interval and interval value in the response of
  tracker. If ``0`` is set, aria2 determines interval based on the
  response of tracker and the download progress.  Default: ``0``

.. option:: --bt-tracker-timeout=<SEC>

  Set timeout in seconds. Default: ``60``

.. option:: --dht-entry-point=<HOST>:<PORT>

  Set host and port as an entry point to IPv4 DHT network.

.. option:: --dht-entry-point6=<HOST>:<PORT>

  Set host and port as an entry point to IPv6 DHT network.

.. option:: --dht-file-path=<PATH>

  Change the IPv4 DHT routing table file to PATH.
  Default: ``$HOME/.aria2/dht.dat``

.. option:: --dht-file-path6=<PATH>

  Change the IPv6 DHT routing table file to PATH.
  Default: ``$HOME/.aria2/dht6.dat``

.. option:: --dht-listen-addr6=<ADDR>

  Specify address to bind socket for IPv6 DHT.  It should be a global
  unicast IPv6 address of the host.

.. option:: --dht-listen-port=<PORT>...

  Set UDP listening port for both IPv4 and IPv6 DHT.
  Multiple ports can be specified by using ``,``, for example: ``6881,6885``.
  You can also use ``-`` to specify a range: ``6881-6999``. ``,`` and ``-`` can be used
  together.
  Default: ``6881-6999``

  .. note::

    Make sure that the specified ports are open for incoming UDP traffic.

.. option:: --dht-message-timeout=<SEC>

  Set timeout in seconds. Default: ``10``

.. option:: --enable-dht[=true|false]

  Enable IPv4 DHT functionality. If a private flag is set in a
  torrent, aria2 doesn't use DHT for that download even if ``true`` is
  given.  Default: ``true``

.. option:: --enable-dht6[=true|false]

   Enable IPv6 DHT functionality. If a private flag is set in a
   torrent, aria2 doesn't use DHT for that download even if ``true`` is
   given. Use :option:`--dht-listen-port` option to specify port number to
   listen on. See also :option:`--dht-listen-addr6` option.

.. option:: --enable-peer-exchange[=true|false]

  Enable Peer Exchange extension. If a private flag is set in a torrent, this
  feature is disabled for that download even if ``true`` is given.
  Default: ``true``

.. option:: --follow-torrent=true|false|mem

  If ``true`` or ``mem`` is specified, when a file whose suffix is ``.torrent`` or content
  type is ``application/x-bittorrent`` is downloaded, aria2 parses it as a torrent
  file and downloads files mentioned in it.
  If ``mem`` is specified, a torrent file is not written to the disk, but is just
  kept in memory.
  If ``false`` is specified, the action mentioned above is not taken.
  Default: ``true``

.. option:: -O, --index-out=<INDEX>=<PATH>

  Set file path for file with index=INDEX. You can find the file index
  using the :option:`--show-files <-S>` option.  PATH is a relative path to the
  path specified in :option:`--dir <-d>` option. You can use this option multiple
  times. Using this option, you can specify the output filenames of
  BitTorrent downloads.

.. option:: --listen-port=<PORT>...

  Set TCP port number for BitTorrent downloads.
  Multiple ports can be specified by using ``,``,  for example: ``6881,6885``.
  You can also use ``-`` to specify a range: ``6881-6999``.
  ``,`` and ``-`` can be used together: ``6881-6889,6999``.
  Default: ``6881-6999``

  .. note::

    Make sure that the specified ports are open for incoming TCP traffic.

.. option:: --max-overall-upload-limit=<SPEED>

  Set max overall upload speed in bytes/sec.
  ``0`` means unrestricted.
  You can append ``K`` or ``M`` (1K = 1024, 1M = 1024K).
  To limit the upload speed per torrent, use :option:`--max-upload-limit <-u>` option.
  Default: ``0``

.. option:: -u, --max-upload-limit=<SPEED>

  Set max upload speed per each torrent in bytes/sec.
  ``0`` means unrestricted.
  You can append ``K`` or ``M`` (1K = 1024, 1M = 1024K).
  To limit the overall upload speed, use :option:`--max-overall-upload-limit` option.
  Default: ``0``

.. option:: --peer-id-prefix=<PEER_ID_PREFIX>

  Specify the prefix of peer ID. The peer ID in
  BitTorrent is 20 byte length. If more than 20
  bytes are specified, only first 20 bytes are
  used. If less than 20 bytes are specified, random
  byte data are added to make its length 20 bytes.
  Default: ``aria2/$VERSION-``, $VERSION is replaced by package version.

.. option:: --seed-ratio=<RATIO>

  Specify share ratio. Seed completed torrents until share ratio reaches
  RATIO.
  You are strongly encouraged to specify equals or more than ``1.0`` here.
  Specify ``0.0`` if you intend to do seeding regardless of share ratio.
  If :option:`--seed-time` option is specified along with this option, seeding ends when
  at least one of the conditions is satisfied.
  Default: ``1.0``

.. option:: --seed-time=<MINUTES>

  Specify seeding time in minutes. Also see the :option:`--seed-ratio` option.

  .. note::
    
    Specifying :option:`--seed-time=0 <--seed-time>` disables seeding after download completed.

.. option:: -T, --torrent-file=<TORRENT_FILE>

  The path to the .torrent file.  You are not required to use this
  option because you can specify .torrent files without :option:`--torrent-file <-T>`.

Metalink Specific Options
~~~~~~~~~~~~~~~~~~~~~~~~~
.. option:: --follow-metalink=true|false|mem

  If ``true`` or ``mem`` is specified, when a file whose suffix is ``.meta4`` or ``.metalink`` or content
  type of ``application/metalink4+xml`` or ``application/metalink+xml`` is downloaded, aria2 parses it as a metalink
  file and downloads files mentioned in it.
  If ``mem`` is specified, a metalink file is not written to the disk, but is just
  kept in memory.
  If ``false`` is specified, the action mentioned above is not taken.
  Default: ``true``

.. option:: --metalink-base-uri=<URI>

  Specify base URI to resolve relative URI in metalink:url and
  metalink:metaurl element in a metalink file stored in local disk. If
  URI points to a directory, URI must end with ``/``.

.. option:: -M, --metalink-file=<METALINK_FILE>

  The file path to .meta4 and .metalink file. Reads input from stdin when ``-`` is
  specified.  You are not required to use this option because you can
  specify .metalink files without :option:`--metalink-file <-M>`.

.. option:: --metalink-language=<LANGUAGE>

  The language of the file to download.

.. option:: --metalink-location=<LOCATION>[,...]

  The location of the preferred server.
  A comma-delimited list of locations is acceptable, for example, ``jp,us``.

.. option:: --metalink-os=<OS>

  The operating system of the file to download.

.. option:: --metalink-version=<VERSION>

  The version of the file to download.

.. option:: --metalink-preferred-protocol=<PROTO>

  Specify preferred protocol.
  The possible values are ``http``, ``https``, ``ftp`` and ``none``.
  Specify ``none`` to disable this feature.
  Default: ``none``
 
.. option:: --metalink-enable-unique-protocol[=true|false]

  If ``true`` is given and several protocols are available for a mirror in a
  metalink file, aria2 uses one of them.
  Use :option:`--metalink-preferred-protocol` option to specify the preference of
  protocol.
  Default: ``true``

RPC Options
~~~~~~~~~~~

.. option:: --enable-rpc[=true|false]

  Enable JSON-RPC/XML-RPC server.  It is strongly recommended to set username
  and password using :option:`--rpc-user` and :option:`--rpc-passwd`
  option. See also :option:`--rpc-listen-port` option.  Default: ``false``

.. option:: --pause[=true|false]

  Pause download after added. This option is effective only when
  :option:`--enable-rpc=true <--enable-rpc>` is given.
  Default: ``false``


.. option:: --rpc-allow-origin-all[=true|false]

  Add Access-Control-Allow-Origin header field with value ``*`` to the
  RPC response.
  Default: ``false``

.. option:: --rpc-listen-all[=true|false]

  Listen incoming JSON-RPC/XML-RPC requests on all network interfaces. If false
  is given, listen only on local loopback interface.  Default: ``false``

.. option:: --rpc-listen-port=<PORT>

  Specify a port number for JSON-RPC/XML-RPC server to listen to.  Possible
  Values: ``1024`` -``65535`` Default: ``6800``

.. option:: --rpc-max-request-size=<SIZE>

  Set max size of JSON-RPC/XML-RPC request. If aria2 detects the request is
  more than SIZE bytes, it drops connection. Default: ``2M``

.. option:: --rpc-passwd=<PASSWD>

  Set JSON-RPC/XML-RPC password.

.. option:: --rpc-user=<USER>

  Set JSON-RPC/XML-RPC user.

Advanced Options
~~~~~~~~~~~~~~~~
.. option:: --allow-overwrite[=true|false]

  Restart download from scratch if the corresponding control file
  doesn't exist.  See also :option:`--auto-file-renaming` option.  Default:
  ``false``

.. option:: --allow-piece-length-change[=true|false]

  If false is given, aria2 aborts download when a piece length is different
  from one in a control file.
  If true is given, you can proceed but some download progress will be lost.
  Default: ``false``

.. option:: --always-resume[=true|false]

  Always resume download. If ``true`` is given, aria2 always tries to
  resume download and if resume is not possible, aborts download.  If
  ``false`` is given, when all given URIs do not support resume or aria2
  encounters ``N`` URIs which does not support resume (``N`` is the value
  specified using :option:`--max-resume-failure-tries` option), aria2
  downloads file from scratch.  See :option:`--max-resume-failure-tries`
  option. Default: ``true``

.. option:: --async-dns[=true|false]

  Enable asynchronous DNS.
  Default: ``true``

.. option:: --async-dns-server=<IPADDRESS>[,...]

  Comma separated list of DNS server address used in asynchronous DNS
  resolver. Usually asynchronous DNS resolver reads DNS server
  addresses from ``/etc/resolv.conf``. When this option is used, it uses
  DNS servers specified in this option instead of ones in
  ``/etc/resolv.conf``. You can specify both IPv4 and IPv6 address. This
  option is useful when the system does not have ``/etc/resolv.conf`` and
  user does not have the permission to create it.

.. option:: --auto-file-renaming[=true|false]

  Rename file name if the same file already exists.
  This option works only in HTTP(S)/FTP download.
  The new file name has a dot and a number(1..9999) appended.
  Default: ``true``

.. option:: --auto-save-interval=<SEC>

  Save a control file(\*.aria2) every SEC seconds.
  If ``0`` is given, a control file is not saved during download. aria2 saves a
  control file when it stops regardless of the value.
  The possible values are between ``0`` to ``600``.
  Default: ``60``

.. option:: --conditional-get[=true|false]

  Download file only when the local file is older than remote
  file. This function only works with HTTP(S) downloads only.  It does
  not work if file size is specified in Metalink. It also ignores
  Content-Disposition header.  If a control file exists, this option
  will be ignored.  This function uses If-Modified-Since header to get
  only newer file conditionally. When getting modification time of
  local file, it uses user supplied filename(see :option:`--out <-o>` option) or
  filename part in URI if :option:`--out <-o>` is not specified.
  To overwrite existing file, :option:`--allow-overwrite` is required.
  Default: ``false``

.. option:: --conf-path=<PATH>

  Change the configuration file path to PATH.
  Default: ``$HOME/.aria2/aria2.conf``

.. option:: -D, --daemon[=true|false]

  Run as daemon. The current working directory will be changed to ``/``
  and standard input, standard output and standard error will be
  redirected to ``/dev/null``. Default: ``false``

.. option:: --deferred-input[=true|false]

  If ``true`` is given, aria2 does not read all URIs and options from file
  specified by :option:`--input-file <-i>` option at startup,
  but it reads one by one when it
  needs later. This may reduce memory usage if input file contains a
  lot of URIs to download.  If ``false`` is given, aria2 reads all URIs
  and options at startup.
  Default: ``false``

.. option:: --disable-ipv6[=true|false]

  Disable IPv6. This is useful if you have to use broken DNS and want
  to avoid terribly slow AAAA record lookup. Default: ``false``

.. option:: --download-result=<OPT>

  This option changes the way ``Download Results`` is formatted. If OPT
  is ``default``, print GID, status, average download speed and
  path/URI. If multiple files are involved, path/URI of first
  requested file is printed and remaining ones are omitted.  If OPT is
  ``full``, print GID, status, average download speed, percentage of
  progress and path/URI. The percentage of progress and path/URI are
  printed for each requested file in each row.
  Default: ``default``

.. option:: --enable-async-dns6[=true|false]

  Enable IPv6 name resolution in asynchronous DNS resolver. This
  option will be ignored when :option:`--async-dns=false. <--async-dns>`
  Default: ``false``

.. option:: --event-poll=<POLL>

  Specify the method for polling events.  The possible values are
  ``epoll``, ``kqueue``, ``port``, ``poll`` and ``select``.  For each ``epoll``,
  ``kqueue``, ``port`` and ``poll``, it is available if system supports it.
  ``epoll`` is available on recent Linux. ``kqueue`` is available on
  various \*BSD systems including Mac OS X. ``port`` is available on Open
  Solaris. The default value may vary depending on the system you use.

.. option:: --file-allocation=<METHOD>

  Specify file allocation method.
  ``none`` doesn't pre-allocate file space. ``prealloc`` pre-allocates file space
  before download begins. This may take some time depending on the size of the
  file.
  If you are using newer file systems such as ext4
  (with extents support), btrfs, xfs or NTFS(MinGW build only), ``falloc`` is
  your best choice. It allocates large(few GiB)
  files almost instantly. Don't use ``falloc`` with
  legacy file systems such as ext3 and FAT32 because it takes
  almost same time as ``prealloc`` and it blocks aria2
  entirely until allocation finishes. ``falloc`` may
  not be available if your system doesn't have
  :manpage:`posix_fallocate(3)` function.
  Possible Values: ``none``, ``prealloc``, ``falloc``
  Default: ``prealloc``

.. option:: --hash-check-only[=true|false]

  If ``true`` is given, after hash check using
  :option:`--check-integrity <-V>` option,
  abort download whether or not download is complete.
  Default: ``false``

.. option:: --human-readable[=true|false]

  Print sizes and speed in human readable format (e.g., 1.2Ki, 3.4Mi)
  in the console readout. Default: ``true``

.. option:: --interface=<INTERFACE>

  Bind sockets to given interface. You can specify interface name, IP
  address and hostname.
  Possible Values: interface, IP address, hostname

  .. note::

    If an interface has multiple addresses, it is highly recommended to
    specify IP address explicitly. See also :option:`--disable-ipv6`.  If your
    system doesn't have :manpage:`getifaddrs(3)`, this option doesn't accept interface
    name.

.. option:: --max-download-result=<NUM>

  Set maximum number of download result kept in memory. The download
  results are completed/error/removed downloads. The download results
  are stored in FIFO queue and it can store at most NUM download
  results. When queue is full and new download result is created,
  oldest download result is removed from the front of the queue and
  new one is pushed to the back. Setting big number in this option may
  result high memory consumption after thousands of
  downloads. Specifying 0 means no download result is kept. Default:
  ``1000``

.. option:: --max-resume-failure-tries=<N>

  When used with :option:`--always-resume=false, <--always-resume>` aria2 downloads file from
  scratch when aria2 detects N number of URIs that does not support
  resume. If N is ``0``, aria2 downloads file from scratch when all
  given URIs do not support resume.  See :option:`--always-resume` option.
  Default: ``0``

.. option:: --log-level=<LEVEL>

  Set log level to output.
  LEVEL is either ``debug``, ``info``, ``notice``, ``warn`` or ``error``.
  Default: ``debug``

.. option:: --on-bt-download-complete=<COMMAND>

  For BitTorrent, a command specified in :option:`--on-download-complete` is
  called after download completed and seeding is over. On the other
  hand, this option set the command to be executed after download
  completed but before seeding.
  See `Event Hook`_ for more details about COMMAND.
  Possible Values: ``/path/to/command``

.. option:: --on-download-complete=<COMMAND>

  Set the command to be executed after download completed.  See
  See `Event Hook`_ for more details about COMMAND.
  See also :option:`--on-download-stop` option.
  Possible Values: ``/path/to/command``

.. option:: --on-download-error=<COMMAND>

  Set the command to be executed after download aborted due to error.
  See `Event Hook`_ for more details about COMMAND.
  See also :option:`--on-download-stop` option.  Possible Values:
  ``/path/to/command``

.. option:: --on-download-pause=<COMMAND>

  Set the command to be executed after download was paused.
  See `Event Hook`_ for more details about COMMAND.
  Possible Values: ``/path/to/command``

.. option:: --on-download-start=<COMMAND>

  Set the command to be executed after download got started.
  See `Event Hook`_ for more details about COMMAND.
  Possible Values: ``/path/to/command``

.. option:: --on-download-stop=<COMMAND>

  Set the command to be executed after download stopped. You can override
  the command to be executed for particular download result using
  :option:`--on-download-complete` and :option:`--on-download-error`. If they are
  specified, command specified in this option is not executed.
  See `Event Hook`_ for more details about COMMAND.
  Possible Values: ``/path/to/command``

.. option:: --piece-length=<LENGTH>

  Set a piece length for HTTP/FTP downloads. This is the boundary when
  aria2 splits a file. All splits occur at multiple of this
  length. This option will be ignored in BitTorrent downloads.  It
  will be also ignored if Metalink file contains piece hashes.
  Default: ``1M``

  .. note::
    
    The possible usecase of :option:`--piece-length`
    option is change the request range in one HTTP pipelined request.
    To enable HTTP pipelining use
    :option:`--enable-http-pipelining`.

.. option:: --show-console-readout[=true|false]

  Show console readout. Default: ``true``

.. option:: --summary-interval=<SEC>

  Set interval in seconds to output download progress summary.
  Setting ``0`` suppresses the output.
  Default: ``60``

  .. note::

    In multi file torrent downloads, the files adjacent forward to the specified files
    are also allocated if they share the same piece.

.. option:: -Z, --force-sequential[=true|false]

  Fetch URIs in the command-line sequentially and download each URI in a
  separate session, like the usual command-line download utilities.
  Default: ``false``

.. option:: --max-overall-download-limit=<SPEED>

  Set max overall download speed in bytes/sec.  ``0`` means
  unrestricted.  You can append ``K`` or ``M`` (1K = 1024, 1M = 1024K).  To
  limit the download speed per download, use :option:`--max-download-limit`
  option.  Default: ``0``

.. option:: --max-download-limit=<SPEED>

  Set max download speed per each download in bytes/sec. ``0`` means
  unrestricted.  You can append ``K`` or ``M`` (1K = 1024, 1M = 1024K).  To
  limit the overall download speed, use :option:`--max-overall-download-limit`
  option.  Default: ``0``

.. option:: --no-conf[=true|false]

  Disable loading aria2.conf file.

.. option:: --no-file-allocation-limit=<SIZE>

  No file allocation is made for files whose size is smaller than SIZE.
  You can append ``K`` or ``M`` (1K = 1024, 1M = 1024K).
  Default: ``5M``

.. option:: -P, --parameterized-uri[=true|false]

  Enable parameterized URI support.
  You can specify set of parts: ``http://{sv1,sv2,sv3}/foo.iso``.
  Also you can specify numeric sequences with step counter:
  ``http://host/image[000-100:2].img``.
  A step counter can be omitted.
  If all URIs do not point to the same file, such as the second example above,
  -Z option is required.
  Default: ``false``

.. option:: -q, --quiet[=true|false]

  Make aria2 quiet (no console output).
  Default: ``false``

.. option:: --realtime-chunk-checksum[=true|false]

   Validate chunk of data by calculating checksum while downloading a file if
   chunk checksums are provided.
   Default: ``true``


.. option:: --remove-control-file[=true|false]

   Remove control file before download. Using with
   :option:`--allow-overwrite=true, <--allow-overwrite>` download always starts from
   scratch. This will be useful for users behind proxy server which
   disables resume.

.. option:: --save-session=<FILE>

  Save error/unfinished downloads to FILE on exit.  You can pass this
  output file to aria2c with :option:`--input-file <-i>` option on restart. Please note that
  downloads added by :func:`aria2.addTorrent` and
  :func:`aria2.addMetalink`
  RPC method and whose metadata could not be saved as a file are not saved.
  Downloads removed using
  :func:`aria2.remove` and
  :func:`aria2.forceRemove`
  will not be saved.

.. option:: --stop=<SEC>

  Stop application after SEC seconds has passed.
  If ``0`` is given, this feature is disabled.
  Default: ``0``

.. option:: --stop-with-process=<PID>

  Stop application when process PID is not running.  This is useful if
  aria2 process is forked from a parent process. The parent process
  can fork aria2 with its own pid and when parent process exits for
  some reason, aria2 can detect it and shutdown itself.

.. option:: --truncate-console-readout[=true|false]

  Truncate console readout to fit in a single line.
  Default: ``true``
 
.. option:: -v, --version

  Print the version number, copyright and the configuration information and
  exit.

Options That Take An Optional Argument
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
The options that have its argument surrounded by square brackets([])
take an optional argument. Usually omiting the argument is evaluated to ``true``.
If you use short form of these options(such as ``-V``) and give
an argument, then the option name and its argument should be concatenated(e.g.
``-Vfalse``). If any spaces are inserted between the option name and the argument,
the argument will be treated as URI and usually this is not what you expect.

URI, MAGNET, TORRENT_FILE, METALINK_FILE
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can specify multiple URIs in command-line.  Unless you specify
:option:`--force-sequential <-Z>` option, all URIs must point to the same file or downloading will
fail.

You can specify arbitrary number of BitTorrent Magnet URI. Please note
that they are always treated as a separate download.  Both hex encoded
40 characters Info Hash and Base32 encoded 32 characters Info Hash are
supported. The multiple ``tr`` parameters are supported.  Because
BitTorrent Magnet URI is likely to contain ``&`` character, it is highly
recommended to always quote URI with single(``'``) or double(``"``) quotation.
It is strongly recommended to enable DHT especially when ``tr``
parameter is missing. See http://www.bittorrent.org/beps/bep_0009.html
for more details about BitTorrent Magnet URI.

You can also specify arbitrary number of torrent files and Metalink
documents stored on a local drive. Please note that they are always
treated as a separate download. Both Metalink4 and Metalink version
3.0 are supported.

You can specify both torrent file with -T option and URIs. By doing
this, you can download a file from both torrent swarm and HTTP(S)/FTP
server at the same time, while the data from HTTP(S)/FTP are uploaded
to the torrent swarm.  For single file torrents, URI can be a complete
URI pointing to the resource or if URI ends with /, name in torrent
file in torrent is added. For multi-file torrents, name and path are
added to form a URI for each file.

.. note::

  Make sure that URI is quoted with single(``'``) or double(``"``) quotation if it
  contains ``&`` or any characters that have special meaning in shell.

Resuming Download
~~~~~~~~~~~~~~~~~

Usually, you can resume transfer by just issuing same command(aria2c
URI) if the previous transfer is made by aria2.

If the previous transfer is made by a browser or wget like sequential
download manager, then use :option:`--continue <-c>` option to continue the transfer.

Event Hook
~~~~~~~~~~

aria2 provides options to specify arbitrary command after specific event
occurred. Currently following options are available:
:option:`--on-bt-download-complete`,
:option:`--on-download-pause`,
:option:`--on-download-complete`.
:option:`--on-download-start`,
:option:`--on-download-error`,
:option:`--on-download-stop`.

aria2 passes 3 arguments to specified command when it is executed.
These arguments are: GID, the number of files and file path.  For
HTTP, FTP downloads, usually the number of files is 1.  BitTorrent
download can contain multiple files.
If number of files is more than one, file path is first one.  In
other words, this is the value of path key of first struct whose
selected key is true in the response of
:func:`aria2.getFiles`
RPC method.
If you want to get all file paths, consider to use JSON-RPC/XML-RPC.  Please
note that file path may change during download in HTTP because of
redirection or Content-Disposition header.

Let's see an example of how arguments are passed to command:

.. code-block:: console

  $ cat hook.sh
  #!/bin/sh
  echo "Called with [$1] [$2] [$3]"
  $ aria2c --on-download-complete hook.sh http://example.org/file.iso
  Called with [1] [1] [/path/to/file.iso]


EXIT STATUS
-----------

Because aria2 can handle multiple downloads at once, it encounters
lots of errors in a session.  aria2 returns the following exit status
based on the last error encountered.

0
  If all downloads were successful.

1
  If an unknown error occurred.

2
  If time out occurred.

3
  If a resource was not found.

4
  If aria2 saw the specfied number of "resource not found" error.
  See :option:`--max-file-not-found` option).

5
  If a download aborted because download speed was too slow.
  See :option:`--lowest-speed-limit` option)

6
  If network problem occurred.

7
  If there were unfinished downloads. This error is only reported if
  all finished downloads were successful and there were unfinished
  downloads in a queue when aria2 exited by pressing :kbd:`Ctrl-C` by an user
  or sending TERM or INT signal.

8
  If remote server did not support resume when resume was required to
  complete download.

9
  If there was not enough disk space available.

10
  If piece length was different from one in .aria2 control file. See
  :option:`--allow-piece-length-change` option.

11
  If aria2 was downloading same file at that moment.

12
  If aria2 was downloading same info hash torrent at that moment.

13
  If file already existed. See :option:`--allow-overwrite` option.

14
  If renaming file failed. See :option:`--auto-file-renaming` option.

15
  If aria2 could not open existing file.

16
  If aria2 could not create new file or truncate existing file.

17
  If file I/O error occurred.

18
  If aria2 could not create directory.

19
  If name resolution failed.

20
  If aria2 could not parse Metalink document.

21
  If FTP command failed.

22
  If HTTP response header was bad or unexpected.

23
  If too many redirections occurred.

24
  If HTTP authorization failed.

25
  If aria2 could not parse bencoded file(usually .torrent file).

26
  If .torrent file was corrupted or missing information that aria2 needed.

27
  If Magnet URI was bad.

28
  If bad/unrecognized option was given or unexpected option argument
  was given.

29
  If the remote server was unable to handle the request due to a
  temporary overloading or maintenance.

30
  If aria2 could not parse JSON-RPC request.

.. note::

  An error occurred in a finished download will not be reported
  as exit status.

ENVIRONMENT
-----------
aria2 recognizes the following environment variables.

``http_proxy [http://][USER:PASSWORD@]HOST[:PORT]``
  Specify proxy server for use in HTTP.
  Overrides http-proxy value in configuration file.
  The command-line option :option:`--http-proxy` overrides this value.

``https_proxy [http://][USER:PASSWORD@]HOST[:PORT]``
  Specify proxy server for use in HTTPS.
  Overrides https-proxy value in configuration file.
  The command-line option :option:`--https-proxy` overrides this value.

``ftp_proxy [http://][USER:PASSWORD@]HOST[:PORT]``
  Specify proxy server for use in FTP.
  Overrides ftp-proxy value in configuration file.
  The command-line option :option:`--ftp-proxy` overrides this value.

``all_proxy [http://][USER:PASSWORD@]HOST[:PORT]``
  Specify proxy server for use if no protocol-specific proxy is specified.
  Overrides all-proxy value in configuration file.
  The command-line option :option:`--all-proxy` overrides this value.

.. note::
  
  Although aria2 accepts ``ftp://`` and ``https://`` scheme in proxy URI, it
  simply assumes that ``http://`` is specified and does not change its
  behavior based on the specified scheme.

``no_proxy [DOMAIN,...]``
  Specify comma-separated hostname, domains and network address with
  or without CIDR block to which proxy should not be used.  Overrides
  no-proxy value in configuration file.  The command-line option
  :option:`--no-proxy` overrides this value.

FILES
-----
aria2.conf
~~~~~~~~~~

By default, aria2 parses ``$HOME/.aria2/aria2.conf`` as a configuraiton
file. You can specify the path to configuration file using
:option:`--conf-path` option.  If you don't want to use the configuraitonf
file, use :option:`--no-conf` option.

The configuration file is a text file and has 1 option per each
line. In each line, you can specify name-value pair in the format:
``NAME=VALUE``, where name is the long command-line option name without
``--`` prefix. You can use same syntax for the command-line option. The
lines beginning ``#`` are treated as comments::

  # sample configuration file for aria2c
  listen-port=60000
  dht-listen-port=60000
  seed-ratio=1.0
  max-upload-limit=50K
  ftp-pasv=true


dht.dat
~~~~~~~~

By default, the routing table of IPv4 DHT is saved to the path
``$HOME/.aria2/dht.dat`` and the routing table of IPv6 DHT is saved to
the path ``$HOME/.aria2/dht6.dat``.

Netrc
~~~~~

Netrc support is enabled by default for HTTP(S)/FTP.  To disable netrc
support, specify :option:`--no-netrc <-n>` option.  Your .netrc file should have correct
permissions(600).

If machine name starts ``.``, aria2 performs domain-match instead of
exact match. This is an extension of aria2. For example of domain
match, imagine the following .netrc entry::

  machine .example.org login myid password mypasswd


``aria2.example.org`` domain-matches ``.example.org`` and uses ``myid`` and
``mypasswd``.

Some domain-match example follow: ``example.net`` does not domain-match
``.example.org``. ``example.org`` does not domain-match ``.example.org``
because of preceding ``.``. If you want to match ``example.org``, specify
``example.org``.

Control File
~~~~~~~~~~~~

aria2 uses a control file to track the progress of a download.  A
control file is placed in the same directory as the downloading file
and its filename is the filename of downloading file with ``.aria2``
appended.  For example, if you are downloading file.zip, then the
control file should be file.zip.aria2.  (There is a exception for this
naming convention.  If you are downloading a multi torrent, its
control file is the "top directory" name of the torrent with ``.aria2``
appended.  The "top directory" name is a value of "name" key in "info"
directory in a torrent file.)

Usually a control file is deleted once download completed.  If aria2
decides that download cannot be resumed(for example, when downloading
a file from a HTTP server which doesn't support resume), a control
file is not created.

Normally if you lose a control file, you cannot resume download.  But
if you have a torrent or metalink with chunk checksums for the file,
you can resume the download without a control file by giving -V option
to aria2c in command-line.

Input File
~~~~~~~~~~

The input file can contain a list of URIs for aria2 to download.  You
can specify multiple URIs for a single entity: separate URIs on a
single line using the TAB character.

Each line is treated as if it is provided in command-line argument.
Therefore they are affected by :option:`--force-sequential <-Z>` and :option:`--parameterized-uri <-P>` options.

Since URIs in the input file are directly read by aria2, they must not
be quoted with single(``'``) or double(``"``) quotation.

Lines starting with ``#`` are treated as comments and skipped.

Additionally, the following options can be specified after each line
of URIs. These optional lines must start with white space(s).

.. hlist::
  :columns: 3

  * :option:`all-proxy <--all-proxy>`
  * :option:`all-proxy-passwd <--all-proxy-passwd>`
  * :option:`all-proxy-user <--all-proxy-user>`
  * :option:`allow-overwrite <--allow-overwrite>`
  * :option:`allow-piece-length-change <--allow-piece-length-change>`
  * :option:`always-resume <--always-resume>`
  * :option:`async-dns <--async-dns>`
  * :option:`auto-file-renaming <--auto-file-renaming>`
  * :option:`bt-enable-lpd <--bt-enable-lpd>`
  * :option:`bt-exclude-tracker <--bt-exclude-tracker>`
  * :option:`bt-external-ip <--bt-external-ip>`
  * :option:`bt-hash-check-seed <--bt-hash-check-seed>`
  * :option:`bt-max-open-files <--bt-max-open-files>`
  * :option:`bt-max-peers <--bt-max-peers>`
  * :option:`bt-metadata-only <--bt-metadata-only>`
  * :option:`bt-min-crypto-level <--bt-min-crypto-level>`
  * :option:`bt-prioritize-piece <--bt-prioritize-piece>`
  * :option:`bt-request-peer-speed-limit <--bt-request-peer-speed-limit>`
  * :option:`bt-require-crypto <--bt-require-crypto>`
  * :option:`bt-save-metadata <--bt-save-metadata>`
  * :option:`bt-seed-unverified <--bt-seed-unverified>`
  * :option:`bt-stop-timeout <--bt-stop-timeout>`
  * :option:`bt-tracker <--bt-tracker>`
  * :option:`bt-tracker-connect-timeout <--bt-tracker-connect-timeout>`
  * :option:`bt-tracker-interval <--bt-tracker-interval>`
  * :option:`bt-tracker-timeout <--bt-tracker-timeout>`
  * :option:`bt-remove-unselected-file <--bt-remove-unselected-file>`
  * :option:`check-integrity <-V>`
  * :option:`conditional-get <--conditional-get>`
  * :option:`connect-timeout <--connect-timeout>`
  * :option:`continue <-c>`
  * :option:`dir <-d>`
  * :option:`dry-run <--dry-run>`
  * :option:`enable-async-dns6 <--enable-async-dns6>`
  * :option:`enable-http-keep-alive <--enable-http-keep-alive>`
  * :option:`enable-http-pipelining <--enable-http-pipelining>`
  * :option:`enable-peer-exchange <--enable-peer-exchange>`
  * :option:`file-allocation <--file-allocation>`
  * :option:`follow-metalink <--follow-metalink>`
  * :option:`follow-torrent <--follow-torrent>`
  * :option:`ftp-passwd <--ftp-passwd>`
  * :option:`ftp-pasv <-p>`
  * :option:`ftp-proxy <--ftp-proxy>`
  * :option:`ftp-proxy-passwd <--ftp-proxy-passwd>`
  * :option:`ftp-proxy-user <--ftp-proxy-user>`
  * :option:`ftp-reuse-connection <--ftp-reuse-connection>`
  * :option:`ftp-type <--ftp-type>`
  * :option:`ftp-user <--ftp-user>`
  * :option:`header <--header>`
  * :option:`http-accept-gzip <--http-accept-gzip>`
  * :option:`http-auth-challenge <--http-auth-challenge>`
  * :option:`http-no-cache <--http-no-cache>`
  * :option:`http-passwd <--http-passwd>`
  * :option:`http-proxy <--http-proxy>`
  * :option:`http-proxy-passwd <--http-proxy-passwd>`
  * :option:`http-proxy-user <--http-proxy-user>`
  * :option:`http-user <--http-user>`
  * :option:`https-proxy <--https-proxy>`
  * :option:`https-proxy-passwd <--https-proxy-passwd>`
  * :option:`https-proxy-user <--https-proxy-user>`
  * :option:`index-out <-O>`
  * :option:`lowest-speed-limit <--lowest-speed-limit>`
  * :option:`max-connection-per-server <-x>`
  * :option:`max-download-limit <--max-download-limit>`
  * :option:`max-file-not-found <--max-file-not-found>`
  * :option:`max-resume-failure-tries <--max-resume-failure-tries>`
  * :option:`max-tries <-m>`
  * :option:`max-upload-limit <-u>`
  * :option:`metalink-enable-unique-protocol <--metalink-enable-unique-protocol>`
  * :option:`metalink-language <--metalink-language>`
  * :option:`metalink-location <--metalink-location>`
  * :option:`metalink-os <--metalink-os>`
  * :option:`metalink-preferred-protocol <--metalink-preferred-protocol>`
  * :option:`metalink-version <--metalink-version>`
  * :option:`min-split-size <-k>`
  * :option:`no-file-allocation-limit <--no-file-allocation-limit>`
  * :option:`no-netrc <-n>`
  * :option:`no-proxy <--no-proxy>`
  * :option:`out <-o>`
  * :option:`parameterized-uri <-P>`
  * :option:`proxy-method <--proxy-method>`
  * :option:`realtime-chunk-checksum <--realtime-chunk-checksum>`
  * :option:`referer <--referer>`
  * :option:`remote-time <-R>`
  * :option:`remove-control-file <--remove-control-file>`
  * :option:`reuse-uri <--reuse-uri>`
  * :option:`seed-ratio <--seed-ratio>`
  * :option:`seed-time <--seed-time>`
  * :option:`select-file <--select-file>`
  * :option:`split <-s>`
  * :option:`timeout <-t>`
  * :option:`use-head <--use-head>`
  * :option:`user-agent <-U>`
  * :option:`retry-wait <--retry-wait>`
  * :option:`metalink-base-uri <--metalink-base-uri>`
  * :option:`pause <--pause>`
  * :option:`stream-piece-selector <--stream-piece-selector>`
  * :option:`hash-check-only <--hash-check-only>`
  * :option:`checksum <--checksum>`
  * :option:`piece-length <--piece-length>`
  * :option:`uri-selector <--uri-selector>`
  
These options have exactly same meaning of the ones in the
command-line options, but it just applies to the URIs it belongs to.
Please note that for options in input file ``--`` prefix must be
stripped.

For example, the content of uri.txt is::

  http://server/file.iso http://mirror/file.iso
    dir=/iso_images
    out=file.img
  http://foo/bar


If aria2 is executed with ``-i uri.txt -d /tmp`` options, then
``file.iso`` is saved as ``/iso_images/file.img`` and it is downloaded
from \http://server/file.iso and \http://mirror/file.iso.  The file
``bar`` is downloaded from \http://foo/bar and saved as ``/tmp/bar``.

In some cases, :option:`out <-o>` parameter has no effect.
See note of :option:`--out <-o>`
option for the restrictions.

Server Performance Profile
~~~~~~~~~~~~~~~~~~~~~~~~~~

This section describes the format of server performance profile.  The
file is plain text and each line has several ``NAME=VALUE`` pair,
delimited by comma.  Currently following NAMEs are recognized:

``host``
  Hostname of the server. Required.

``protocol``
  Protocol for this profile, such as ftp, http. Required.

``dl_speed``
  The average download speed observed in the previous download in
  bytes per sec.  Required.

``sc_avg_speed``
  The average download speed observed in the previous download in
  bytes per sec. This value is only updated if the download is done in
  single connection environment and only used by
  AdaptiveURISelector. Optional.

``mc_avg_speed``
  The average download speed observed in the previous download in
  bytes per sec. This value is only updated if the download is done in
  multi connection environment and only used by
  AdaptiveURISelector. Optional.

``counter``
  How many times the server is used. Currently this value is only used
  by AdaptiveURISelector.  Optional.

``last_updated``
  Last contact time in GMT with this server, specified in the seconds
  since the Epoch(00:00:00 on January 1, 1970, UTC). Required.

``status``
  ERROR is set when server cannot be reached or out-of-service or
  timeout occurred. Otherwise, OK is set.

Those fields must exist in one line. The order of the fields is not
significant. You can put pairs other than the above; they are simply
ignored.

An example follows::

  host=localhost, protocol=http, dl_speed=32000, last_updated=1222491640, status=OK
  host=localhost, protocol=ftp, dl_speed=0, last_updated=1222491632, status=ERROR


RPC INTERFACE
-------------

aria2 provides JSON-RPC over HTTP and XML-RPC over HTTP and they
basically have the same functionality.  aria2 also provides JSON-RPC
over WebSocket. JSON-RPC over WebSocket uses same method signatures
and response format with JSON-RPC over HTTP, but it additionally has
server-initiated notifications. See `JSON-RPC over WebSocket`_ section for details.

The request path of JSON-RPC interface (for both over HTTP and over
WebSocket) is ``/jsonrpc``.  The request path of XML-RPC interface is
``/rpc``.

The WebSocket URI for JSON-RPC over WebSocket is
``ws://HOST:PORT/jsonrpc``.

The implemented JSON-RPC is based on
JSON-RPC 2.0 <http://jsonrpc.org/specification>, and
supports HTTP POST and GET (JSONP). Using WebSocket as a transport is
the original extension of aria2.

The JSON-RPC interface does not support notification in HTTP, but the
RPC server will send the notification in WebSocket. It also does not
support floating point number. The character encoding must be UTF-8.

When reading following document for JSON-RPC, interpret struct as JSON
object.

Terminology
~~~~~~~~~~~

GID
  GID(or gid) is the key to manage each download. Each download has an
  unique GID. Currently GID looks like an integer, but don't treat it
  as integer because it may be changed to another type in the future
  release. Please note that GID is session local and not persisted
  when aria2 exits.

Methods
~~~~~~~

All code examples come from Python2.7 interpreter.


.. function:: aria2.addUri(uris[, options[, position]])

  
  
  
  
  This method adds new HTTP(S)/FTP/BitTorrent Magnet URI.  *uris* is of
  type array and its element is URI which is of type string.  For
  BitTorrent Magnet URI, *uris* must have only one element and it should
  be BitTorrent Magnet URI.  URIs in *uris* must point to the same file.
  If you mix other URIs which point to another file, aria2 does not
  complain but download may fail.  *options* is of type struct and its
  members are a pair of option name and value. See :ref:`rpc_options` below for
  more details.  If *position* is given as an integer starting from 0,
  the new download is inserted at *position* in the waiting queue. If
  *position* is not given or *position* is larger than the size of the
  queue, it is appended at the end of the queue.  This method returns
  GID of registered download.
  
  **JSON-RPC Example**
  
  
  The following example adds \http://example.org/file to aria2::
  
    >>> import urllib2, json
    >>> jsonreq = json.dumps({'jsonrpc':'2.0', 'id':'qwer',
    ...                       'method':'aria2.addUri',
    ...                       'params':[['http://example.org/file']]})
    >>> c = urllib2.urlopen('http://localhost:6800/jsonrpc', jsonreq)
    >>> c.read()
    '{"id":"qwer","jsonrpc":"2.0","result":"1"}'
  
  
  **XML-RPC Example**
  
  
  The following example adds \http://example.org/file to aria2::
  
    >>> import xmlrpclib
    >>> s = xmlrpclib.ServerProxy('http://localhost:6800/rpc')
    >>> s.aria2.addUri(['http://example.org/file'])
    '1'
  
  
  The following example adds 2 sources and some options::
  
    >>> s.aria2.addUri(['http://example.org/file', 'http://mirror/file'],
                        dict(dir="/tmp"))
    '2'
  
  
  The following example adds a download and insert it to the front of
  waiting downloads::
  
    >>> s.aria2.addUri(['http://example.org/file'], {}, 0)
    '3'
  
  

.. function:: aria2.addTorrent(torrent[, uris[, options[, position]]])

  
  
  
  
  This method adds BitTorrent download by uploading .torrent file.  If
  you want to add BitTorrent Magnet URI, use :func:`aria2.addUri` method
  instead.  *torrent* is of type base64 which contains Base64-encoded
  .torrent file.  *uris* is of type array and its element is URI which
  is of type string. *uris* is used for Web-seeding.  For single file
  torrents, URI can be a complete URI pointing to the resource or if URI
  ends with /, name in torrent file is added. For multi-file torrents,
  name and path in torrent are added to form a URI for each file.
  *options* is of type struct and its members are a pair of option name
  and value. See :ref:`rpc_options` below for more details.  If *position* is
  given as an integer starting from 0, the new download is inserted at
  *position* in the waiting queue. If *position* is not given or
  *position* is larger than the size of the queue, it is appended at the
  end of the queue.  This method returns GID of registered download.
  The uploaded data is saved as a file named hex string of SHA-1 hash of data
  plus ".torrent" in the directory specified by :option:`--dir <-d>`
  option.
  The example of filename is 0a3893293e27ac0490424c06de4d09242215f0a6.torrent.
  If same file already exists, it is overwritten.
  If the file cannot be saved successfully,
  the downloads added by this method are not saved by
  :option:`--save-session`.
  
  **JSON-RPC Example**
  
  
  The following example adds local file file.torrent to aria2::
  
    >>> import urllib2, json, base64
    >>> torrent = base64.b64encode(open('file.torrent').read())
    >>> jsonreq = json.dumps({'jsonrpc':'2.0', 'id':'asdf',
    ...                       'method':'aria2.addTorrent', 'params':[torrent]})
    >>> c = urllib2.urlopen('http://localhost:6800/jsonrpc', jsonreq)
    >>> c.read()
    '{"id":"asdf","jsonrpc":"2.0","result":"6"}'
  
  
  **XML-RPC Example**
  
  
  The following example adds local file file.torrent to aria2::
  
    >>> import xmlrpclib
    >>> s = xmlrpclib.ServerProxy('http://localhost:6800/rpc')
    >>> s.aria2.addTorrent(xmlrpclib.Binary(open('file.torrent').read()))
    '6'
  
  

.. function:: aria2.addMetalink(metalink[, options[, position]])

  
  
  
  
  This method adds Metalink download by uploading .metalink file.
  *metalink* is of type base64 which contains Base64-encoded .metalink
  file.  *options* is of type struct and its members are a pair of
  option name and value. See :ref:`rpc_options` below for more details.  If
  *position* is given as an integer starting from 0, the new download is
  inserted at *position* in the waiting queue. If *position* is not
  given or *position* is larger than the size of the queue, it is
  appended at the end of the queue.  This method returns array of GID of
  registered download.
  The uploaded data is saved as a file named hex string of SHA-1 hash of data
  plus ".metalink" in the directory specified by :option:`--dir <-d>`
  option.
  The example of filename is 0a3893293e27ac0490424c06de4d09242215f0a6.metalink.
  If same file already exists, it is overwritten.
  If the file cannot be saved successfully,
  the downloads added by this method are not saved by
  :option:`--save-session`.
  
  **JSON-RPC Example**
  
  
  The following example adds local file file.meta4 to aria2::
  
    >>> import urllib2, json, base64
    >>> metalink = base64.b64encode(open('file.meta4').read())
    >>> jsonreq = json.dumps({'jsonrpc':'2.0', 'id':'qwer',
    ...                       'method':'aria2.addMetalink', 'params':[metalink]})
    >>> c = urllib2.urlopen('http://localhost:6800/jsonrpc', jsonreq)
    >>> c.read()
    '{"id":"qwer","jsonrpc":"2.0","result":["8"]}'
  
  
  **XML-RPC Example**
  
  
  The following example adds local file file.meta4 to aria2::
  
    >>> import xmlrpclib
    >>> s = xmlrpclib.ServerProxy('http://localhost:6800/rpc')
    >>> s.aria2.addMetalink(xmlrpclib.Binary(open('file.meta4').read()))
    ['8']
  
  

.. function:: aria2.remove(gid)

  
  
  
  
  This method removes the download denoted by *gid*. *gid* is of type
  string. If specified download is in progress, it is stopped at
  first. The status of removed download becomes ``"removed"``.  This method
  returns GID of removed download.
  
  **JSON-RPC Example**
  
  
  The following example removes download whose GID is "3"::
  
    >>> import urllib2, json
    >>> jsonreq = json.dumps({'jsonrpc':'2.0', 'id':'qwer',
    ...                       'method':'aria2.remove', 'params':['3']})
    >>> c = urllib2.urlopen('http://localhost:6800/jsonrpc', jsonreq)
    >>> c.read()
    '{"id":"qwer","jsonrpc":"2.0","result":"3"}'
  
  
  **XML-RPC Example**
  
  
  The following example removes download whose GID is "3"::
  
    >>> import xmlrpclib
    >>> s = xmlrpclib.ServerProxy('http://localhost:6800/rpc')
    >>> s.aria2.remove('3')
    '3'
  
  

.. function:: aria2.forceRemove(gid)

  
  
  
  
  This method removes the download denoted by *gid*.  This method
  behaves just like :func:`aria2.remove` except that this method removes
  download without any action which takes time such as contacting
  BitTorrent tracker.
  

.. function:: aria2.pause(gid)

  
  
  
  
  This method pauses the download denoted by *gid*. *gid* is of type
  string. The status of paused download becomes ``"paused"``.  If the
  download is active, the download is placed on the first position of
  waiting queue.  As long as the status is ``"paused"``, the download is not
  started.  To change status to ``"waiting"``, use :func:`aria2.unpause` method.
  This method returns GID of paused download.
  

.. function:: aria2.pauseAll()

  
  
  
  
  This method is equal to calling :func:`aria2.pause` for every active/waiting
  download. This methods returns ``"OK"`` for success.
  

.. function:: aria2.forcePause(pid)

  
  
  
  
  This method pauses the download denoted by *gid*.  This method
  behaves just like :func:`aria2.pause` except that this method pauses
  download without any action which takes time such as contacting
  BitTorrent tracker.
  

.. function:: aria2.forcePauseAll()

  
  
  
  
  This method is equal to calling :func:`aria2.forcePause` for every
  active/waiting download. This methods returns ``"OK"`` for success.
  

.. function:: aria2.unpause(gid)

  
  
  
  
  This method changes the status of the download denoted by *gid* from
  ``"paused"`` to ``"waiting"``. This makes the download eligible to restart.
  *gid* is of type string.  This method returns GID of unpaused
  download.
  

.. function:: aria2.unpauseAll()

  
  
  
  
  This method is equal to calling :func:`aria2.unpause` for every active/waiting
  download. This methods returns ``"OK"`` for success.
  

.. function:: aria2.tellStatus(gid[, keys])

  
  
  
  
  This method returns download progress of the download denoted by
  *gid*. *gid* is of type string. *keys* is array of string. If it is
  specified, the response contains only keys in *keys* array. If *keys*
  is empty or not specified, the response contains all keys.  This is
  useful when you just want specific keys and avoid unnecessary
  transfers. For example, ``aria2.tellStatus("1", ["gid", "status"])``
  returns *gid* and 'status' key.  The response is of type struct and it
  contains following keys. The value type is string.
  
  ``gid``
    GID of this download.
  
  ``status``
    ``"active"`` for currently downloading/seeding entry. ``"waiting"`` for the
    entry in the queue; download is not started.  ``"paused"`` for the
    paused entry.  ``"error"`` for the stopped download because of
    error. ``"complete"`` for the stopped and completed download. ``"removed"``
    for the download removed by user.
  
  ``totalLength``
    Total length of this download in bytes.
  
  ``completedLength``
    Completed length of this download in bytes.
  
  ``uploadLength``
    Uploaded length of this download in bytes.
  
  ``bitfield``
    Hexadecimal representation of the download progress. The highest bit
    corresponds to piece index 0. The set bits indicate the piece is
    available and unset bits indicate the piece is missing. The spare
    bits at the end are set to zero.  When download has not started yet,
    this key will not be included in the response.
  
  ``downloadSpeed``
    Download speed of this download measured in bytes/sec.
  
  ``uploadSpeed``
    Upload speed of this download measured in bytes/sec.
  
  ``infoHash``
    InfoHash. BitTorrent only.
  
  ``numSeeders``
    The number of seeders the client has connected to. BitTorrent only.
  
  ``pieceLength``
    Piece length in bytes.
  
  ``numPieces``
    The number of pieces.
  
  ``connections``
    The number of peers/servers the client has connected to.
  
  ``errorCode``
    The last error code occurred in this download. The value is of type
    string. The error codes are defined in `EXIT STATUS`_ section. This value is only available for
    stopped/completed downloads.
  
  ``followedBy``
    List of GIDs which are generated by the consequence of this
    download. For example, when aria2 downloaded Metalink file, it
    generates downloads described in it(see :option:`--follow-metalink`
    option). This value is useful to track these auto generated
    downloads. If there is no such downloads, this key will not
    be included in the response.
  
  ``belongsTo``
    GID of a parent download. Some downloads are a part of another
    download.  For example, if a file in Metalink has BitTorrent
    resource, the download of .torrent is a part of that file.  If this
    download has no parent, this key will not be included in the
    response.
  
  ``dir``
    Directory to save files. This key is not available for stopped
    downloads.
  
  ``files``
    Returns the list of files. The element of list is the same struct
    used in :func:`aria2.getFiles` method.
  
  
  ``bittorrent``
    Struct which contains information retrieved from .torrent
    file. BitTorrent only. It contains following keys.
  
    ``announceList``
      List of lists of announce URI. If .torrent file contains announce
      and no announce-list, announce is converted to announce-list
      format.
  
    ``comment``
      The comment for the torrent. comment.utf-8 is used if available.
  
    ``creationDate``
      The creation time of the torrent. The value is an integer since
      the Epoch, measured in seconds.
  
    ``mode``
      File mode of the torrent. The value is either 'single' or 'multi'.
  
    ``info``
      Struct which contains data from Info dictionary. It contains
      following keys.
  
      ``name``
        name in info dictionary. name.utf-8 is used if available.
  
  **JSON-RPC Example**
  
  
  The following example gets information about download whose GID is
  "1"::
  
    >>> import urllib2, json
    >>> from pprint import pprint
    >>> jsonreq = json.dumps({'jsonrpc':'2.0', 'id':'qwer',
    ...                       'method':'aria2.tellStatus', 'params':['1']})
    >>> c = urllib2.urlopen('http://localhost:6800/jsonrpc', jsonreq)
    >>> pprint(json.loads(c.read()))
    {u'id': u'qwer',
     u'jsonrpc': u'2.0',
     u'result': {u'bitfield': u'0000000000',
                 u'completedLength': u'901120',
                 u'connections': u'1',
                 u'dir': u'/downloads',
                 u'downloadSpeed': u'15158',
                 u'files': [{u'index': u'1',
                             u'length': u'34896138',
                             u'completedLength': u'34896138',
                             u'path': u'/downloads/file',
                             u'selected': u'true',
                             u'uris': [{u'status': u'used',
                                        u'uri': u'http://example.org/file'}]}],
                 u'gid': u'1',
                 u'numPieces': u'34',
                 u'pieceLength': u'1048576',
                 u'status': u'active',
                 u'totalLength': u'34896138',
                 u'uploadLength': u'0',
                 u'uploadSpeed': u'0'}}
  
  
  The following example gets information specifying keys you are
  interested in::
  
    >>> jsonreq = json.dumps({'jsonrpc':'2.0', 'id':'qwer',
    ...                       'method':'aria2.tellStatus',
    ...                       'params':['1', ['gid',
    ...                                       'totalLength',
    ...                                       'completedLength']]})
    >>> c = urllib2.urlopen('http://localhost:6800/jsonrpc', jsonreq)
    >>> pprint(json.loads(c.read()))
    {u'id': u'qwer',
     u'jsonrpc': u'2.0',
     u'result': {u'completedLength': u'5701632',
                 u'gid': u'1',
                 u'totalLength': u'34896138'}}
  
  
  **XML-RPC Example**
  
  
  The following example gets information about download whose GID is
  "1"::
  
    >>> import xmlrpclib
    >>> from pprint import pprint
    >>> s = xmlrpclib.ServerProxy('http://localhost:6800/rpc')
    >>> r = s.aria2.tellStatus('1')
    >>> pprint(r)
    {'bitfield': 'ffff80',
     'completedLength': '34896138',
     'connections': '0',
     'dir': '/downloads',
     'downloadSpeed': '0',
     'errorCode': '0',
     'files': [{'index': '1',
                'length': '34896138',
                'completedLength': '34896138',
                'path': '/downloads/file',
                'selected': 'true',
                'uris': [{'status': 'used',
                          'uri': 'http://example.org/file'}]}],
     'gid': '1',
     'numPieces': '17',
     'pieceLength': '2097152',
     'status': 'complete',
     'totalLength': '34896138',
     'uploadLength': '0',
     'uploadSpeed': '0'}
  
  
  The following example gets information specifying keys you are
  interested in::
  
    >>> r = s.aria2.tellStatus('1', ['gid', 'totalLength', 'completedLength'])
    >>> pprint(r)
    {'completedLength': '34896138', 'gid': '1', 'totalLength': '34896138'}
  
  

.. function:: aria2.getUris(gid)

  
  
  
  
  This method returns URIs used in the download denoted by *gid*.  *gid*
  is of type string. The response is of type array and its element is of
  type struct and it contains following keys. The value type is string.
  
  ``uri``
    URI
  
  ``status``
    'used' if the URI is already used. 'waiting' if the URI is waiting
    in the queue.
  
  **JSON-RPC Example**
  ::
  
    >>> import urllib2, json
    >>> from pprint import pprint
    >>> jsonreq = json.dumps({'jsonrpc':'2.0', 'id':'qwer',
    ...                       'method':'aria2.getUris', 'params':['1']})
    >>> c = urllib2.urlopen('http://localhost:6800/jsonrpc', jsonreq)
    >>> pprint(json.loads(c.read()))
    {u'id': u'qwer',
     u'jsonrpc': u'2.0',
     u'result': [{u'status': u'used',
                  u'uri': u'http://example.org/file'}]}
  
  
  **XML-RPC Example**
  ::
  
    >>> import xmlrpclib
    >>> from pprint import pprint
    >>> s = xmlrpclib.ServerProxy('http://localhost:6800/rpc')
    >>> r = s.aria2.getUris('1')
    >>> pprint(r)
    [{'status': 'used', 'uri': 'http://example.org/file'}]
  
  

.. function:: aria2.getFiles(gid)

  
  
  
  
  This method returns file list of the download denoted by *gid*. *gid*
  is of type string. The response is of type array and its element is of
  type struct and it contains following keys. The value type is string.
  
  ``index``
    Index of file. Starting with 1. This is the same order with the
    files in multi-file torrent.
  
  ``path``
    File path.
  
  ``length``
    File size in bytes.
  
  ``completedLength``
    Completed length of this file in bytes.  Please note that it is
    possible that sum of completedLength is less than completedLength in
    :func:`aria2.tellStatus` method.
    This is because completedLength in
    :func:`aria2.getFiles`
    only calculates completed pieces. On the other hand, completedLength
    in
    :func:`aria2.tellStatus` takes into account
    of partially completed piece.
  
  ``selected``
    ``"true"`` if this file is selected by :option:`--select-file` option. If
    :option:`--select-file` is not specified or this is single torrent or no
    torrent download, this value is always ``"true"``. Otherwise ``"false"``.
  
  ``uris``
    Returns the list of URI for this file. The element of list is the
    same struct used in :func:`aria2.getUris` method.
  
  **JSON-RPC Example**
  ::
  
    >>> import urllib2, json
    >>> from pprint import pprint
    >>> jsonreq = json.dumps({'jsonrpc':'2.0', 'id':'qwer',
    ...                       'method':'aria2.getFiles', 'params':['1']})
    >>> c = urllib2.urlopen('http://localhost:6800/jsonrpc', jsonreq)
    >>> pprint(json.loads(c.read()))
    {u'id': u'qwer',
     u'jsonrpc': u'2.0',
     u'result': [{u'index': u'1',
                  u'length': u'34896138',
                  u'completedLength': u'34896138',
                  u'path': u'/downloads/file',
                  u'selected': u'true',
                  u'uris': [{u'status': u'used',
                             u'uri': u'http://example.org/file'}]}]}
  
  
  **XML-RPC Example**
  ::
  
    >>> import xmlrpclib
    >>> from pprint import pprint
    >>> s = xmlrpclib.ServerProxy('http://localhost:6800/rpc')
    >>> r = s.aria2.getFiles('1')
    >>> pprint(r)
    [{'index': '1',
      'length': '34896138',
      'completedLength': '34896138',
      'path': '/downloads/file',
      'selected': 'true',
      'uris': [{'status': 'used',
                'uri': 'http://example.org/file'}]}]
  
  

.. function:: aria2.getPeers(gid)

  
  
  
  
  This method returns peer list of the download denoted by *gid*. *gid*
  is of type string. This method is for BitTorrent only.  The response
  is of type array and its element is of type struct and it contains
  following keys. The value type is string.
  
  ``peerId``
    Percent-encoded peer ID.
  
  ``ip``
    IP address of the peer.
  
  ``port``
    Port number of the peer.
  
  ``bitfield``
    Hexadecimal representation of the download progress of the peer. The
    highest bit corresponds to piece index 0. The set bits indicate the
    piece is available and unset bits indicate the piece is missing. The
    spare bits at the end are set to zero.
  
  ``amChoking``
    ``"true"`` if this client is choking the peer. Otherwise ``"false"``.
  
  ``peerChoking``
    ``"true"`` if the peer is choking this client. Otherwise ``"false"``.
  
  ``downloadSpeed``
    Download speed (byte/sec) that this client obtains from the peer.
  
  ``uploadSpeed``
    Upload speed(byte/sec) that this client uploads to the peer. 
  
  ``seeder``
    ``"true"`` is this client is a seeder. Otherwise ``"false"``.
  
  **JSON-RPC Example**
  ::
  
    >>> import urllib2, json
    >>> from pprint import pprint
    >>> jsonreq = json.dumps({'jsonrpc':'2.0', 'id':'qwer',
    ...                       'method':'aria2.getPeers', 'params':['1']})
    >>> c = urllib2.urlopen('http://localhost:6800/jsonrpc', jsonreq)
    >>> pprint(json.loads(c.read()))
    {u'id': u'qwer',
     u'jsonrpc': u'2.0',
     u'result': [{u'amChoking': u'true',
                  u'bitfield': u'ffffffffffffffffffffffffffffffffffffffff',
                  u'downloadSpeed': u'10602',
                  u'ip': u'10.0.0.9',
                  u'peerChoking': u'false',
                  u'peerId': u'aria2%2F1%2E10%2E5%2D%87%2A%EDz%2F%F7%E6',
                  u'port': u'6881',
                  u'seeder': u'true',
                  u'uploadSpeed': u'0'},
                 {u'amChoking': u'false',
                  u'bitfield': u'ffffeff0fffffffbfffffff9fffffcfff7f4ffff',
                  u'downloadSpeed': u'8654',
                  u'ip': u'10.0.0.30',
                  u'peerChoking': u'false',
                  u'peerId': u'bittorrent client758',
                  u'port': u'37842',
                  u'seeder': u'false',
                  u'uploadSpeed': u'6890'}]}
  
  
  **XML-RPC Example**
  ::
  
    >>> import xmlrpclib
    >>> from pprint import pprint
    >>> s = xmlrpclib.ServerProxy('http://localhost:6800/rpc')
    >>> r = s.aria2.getPeers('1')
    >>> pprint(r)
    [{'amChoking': 'true',
      'bitfield': 'ffffffffffffffffffffffffffffffffffffffff',
      'downloadSpeed': '10602',
      'ip': '10.0.0.9',
      'peerChoking': 'false',
      'peerId': 'aria2%2F1%2E10%2E5%2D%87%2A%EDz%2F%F7%E6',
      'port': '6881',
      'seeder': 'true',
      'uploadSpeed': '0'},
     {'amChoking': 'false',
      'bitfield': 'ffffeff0fffffffbfffffff9fffffcfff7f4ffff',
      'downloadSpeed': '8654',
      'ip': '10.0.0.30',
      'peerChoking': 'false',
      'peerId': 'bittorrent client758',
      'port': '37842',
      'seeder': 'false,
      'uploadSpeed': '6890'}]
  
  

.. function:: aria2.getServers(gid)

  
  
  
  
  This method returns currently connected HTTP(S)/FTP servers of the download denoted by *gid*. *gid* is of type string. The response
  is of type array and its element is of type struct and it contains
  following keys. The value type is string.
  
  ``index``
    Index of file. Starting with 1. This is the same order with the
    files in multi-file torrent.
  
  ``servers``
    The list of struct which contains following keys.
  
    ``uri``
      URI originally added.
  
    ``currentUri``
      This is the URI currently used for downloading. If redirection is
      involved, currentUri and uri may differ.
  
    ``downloadSpeed``
      Download speed (byte/sec)
  
  **JSON-RPC Example**
  ::
  
    >>> import urllib2, json
    >>> from pprint import pprint
    >>> jsonreq = json.dumps({'jsonrpc':'2.0', 'id':'qwer',
    ...                       'method':'aria2.getServers', 'params':['1']})
    >>> c = urllib2.urlopen('http://localhost:6800/jsonrpc', jsonreq)
    >>> pprint(json.loads(c.read()))
    {u'id': u'qwer',
     u'jsonrpc': u'2.0',
     u'result': [{u'index': u'1',
                  u'servers': [{u'currentUri': u'http://example.org/file',
                                u'downloadSpeed': u'10467',
                                u'uri': u'http://example.org/file'}]}]}
  
  
  **XML-RPC Example**
  ::
  
    >>> import xmlrpclib
    >>> from pprint import pprint
    >>> s = xmlrpclib.ServerProxy('http://localhost:6800/rpc')
    >>> r = s.aria2.getServers('1')
    >>> pprint(r)
    [{'index': '1',
      'servers': [{'currentUri': 'http://example.org/dl/file',
                   'downloadSpeed': '20285',
                   'uri': 'http://example.org/file'}]}]
  
  

.. function:: aria2.tellActive([keys])

  
  
  
  
  This method returns the list of active downloads.  The response is of
  type array and its element is the same struct returned by
  :func:`aria2.tellStatus` method. For *keys* parameter, please refer to
  :func:`aria2.tellStatus` method.
  

.. function:: aria2.tellWaiting(offset, num, [keys])

  
  
  
  
  This method returns the list of waiting download, including paused
  downloads. *offset* is of type integer and specifies the offset from
  the download waiting at the front. *num* is of type integer and
  specifies the number of downloads to be returned.  For *keys*
  parameter, please refer to :func:`aria2.tellStatus` method.
  
  If *offset* is a positive integer, this method returns downloads in the
  range of [*offset*, *offset* + *num*).
  
  *offset* can be a negative integer. *offset* == -1 points last
  download in the waiting queue and *offset* == -2 points the download
  before the last download, and so on. The downloads in the response are
  in reversed order.
  
  For example, imagine that three downloads "A","B" and "C" are waiting
  in this order. aria2.tellWaiting(0, 1) returns
  ["A"]. aria2.tellWaiting(1, 2) returns ["B", "C"].
  aria2.tellWaiting(-1, 2) returns ["C", "B"].
  
  The response is of type array and its element is the same struct
  returned by :func:`aria2.tellStatus` method.
  

.. function:: aria2.tellStopped(offset, num, [keys])

  
  
  
  
  This method returns the list of stopped download.  *offset* is of type
  integer and specifies the offset from the oldest download. *num* is of
  type integer and specifies the number of downloads to be returned.
  For *keys* parameter, please refer to :func:`aria2.tellStatus` method.
  
  *offset* and *num* have the same semantics as :func:`aria2.tellWaiting`
  method.
  
  The response is of type array and its element is the same struct
  returned by :func:`aria2.tellStatus` method.
  

.. function:: aria2.changePosition(gid, pos, how)

  
  
  
  
  This method changes the position of the download denoted by
  *gid*. *pos* is of type integer. *how* is of type string. If *how* is
  ``"POS_SET"``, it moves the download to a position relative to the
  beginning of the queue.  If *how* is ``"POS_CUR"``, it moves the download
  to a position relative to the current position. If *how* is ``"POS_END"``,
  it moves the download to a position relative to the end of the
  queue. If the destination position is less than 0 or beyond the end of
  the queue, it moves the download to the beginning or the end of the
  queue respectively. The response is of type integer and it is the
  destination position.
  
  For example, if GID#1 is placed in position 3, ``aria2.changePosition('1',
  -1, 'POS_CUR')`` will change its position to 2. Additional
  ``aria2.changePosition('1', 0, 'POS_SET')`` will change its position to 0(the
  beginning of the queue).
  
  **JSON-RPC Example**
  
  
  The following example moves the download whose GID is "3" to the
  front of the waiting queue::
  
    >>> import urllib2, json
    >>> from pprint import pprint
    >>> jsonreq = json.dumps({'jsonrpc':'2.0', 'id':'qwer',
    ...                       'method':'aria2.changePosition',
    ...                       'params':['3', 0, 'POS_SET']})
    >>> c = urllib2.urlopen('http://localhost:6800/jsonrpc', jsonreq)
    >>> pprint(json.loads(c.read()))
    {u'id': u'qwer', u'jsonrpc': u'2.0', u'result': 0}
  
  
  **XML-RPC Example**
  
  
  The following example moves the download whose GID is "3" to the
  front of the waiting queue::
  
    >>> import xmlrpclib
    >>> s = xmlrpclib.ServerProxy('http://localhost:6800/rpc')
    >>> s.aria2.changePosition('3', 0, 'POS_SET')
    0
  
  

.. function:: aria2.changeUri(gid, fileIndex, delUris, addUris[, position])

  
  
  
  
  This method removes URIs in *delUris* from and appends URIs in
  *addUris* to download denoted by *gid*. *delUris* and *addUris* are
  list of string. A download can contain multiple files and URIs are
  attached to each file.  *fileIndex* is used to select which file to
  remove/attach given URIs. *fileIndex* is 1-based. *position* is used
  to specify where URIs are inserted in the existing waiting URI
  list. *position* is 0-based. When *position* is omitted, URIs are
  appended to the back of the list.  This method first execute removal
  and then addition. *position* is the position after URIs are removed,
  not the position when this method is called.  When removing URI, if
  same URIs exist in download, only one of them is removed for each URI
  in *delUris*. In other words, there are three URIs
  ``http://example.org/aria2`` and you want remove them all, you have to
  specify (at least) 3 ``http://example.org/aria2`` in *delUris*.  This
  method returns a list which contains 2 integers. The first integer is
  the number of URIs deleted. The second integer is the number of URIs
  added.
  
  **JSON-RPC Example**
  
  
  The following example adds 1 URI \http://example.org/file to the file
  whose index is "1" and belongs to the download whose GID is "2"::
  
    >>> import urllib2, json
    >>> from pprint import pprint
    >>> jsonreq = json.dumps({'jsonrpc':'2.0', 'id':'qwer',
    ...                       'method':'aria2.changeUri',
    ...                       'params':['2', 1, [], ['http://example.org/file']]})
    >>> c = urllib2.urlopen('http://localhost:6800/jsonrpc', jsonreq)
    >>> pprint(json.loads(c.read()))
    {u'id': u'qwer', u'jsonrpc': u'2.0', u'result': [0, 1]}
  
  
  **XML-RPC Example**
  
  
  The following example adds 1 URI \http://example.org/file to the file
  whose index is "1" and belongs to the download whose GID is "2"::
  
    >>> import xmlrpclib
    >>> s = xmlrpclib.ServerProxy('http://localhost:6800/rpc')
    >>> s.aria2.changeUri('2', 1, [], ['http://example.org/file'])
    [0, 1]
  
  

.. function:: aria2.getOption(gid)

  
  
  
  
  This method returns options of the download denoted by *gid*.  The
  response is of type struct. Its key is the name of option.  The value
  type is string. Note that this method does not return options which
  have no default value and have not been set by the command-line
  options, configuration files or RPC methods.
  
  **JSON-RPC Example**
  
  
  The following example gets options of the download whose GID is "1"::
  
    >>> import urllib2, json
    >>> from pprint import pprint
    >>> jsonreq = json.dumps({'jsonrpc':'2.0', 'id':'qwer',
    ...                       'method':'aria2.getOption', 'params':['1']})
    >>> c = urllib2.urlopen('http://localhost:6800/jsonrpc', jsonreq)
    >>> pprint(json.loads(c.read()))
    {u'id': u'qwer',
     u'jsonrpc': u'2.0',
     u'result': {u'allow-overwrite': u'false',
                 u'allow-piece-length-change': u'false',
                 u'always-resume': u'true',
                 u'async-dns': u'true',
     ...
  
  
  **XML-RPC Example**
  
  
  The following example gets options of the download whose GID is "1"::
  
    >>> import xmlrpclib
    >>> from pprint import pprint
    >>> s = xmlrpclib.ServerProxy('http://localhost:6800/rpc')
    >>> r = s.aria2.getOption('1')
    >>> pprint(r)
    {'allow-overwrite': 'false',
     'allow-piece-length-change': 'false',
     'always-resume': 'true',
     'async-dns': 'true',
     ....
  
  

.. function:: aria2.changeOption(gid, options)

  
  
  
  
  This method changes options of the download denoted by *gid*
  dynamically.  *gid* is of type string.  *options* is of type struct.
  The following options are available for active downloads:
  
  * :option:`bt-max-peers <--bt-max-peers>`
  * :option:`bt-request-peer-speed-limit <--bt-request-peer-speed-limit>`
  * :option:`bt-remove-unselected-file <--bt-remove-unselected-file>`
  * :option:`max-download-limit <--max-download-limit>`
  * :option:`max-upload-limit <-u>`
  
  For waiting or paused downloads, in addition to the above options,
  options listed in `Input File`_ subsection are available,
  except for following options:
  :option:`dry-run <--dry-run>`,
  :option:`metalink-base-uri <--metalink-base-uri>`,
  :option:`parameterized-uri <-P>`,
  :option:`pause <--pause>` and
  :option:`piece-length <--piece-length>`.
  This method returns ``"OK"`` for success.
  
  **JSON-RPC Example**
  
  
  The following example sets
  :option:`max-download-limit <--max-download-limit>` option to ``"20K"`` for
  the download whose GID is "1"::
  
    >>> import urllib2, json
    >>> from pprint import pprint
    >>> jsonreq = json.dumps({'jsonrpc':'2.0', 'id':'qwer',
    ...                       'method':'aria2.changeOption',
    ...                       'params':['1', {'max-download-limit':'10K'}]})
    >>> c = urllib2.urlopen('http://localhost:6800/jsonrpc', jsonreq)
    >>> pprint(json.loads(c.read()))
    {u'id': u'qwer', u'jsonrpc': u'2.0', u'result': u'OK'}
  
  
  **XML-RPC Example**
  
  
  The following example sets
  :option:`max-download-limit <--max-download-limit>` option to ``"20K"`` for
  the download whose GID is "1"::
  
    >>> import xmlrpclib
    >>> s = xmlrpclib.ServerProxy('http://localhost:6800/rpc')
    >>> s.aria2.changeOption('1', {'max-download-limit':'20K'})
    'OK'
  
  

.. function:: aria2.getGlobalOption()

  
  
  
  
  This method returns global options.  The response is of type
  struct. Its key is the name of option.  The value type is string.
  Note that this method does not return options which have no default
  value and have not been set by the command-line options, configuration
  files or RPC methods. Because global options are used as a template
  for the options of newly added download, the response contains keys
  returned by :func:`aria2.getOption` method.
  

.. function:: aria2.changeGlobalOption(options)

  
  
  
  
  This method changes global options dynamically.  *options* is of type
  struct.
  The following options are available:
  
  * :option:`download-result <--download-result>`
  * :option:`log <-l>`
  * :option:`log-level <--log-level>`
  * :option:`max-concurrent-downloads <-j>`
  * :option:`max-download-result <--max-download-result>`
  * :option:`max-overall-download-limit <--max-overall-download-limit>`
  * :option:`max-overall-upload-limit <--max-overall-upload-limit>`
  * :option:`save-cookies <--save-cookies>`
  * :option:`save-session <--save-session>`
  * :option:`server-stat-of <--server-stat-of>`
  
  In addition to them, options listed in `Input File`_ subsection
  are available, except for following options:
  :option:`checksum <--checksum>`,
  :option:`index-out <-O>`,
  :option:`out <-o>`,
  :option:`pause <--pause>` and
  :option:`select-file <--select-file>`.
  
  Using :option:`log <-l>` option, you can dynamically start logging or
  change log file. To stop logging, give empty string("") as a parameter
  value. Note that log file is always opened in append mode. This method
  returns ``"OK"`` for success.
  

.. function:: aria2.getGlobalStat()

  
  
  
  
  This method returns global statistics such as overall download and
  upload speed. The response is of type struct and contains following
  keys. The value type is string.
  
  ``downloadSpeed``
    Overall download speed (byte/sec).
  
  ``uploadSpeed``
    Overall upload speed(byte/sec).
  
  ``numActive``
    The number of active downloads.
  
  ``numWaiting``
    The number of waiting downloads.
  
  ``numStopped``
    The number of stopped downloads.
  
  **JSON-RPC Example**
  ::
  
    >>> import urllib2, json
    >>> from pprint import pprint
    >>> jsonreq = json.dumps({'jsonrpc':'2.0', 'id':'qwer',
    ...                       'method':'aria2.getGlobalStat'})
    >>> c = urllib2.urlopen('http://localhost:6800/jsonrpc', jsonreq)
    >>> pprint(json.loads(c.read()))
    {u'id': u'qwer',
     u'jsonrpc': u'2.0',
     u'result': {u'downloadSpeed': u'21846',
                 u'numActive': u'2',
                 u'numStopped': u'0',
                 u'numWaiting': u'0',
                 u'uploadSpeed': u'0'}}
  
  
  **XML-RPC Example**
  ::
  
    >>> import xmlrpclib
    >>> from pprint import pprint
    >>> s = xmlrpclib.ServerProxy('http://localhost:6800/rpc')
    >>> r = s.aria2.getGlobalStat()
    >>> pprint(r)
    {'downloadSpeed': '23136',
     'numActive': '2',
     'numStopped': '0',
     'numWaiting': '0',
     'uploadSpeed': '0'}
  
  

.. function:: aria2.purgeDownloadResult()

  
  
  
  
  This method purges completed/error/removed downloads to free memory.
  This method returns ``"OK"``.
  

.. function:: aria2.removeDownloadResult(gid)

  
  
  
  
  This method removes completed/error/removed download denoted by *gid*
  from memory. This method returns ``"OK"`` for success.
  
  **JSON-RPC Example**
  
  
  The following example removes the download result of the download
  whose GID is "1"::
  
    >>> import urllib2, json
    >>> from pprint import pprint
    >>> jsonreq = json.dumps({'jsonrpc':'2.0', 'id':'qwer',
    ...                       'method':'aria2.removeDownloadResult',
    ...                       'params':['1']})
    >>> c = urllib2.urlopen('http://localhost:6800/jsonrpc', jsonreq)
    >>> pprint(json.loads(c.read()))
    {u'id': u'qwer', u'jsonrpc': u'2.0', u'result': u'OK'}
  
  
  **XML-RPC Example**
  
  
  The following example removes the download result of the download
  whose GID is "1"::
  
    >>> import xmlrpclib
    >>> s = xmlrpclib.ServerProxy('http://localhost:6800/rpc')
    >>> s.aria2.removeDownloadResult('1')
    'OK'
  
  

.. function:: aria2.getVersion()

  
  
  
  
  This method returns version of the program and the list of enabled
  features. The response is of type struct and contains following keys.
  
  ``version``
    Version number of the program in string.
  
  ``enabledFeatures``
    List of enabled features. Each feature name is of type string.
  
  **JSON-RPC Example**
  ::
  
    >>> import urllib2, json
    >>> from pprint import pprint
    >>> jsonreq = json.dumps({'jsonrpc':'2.0', 'id':'qwer',
    ...                       'method':'aria2.getVersion'})
    >>> c = urllib2.urlopen('http://localhost:6800/jsonrpc', jsonreq)
    >>> pprint(json.loads(c.read()))
    {u'id': u'qwer',
     u'jsonrpc': u'2.0',
     u'result': {u'enabledFeatures': [u'Async DNS',
                                      u'BitTorrent',
                                      u'Firefox3 Cookie',
                                      u'GZip',
                                      u'HTTPS',
                                      u'Message Digest',
                                      u'Metalink',
                                      u'XML-RPC'],
                 u'version': u'1.11.0'}}
  
  
  **XML-RPC Example**
  ::
  
    >>> import xmlrpclib
    >>> from pprint import pprint
    >>> s = xmlrpclib.ServerProxy('http://localhost:6800/rpc')
    >>> r = s.aria2.getVersion()
    >>> pprint(r)
    {'enabledFeatures': ['Async DNS',
                         'BitTorrent',
                         'Firefox3 Cookie',
                         'GZip',
                         'HTTPS',
                         'Message Digest',
                         'Metalink',
                         'XML-RPC'],
     'version': '1.11.0'}
  
  

.. function:: aria2.getSessionInfo()

  
  
  
  
  This method returns session information.
  The response is of type struct and contains following key.
  
  ``sessionId``
    Session ID, which is generated each time when aria2 is invoked.
  
  **JSON-RPC Example**
  ::
  
    >>> import urllib2, json
    >>> from pprint import pprint
    >>> jsonreq = json.dumps({'jsonrpc':'2.0', 'id':'qwer',
    ...                       'method':'aria2.getSessionInfo'})
    >>> c = urllib2.urlopen('http://localhost:6800/jsonrpc', jsonreq)
    >>> pprint(json.loads(c.read()))
    {u'id': u'qwer',
     u'jsonrpc': u'2.0',
     u'result': {u'sessionId': u'cd6a3bc6a1de28eb5bfa181e5f6b916d44af31a9'}}
  
  
  **XML-RPC Example**
  ::
  
    >>> import xmlrpclib
    >>> s = xmlrpclib.ServerProxy('http://localhost:6800/rpc')
    >>> s.aria2.getSessionInfo()
    {'sessionId': 'cd6a3bc6a1de28eb5bfa181e5f6b916d44af31a9'}
  
  

.. function:: aria2.shutdown()

  
  
  
  
  This method shutdowns aria2.  This method returns ``"OK"``.
  

.. function:: aria2.forceShutdown()

  
  
  
  
  This method shutdowns :func:`aria2. This method behaves like  aria2.shutdown`
  except that any actions which takes time such as contacting BitTorrent
  tracker are skipped. This method returns ``"OK"``.
  

.. function:: system.multicall(methods)

  
  
  
  
  This methods encapsulates multiple method calls in a single request.
  *methods* is of type array and its element is struct.  The struct
  contains two keys: ``methodName`` and ``params``.  ``methodName`` is the
  method name to call and ``params`` is array containing parameters to the
  method.  This method returns array of responses.  The element of array
  will either be a one-item array containing the return value of each
  method call or struct of fault element if an encapsulated method call
  fails.
  
  **JSON-RPC Example**
  
  
  In the following example, we add 2 downloads. First one is
  \http://example.org/file and second one is file.torrent::
  
    >>> import urllib2, json, base64
    >>> from pprint import pprint
    >>> jsonreq = json.dumps({'jsonrpc':'2.0', 'id':'qwer',
    ...                       'method':'system.multicall',
    ...                       'params':[[{'methodName':'aria2.addUri',
    ...                                   'params':[['http://example.org']]},
    ...                                  {'methodName':'aria2.addTorrent',
    ...                                   'params':[base64.b64encode(open('file.torrent').read())]}]]})
    >>> c = urllib2.urlopen('http://localhost:6800/jsonrpc', jsonreq)
    >>> pprint(json.loads(c.read()))
    {u'id': u'qwer', u'jsonrpc': u'2.0', u'result': [[u'1'], [u'2']]}
  
  
  JSON-RPC also supports Batch request described in JSON-RPC 2.0 Specification::
  
    >>> jsonreq = json.dumps([{'jsonrpc':'2.0', 'id':'qwer',
    ...                        'method':'aria2.addUri',
    ...                        'params':[['http://example.org']]},
    ...                       {'jsonrpc':'2.0', 'id':'asdf',
    ...                        'method':'aria2.addTorrent',
    ...                        'params':[base64.b64encode(open('file.torrent').read())]}])
    >>> c = urllib2.urlopen('http://localhost:6800/jsonrpc', jsonreq)
    >>> pprint(json.loads(c.read()))
    [{u'id': u'qwer', u'jsonrpc': u'2.0', u'result': u'1'},
     {u'id': u'asdf', u'jsonrpc': u'2.0', u'result': u'2'}]
  
  
  **XML-RPC Example**
  
  
  In the following example, we add 2 downloads. First one is
  \http://example.org/file and second one is file.torrent::
  
    >>> import xmlrpclib
    >>> s = xmlrpclib.ServerProxy('http://localhost:6800/rpc')
    >>> mc = xmlrpclib.MultiCall(s)
    >>> mc.aria2.addUri(['http://example.org/file'])
    >>> mc.aria2.addTorrent(xmlrpclib.Binary(open('file.torrent').read()))
    >>> r = mc()
    >>> tuple(r)
    ('2', '3')
  
  
Error Handling
~~~~~~~~~~~~~~

In JSON-RPC, aria2 returns JSON object which contains error code in
code and the error message in message.

In XML-RPC, aria2 returns faultCode=1 and the error message in
faultString.

.. _rpc_options:

Options
~~~~~~~

Same options for :option:`--input-file <-i>` list are available. See `Input File`_ subsection for complete list of options.

In the option struct, name element is option name(without preceding
``--``) and value element is argument as string.

JSON-RPC Example
^^^^^^^^^^^^^^^^
::

  {'split':'1', 'http-proxy':'http://proxy/'}


XML-RPC Example
^^^^^^^^^^^^^^^
.. code-block:: xml

  <struct>
    <member>
      <name>split</name>
      <value><string>1</string></value>
    </member>
    <member>
      <name>http-proxy</name>
      <value><string>http://proxy/</string></value>
    </member>
  </struct>


:option:`header <--header>` and :option:`index-out <-O>`
option are allowed multiple times in
command-line. Since name should be unique in struct(many XML-RPC
library implementation uses hash or dict for struct), single string is
not enough. To overcome this situation, they can take array as value
as well as string.

JSON-RPC Example
^^^^^^^^^^^^^^^^
::

  {'header':['Accept-Language: ja', 'Accept-Charset: utf-8']}


XML-RPC Example
^^^^^^^^^^^^^^^
.. code-block:: xml

  <struct>
    <member>
      <name>header</name>
      <value>
        <array>
          <data>
            <value><string>Accept-Language: ja</string></value>
            <value><string>Accept-Charset: utf-8</string></value>
          </data>
        </array>
      </value>
    </member>
  </struct>


Following example adds a download with 2 options: dir and header.
header option has 2 values, so it uses a list::

  >>> import xmlrpclib
  >>> s = xmlrpclib.ServerProxy('http://localhost:6800/rpc')
  >>> opts = dict(dir='/tmp',
  ...             header=['Accept-Language: ja',
  ...                     'Accept-Charset: utf-8'])
  >>> s.aria2.addUri(['http://example.org/file'], opts)
  '1'


JSON-RPC using HTTP GET
~~~~~~~~~~~~~~~~~~~~~~~

The JSON-RPC interface also supports request via HTTP GET.
The encoding scheme in GET parameters is based on JSON-RPC over HTTP Specification [2008-1-15(RC1)].
The encoding of GET parameters are follows::

  /jsonrpc?method=METHOD_NAME&id=ID&params=BASE64_ENCODED_PARAMS


The ``method`` and ``id`` are always treated as JSON string and their
encoding must be UTF-8.

For example, The encoded string of aria2.tellStatus('3') with id='foo'
looks like this::

  /jsonrpc?method=aria2.tellStatus&id=foo&params=WyIzIl0%3D


The ``params`` parameter is Base64-encoded JSON array which usually
appears in ``params`` attribute in JSON-RPC request object.  In the
above example, the params is ['3'], therefore::

  ['3'] --(Base64)--> WyIzIl0= --(Percent Encode)--> WyIzIl0%3D


The JSON-RPC interface supports JSONP. You can specify the callback
function in 'jsoncallback' parameter::

  /jsonrpc?method=aria2.tellStatus&id=foo&params=WyIzIl0%3D&jsoncallback=cb


For Batch request, ``method`` and ``id`` parameter must not be specified.
Whole request must be specified in ``params`` parameter. For example,
Batch request::

  [{'jsonrpc':'2.0', 'id':'qwer', 'method':'aria2.getVersion'},
   {'jsonrpc':'2.0', 'id':'asdf', 'method':'aria2.tellActive'}]


will be encoded like this::

  /jsonrpc?params=W3sianNvbnJwYyI6ICIyLjAiLCAiaWQiOiAicXdlciIsICJtZXRob2QiOiAiYXJpYTIuZ2V0VmVyc2lvbiJ9LCB7Impzb25ycGMiOiAiMi4wIiwgImlkIjogImFzZGYiLCAibWV0aG9kIjogImFyaWEyLnRlbGxBY3RpdmUifV0%3D


JSON-RPC over WebSocket
~~~~~~~~~~~~~~~~~~~~~~~

JSON-RPC over WebSocket uses same method signatures and response
format with JSON-RPC over HTTP. The supported WebSocket version is 13
which is detailed in :rfc:`6455`.

To send a RPC request to the RPC server, send serialized JSON string
in Text frame. The response from the RPC server is delivered also in
Text frame.

The RPC server will send the notification to the client. The
notification is unidirectional, therefore the client which received
the notification must not respond to it. The method signature of
notification is much like a normal method request but lacks id
key. The value associated by the params key is the data which this
notification carries. The format of this value varies depending on the
notification method. Following notification methods are defined.


.. function:: aria2.onDownloadStart(event)

  
  This notification will be sent if a download is started.
  The *event* is of type struct and it contains following keys.
  The value type is string.
  
  ``gid``
    GID of the download.
  

.. function:: aria2.onDownloadPause(event)

  
  This notification will be sent if a download is paused.  The *event*
  is the same struct of the *event* argument of
  :func:`aria2.onDownloadStart` method.
  

.. function:: aria2.onDownloadStop(event)

  
  This notification will be sent if a download is stopped by the user.
  The *event* is the same struct of the *event* argument of
  :func:`aria2.onDownloadStart` method.
  

.. function:: aria2.onDownloadComplete(event)

  
  This notification will be sent if a download is completed.  In
  BitTorrent downloads, this notification is sent when the download is
  completed and seeding is over. The *event* is the same struct of the
  *event* argument of 
  :func:`aria2.onDownloadStart` method.
  

.. function:: aria2.onDownloadError(event)

  
  This notification will be sent if a download is stopped due to error.
  The *event* is the same struct of the *event* argument of
  :func:`aria2.onDownloadStart` method.
  

.. function:: aria2.onBtDownloadComplete(event)

  
  This notification will be sent if a download is completed in
  BitTorrent (but seeding may not be over).  The *event* is the same struct
  of the *event* argument of 
  :func:`aria2.onDownloadStart` method.
  
Sample XML-RPC Client Code
~~~~~~~~~~~~~~~~~~~~~~~~~~

The following Ruby script adds ``http://localhost/aria2.tar.bz2`` to
aria2c operated on localhost with option :option:`--dir=/downloads <-d>` and
prints its reponse:

.. code-block:: ruby

  #!/usr/bin/env ruby
  
  require 'xmlrpc/client'
  require 'pp'
  
  client=XMLRPC::Client.new2("http://localhost:6800/rpc")
  
  options={ "dir" => "/downloads" }
  result=client.call("aria2.addUri", [ "http://localhost/aria2.tar.bz2" ], options)
  
  pp result


If you are a Python lover, you can use xmlrpclib(for Python3.x, use
xmlrpc.client instead) to interact with aria2::

  import xmlrpclib
  from pprint import pprint
  
  s = xmlrpclib.ServerProxy("http://localhost:6800/rpc")
  r = s.aria2.addUri(["http://localhost/aria2.tar.bz2"], {"dir":"/downloads"})
  pprint(r)


EXAMPLE
-------

HTTP/FTP Segmented Download
~~~~~~~~~~~~~~~~~~~~~~~~~~~
Download a file
^^^^^^^^^^^^^^^
.. code-block:: console

  $ aria2c "http://host/file.zip"


.. note::

  To stop a download, press :kbd:`Ctrl-C`. You can resume the transfer by running aria2c with the same argument in the same directory. You can change URIs as long as they are pointing to the same file.

Download a file from 2 different HTTP servers
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: console

  $ aria2c "http://host/file.zip" "http://mirror/file.zip"


Download a file from 1 host using 2 connections
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: console

  $ aria2c -x2 -k1M "http://host/file.zip"


Download a file from HTTP and FTP servers
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: console

  $ aria2c "http://host1/file.zip" "ftp://host2/file.zip"


Download files listed in a text file concurrently
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: console

  $ aria2c -ifiles.txt -j2

.. note::

  -j option specifies the number of parallel downloads.

Using proxy
^^^^^^^^^^^
For HTTP:

.. code-block:: console

  $ aria2c --http-proxy="http://proxy:8080" "http://host/file"


.. code-block:: console

  $ aria2c --http-proxy="http://proxy:8080" --no-proxy="localhost,127.0.0.1,192.168.0.0/16" "http://host/file"


For FTP:

.. code-block:: console

  $ aria2c --ftp-proxy="http://proxy:8080" "ftp://host/file"


.. note::
  
  See :option:`--http-proxy`, :option:`--https-proxy`, :option:`--ftp-proxy`, :option:`--all-proxy` and
  :option:`--no-proxy` for details.  You can specify proxy in the environment
  variables. See `ENVIRONMENT`_ section.

Proxy with authorization
^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: console

  $ aria2c --http-proxy="http://username:password@proxy:8080" "http://host/file"


.. code-block:: console

  $ aria2c --http-proxy="http://proxy:8080" --http-proxy-user="username" --http-proxy-passwd="password" "http://host/file"


Metalink Download
~~~~~~~~~~~~~~~~~
Download files with remote Metalink
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: console

  $ aria2c --follow-metalink=mem "http://host/file.metalink"


Download using a local metalink file
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: console

  $ aria2c -p --lowest-speed-limit=4000 file.metalink

.. note::

  To stop a download, press :kbd:`Ctrl-C`.
  You can resume the transfer by running aria2c with the same argument in the same
  directory.

Download several local metalink files
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: console

  $ aria2c -j2 file1.metalink file2.metalink


Download only selected files using index
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: console

  $ aria2c --select-file=1-4,8 file.metalink

.. note::

  The index is printed to the console using -S option.

Download a file using a local .metalink file with user preference
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: console

  $ aria2c --metalink-location=jp,us --metalink-version=1.1 --metalink-language=en-US file.metalink


BitTorrent Download
~~~~~~~~~~~~~~~~~~~
Download files from remote BitTorrent file
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: console

  $ aria2c --follow-torrent=mem "http://host/file.torrent"


Download using a local torrent file
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: console

  $ aria2c --max-upload-limit=40K file.torrent

.. note::

  --max-upload-limit specifies the max of upload rate.

.. note::

  To stop a download, press :kbd:`Ctrl-C`. You can resume the transfer by running aria2c with the same argument in the same directory.

Download using BitTorrent Magnet URI
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: console

  $ aria2c "magnet:?xt=urn:btih:248D0A1CD08284299DE78D5C1ED359BB46717D8C&dn=aria2"


.. note::

  Don't forget to quote BitTorrent Magnet URI which includes ``&``
  character with single(``'``) or double(``"``) quotation.

Download 2 torrents
^^^^^^^^^^^^^^^^^^^
.. code-block:: console

  $ aria2c -j2 file1.torrent file2.torrent


Download a file using torrent and HTTP/FTP server
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: console

  $ aria2c -Ttest.torrent "http://host1/file" "ftp://host2/file"

.. note::

  Downloading multi file torrent with HTTP/FTP is not supported.

Download only selected files using index(usually called "selectable download")
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: console

  $ aria2c --select-file=1-4,8 file.torrent

.. note::

  The index is printed to the console using -S option.

Specify output filename
^^^^^^^^^^^^^^^^^^^^^^^

To specify output filename for BitTorrent downloads, you need to know
the index of file in torrent file using :option:`--show-files <-S>` option. For example, the
output looks like this::

  idx|path/length
  ===+======================
    1|dist/base-2.6.18.iso
     |99.9MiB
  ---+----------------------
    2|dist/driver-2.6.18.iso
     |169.0MiB
  ---+----------------------


To save 'dist/base-2.6.18.iso' in '/tmp/mydir/base.iso' and
'dist/driver-2.6.18.iso' in '/tmp/dir/driver.iso', use the following
command:

.. code-block:: console

  $ aria2c --dir=/tmp --index-out=1=mydir/base.iso --index-out=2=dir/driver.iso file.torrent


Change the listening port for incoming peer
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: console

  $ aria2c --listen-port=7000-7001,8000 file.torrent

.. note::

  Since aria2 doesn't configure firewall or router for port forwarding, it's up
  to you to do it manually.

Specify the condition to stop program after torrent download finished
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: console

  $ aria2c --seed-time=120 --seed-ratio=1.0 file.torrent


.. note::

  In the above example, the program exits when the 120 minutes has elapsed since download completed or seed ratio reaches 1.0.

Throttle upload speed
^^^^^^^^^^^^^^^^^^^^^
.. code-block:: console

  $ aria2c --max-upload-limit=100K file.torrent


Enable IPv4 DHT
^^^^^^^^^^^^^^^
.. code-block:: console

  $ aria2c --enable-dht --dht-listen-port=6881 file.torrent

.. note::

  DHT uses udp port. Since aria2 doesn't configure firewall or router for port
  forwarding, it's up to you to do it manually.

Enable IPv6 DHT
^^^^^^^^^^^^^^^
.. code-block:: console

  $ aria2c --enable-dht6 --dht-listen-port=6881 --dht-listen-addr6=YOUR_GLOBAL_UNICAST_IPV6_ADDR --enable-async-dns6

.. note::

  If aria2c is not built with c-ares, :option:`--enable-async-dns6` is
  unnecessary. aria2 shares same port between IPv4 and IPv6 DHT.

Add and remove tracker URI
^^^^^^^^^^^^^^^^^^^^^^^^^^

Removes all tracker announce URIs described in file.torrent and use
``http://tracker1/announce`` and ``http://tracker2/announce`` instead:

.. code-block:: console

  $ aria2c --bt-exclude-tracker="*" --bt-tracker="http://tracker1/announce,http://tracker2/announce" file.torrent


More advanced HTTP features
~~~~~~~~~~~~~~~~~~~~~~~~~~~
Load cookies
^^^^^^^^^^^^
.. code-block:: console

  $ aria2c --load-cookies=cookies.txt "http://host/file.zip"

.. note::

  You can use Firefox/Mozilla/Chromium's cookie file without modification.

Resume download started by web browsers or another programs
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: console

  $ aria2c -c -s2 "http://host/partiallydownloadedfile.zip"


Client certificate authorization for SSL/TLS
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: console

  $ aria2c --certificate=/path/to/mycert.pem --private-key=/path/to/mykey.pem https://host/file

.. note::

  The file specified in :option:`--private-key` must be decrypted. The behavior when
  encrypted one is given is undefined.

Verify peer in SSL/TLS using given CA certificates
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: console

  $ aria2c --ca-certificate=/path/to/ca-certificates.crt --check-certificate https://host/file


And more advanced features
~~~~~~~~~~~~~~~~~~~~~~~~~~
Throttle download speed
^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: console

  $ aria2c --max-download-limit=100K file.metalink


Repair a damaged download
^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: console

  $ aria2c -V file.metalink

.. note::
  
  Repairing damaged downloads can be done efficiently when used with
  BitTorrent or Metalink with chunk checksums.

Drop connection if download speed is lower than specified value
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: console

  $ aria2c --lowest-speed-limit=10K file.metalink


Parameterized URI support
^^^^^^^^^^^^^^^^^^^^^^^^^
You can specify set of parts:

.. code-block:: console

  $ aria2c -P "http://{host1,host2,host3}/file.iso"


You can specify numeric sequence:

.. code-block:: console

  $ aria2c -Z -P "http://host/image[000-100].png"


.. note::

  -Z option is required if the all URIs don't point to the same file, such as the above example.

You can specify step counter:

.. code-block:: console

  $ aria2c -Z -P "http://host/image[A-Z:2].png"


Verify checksum
^^^^^^^^^^^^^^^
.. code-block:: console

  $ aria2c --checksum=sha-1=0192ba11326fe2298c8cb4de616f4d4140213837 http://example.org/file


Parallel downloads of arbitrary number of URI,metalink,torrent
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: console

  $ aria2c -j3 -Z "http://host/file1" file2.torrent file3.metalink


BitTorrent Encryption
^^^^^^^^^^^^^^^^^^^^^
Encrypt whole payload using ARC4:

.. code-block:: console

  $ aria2c --bt-min-crypto-level=arc4 --bt-require-crypto=true file.torrent


SEE ALSO
--------
Project Web Site: http://aria2.sourceforge.net/

aria2 Wiki: http://sourceforge.net/apps/trac/aria2/wiki

Metalink Homepage: http://www.metalinker.org/

The Metalink Download Description Format: :rfc:`5854`

COPYRIGHT
---------
Copyright (C) 2006, 2011 Tatsuhiro Tsujikawa

This program is free software; you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation; either version 2 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program; if not, write to the Free Software
Foundation, Inc., 51 Franklin Street, Fifth Floor, Boston, MA  02110-1301  USA

In addition, as a special exception, the copyright holders give
permission to link the code of portions of this program with the
OpenSSL library under certain conditions as described in each
individual source file, and distribute linked combinations
including the two.
You must obey the GNU General Public License in all respects
for all of the code used other than OpenSSL.  If you modify
file(s) with this exception, you may extend this exception to your
version of the file(s), but you are not obligated to do so.  If you
do not wish to do so, delete this exception statement from your
version.  If you delete this exception statement from all source
files in the program, then also delete it here.