#${
  "uuid": "<payload-uuid>",
  "next": {
    "always": "https://xumm.app/sign/<payload-uuid>",
    "no_push_msg_received": "https://xumm.app/sign/<payload-uuid>/qr"
  },
  "refs": {
    "qr_png": "https://xumm.app/sign/<payload-uuid>_q.png",
    "qr_matrix": "https://xumm.app/sign/<payload-uuid>_q.json",
    "qr_uri_quality_opts": [
      "m",
      "q",
      "h"
    ],
    "websocket_status": "wss://xumm.app/sign/<payload-uuid>"
  },
  "pushed": true
}JSON-RPC Interface

The headless daemon bitcoind has the JSON-RPC API enabled by default, the GUI bitcoin-qt has it disabled by default. This can be changed with the -server option. In the GUI it is possible to execute RPC methods in the Debug Console Dialog.

Parameter passing

The JSON-RPC server supports both by-position and by-name parameter structures described in the JSON-RPC specification. For extra convenience, to avoid the need to name every parameter value, all RPC methods accept a named parameter called args, which can be set to an array of initial positional values that are combined with named values.

Examples:

# "params": ["mywallet", false, false, "", false, false, true]
bitcoin-cli createwallet mywallet false false "" false false true

# "params": {"wallet_name": "mywallet", "load_on_startup": true}
bitcoin-cli -named createwallet wallet_name=mywallet load_on_startup=true

# "params": {"args": ["mywallet"], "load_on_startup": true}
bitcoin-cli -named createwallet mywallet load_on_startup=true
Versioning

The RPC interface might change from one major version of Bitcoin Core to the next. This makes the RPC interface implicitly versioned on the major version. The version tuple can be retrieved by e.g. the getnetworkinfo RPC in version.

Usually deprecated features can be re-enabled during the grace-period of one major version via the -deprecatedrpc= command line option. The release notes of a new major release come with detailed instructions on what RPC features were deprecated and how to re-enable them temporarily.

Security

The RPC interface allows other programs to control Bitcoin Core, including the ability to spend funds from your wallets, affect consensus verification, read private data, and otherwise perform operations that can cause loss of money, data, or privacy. This section suggests how you should use and configure Bitcoin Core to reduce the risk that its RPC interface will be abused.

Securing the executable: Anyone with physical or remote access to the computer, container, or virtual machine running Bitcoin Core can compromise either the whole program or just the RPC interface. This includes being able to record any passphrases you enter for unlocking your encrypted wallets or changing settings so that your Bitcoin Core program tells you that certain transactions have multiple confirmations even when they aren't part of the best block chain. For this reason, you should not use Bitcoin Core for security sensitive operations on systems you do not exclusively control, such as shared computers or virtual private servers.

Securing local network access: By default, the RPC interface can only be accessed by a client running on the same computer and only after the client provides a valid authentication credential (username and passphrase). Any program on your computer with access to the file system and local network can obtain this level of access. Additionally, other programs on your computer can attempt to provide an RPC interface on the same port as used by Bitcoin Core in order to trick you into revealing your authentication credentials. For this reason, it is important to only use Bitcoin Core for security-sensitive operations on a computer whose other programs you trust.

Securing remote network access: You may optionally allow other computers to remotely control Bitcoin Core by setting the rpcallowip and rpcbind configuration parameters. These settings are only meant for enabling connections over secure private networks or connections that have been otherwise secured (e.g. using a VPN or port forwarding with SSH or stunnel). Do not enable RPC connections over the public Internet. Although Bitcoin Core's RPC interface does use authentication, it does not use encryption, so your login credentials are sent as clear text that can be read by anyone on your network path. Additionally, the RPC interface has not been hardened to withstand arbitrary Internet traffic, so changing the above settings to expose it to the Internet (even using something like a Tor onion service) could expose you to unconsidered vulnerabilities. See bitcoind -help for more information about these settings and other settings described in this document.

Related, if you use Bitcoin Core inside a Docker container, you may need to expose the RPC port to the host system. The default way to do this in Docker also exposes the port to the public Internet. Instead, expose it only on the host system's localhost, for example: -p 127.0.0.1:8332:8332

Secure authentication: By default, Bitcoin Core generates unique login credentials each time it restarts and puts them into a file readable only by the user that started Bitcoin Core, allowing any of that user's RPC clients with read access to the file to login automatically. The file is .cookie in the Bitcoin Core configuration directory, and using these credentials is the preferred RPC authentication method. If you need to generate static login credentials for your programs, you can use the script in the share/rpcauth directory in the Bitcoin Core source tree. As a final fallback, you can directly use manually-chosen rpcuser and rpcpassword configuration parameters---but you must ensure that you choose a strong and unique passphrase (and still don't use insecure networks, as mentioned above).

Secure string handling: The RPC interface does not guarantee any escaping of data beyond what's necessary to encode it as JSON, although it does usually provide serialized data using a hex representation of the bytes. If you use RPC data in your programs or provide its data to other programs, you must ensure any problem strings are properly escaped. For example, the createwallet RPC accepts arguments such as wallet_name which is a string and could be used for a path traversal attack without application level checks. Multiple websites have been manipulated because they displayed decoded hex strings that included HTML <script> tags. For this reason, and others, it is recommended to display all serialized data in hex form only.

RPC consistency guarantees

State that can be queried via RPCs is guaranteed to be at least up-to-date with the chain state immediately prior to the call's execution. However, the state returned by RPCs that reflect the mempool may not be up-to-date with the current mempool state.

Transaction Pool

The mempool state returned via an RPC is consistent with itself and with the chain state at the time of the call. Thus, the mempool state only encompasses transactions that are considered mine-able by the node at the time of the RPC.

The mempool state returned via an RPC reflects all effects of mempool and chain state related RPCs that returned prior to this call.

Wallet

The wallet state returned via an RPC is consistent with itself and with the chain state at the time of the call.

Wallet RPCs will return the latest chain state consistent with prior non-wallet RPCs. The effects of all blocks (and transactions in blocks) at the time of the call is reflected in the state of all wallet transactions. For example, if a block contains transactions that conflicted with mempool transactions, the wallet would reflect the removal of these mempool transactions in the state.

However, the wallet may not be up-to-date with the current state of the mempool or the state of the mempool by an RPC that returned before this RPC. For example, a wallet transaction that was BIP-125-replaced in the mempool prior to this RPC may not yet be reflected as such in this RPC response.

Limitations

There is a known issue in the JSON-RPC interface that can cause a node to crash if too many http connections are being opened at the same time because the system runs out of available file descriptors. To prevent this from happening you might want to increase the number of maximum allowed file descriptors in your system and try to prevent opening too many connections to your JSON-RPC interface at the same time if this is under your control. It is hard to give general advice since this depends on your system but if you make several hundred requests at once you are definitely at risk of encountering this issue.1
2GET {https://yourApp/callback}
    #id_$token=...&
    #access_token=...&
    state=...&
    expires_in=...
3#&$GET https://{yourDomain}/authorize
    ?response_type=id_token token&
    client_id=...&
    redirect_uri=...&
    state=...&
    scope=openid...&
    nonce=...&
    audience=...&
    response_mode=...&
    prompt=none
4# Copyright (c) 2013-2020 The Bitcoin Core developers
# Distributed under the MIT software license, see the accompanying
# file COPYING or http://www.opensource.org/licenses/mit-license.php.

# Pattern rule to print variables, e.g. make print-top_srcdir
print-%: FORCE
	@echo '$*'='$($*)'

ACLOCAL_AMFLAGS = -I build-aux/m4
SUBDIRS = src
if ENABLE_MAN
SUBDIRS += doc/man
endif
.PHONY: deploy FORCE
.INTERMEDIATE: $(COVERAGE_INFO)

export PYTHONPATH

if BUILD_BITCOIN_LIBS
pkgconfigdir = $(libdir)/pkgconfig
pkgconfig_DATA = libbitcoinconsensus.pc
endif

BITCOIND_BIN=$(top_builddir)/src/$(BITCOIN_DAEMON_NAME)$(EXEEXT)
BITCOIN_QT_BIN=$(top_builddir)/src/qt/$(BITCOIN_GUI_NAME)$(EXEEXT)
BITCOIN_TEST_BIN=$(top_builddir)/src/test/$(BITCOIN_TEST_NAME)$(EXEEXT)
BITCOIN_CLI_BIN=$(top_builddir)/src/$(BITCOIN_CLI_NAME)$(EXEEXT)
BITCOIN_TX_BIN=$(top_builddir)/src/$(BITCOIN_TX_NAME)$(EXEEXT)
BITCOIN_UTIL_BIN=$(top_builddir)/src/$(BITCOIN_UTIL_NAME)$(EXEEXT)
BITCOIN_WALLET_BIN=$(top_builddir)/src/$(BITCOIN_WALLET_TOOL_NAME)$(EXEEXT)
BITCOIN_NODE_BIN=$(top_builddir)/src/$(BITCOIN_MP_NODE_NAME)$(EXEEXT)
BITCOIN_GUI_BIN=$(top_builddir)/src/$(BITCOIN_MP_GUI_NAME)$(EXEEXT)
BITCOIN_WIN_INSTALLER=$(PACKAGE)-$(PACKAGE_VERSION)-win64-setup$(EXEEXT)

empty :=
space := $(empty) $(empty)

OSX_APP=Bitcoin-Qt.app
OSX_VOLNAME = $(subst $(space),-,$(PACKAGE_NAME))
OSX_DMG = $(OSX_VOLNAME).dmg
OSX_DEPLOY_SCRIPT=$(top_srcdir)/contrib/macdeploy/macdeployqtplus
OSX_INSTALLER_ICONS=$(top_srcdir)/src/qt/res/icons/bitcoin.icns
OSX_PLIST=$(top_builddir)/share/qt/Info.plist #not installed

DIST_CONTRIB = \
	       $(top_srcdir)/test/sanitizer_suppressions/lsan \
	       $(top_srcdir)/test/sanitizer_suppressions/tsan \
	       $(top_srcdir)/test/sanitizer_suppressions/ubsan \
	       $(top_srcdir)/contrib/linearize/linearize-data.py \
	       $(top_srcdir)/contrib/linearize/linearize-hashes.py \
	       $(top_srcdir)/contrib/signet/miner

DIST_SHARE = \
  $(top_srcdir)/share/genbuild.sh \
  $(top_srcdir)/share/rpcauth

BIN_CHECKS=$(top_srcdir)/contrib/devtools/symbol-check.py \
           $(top_srcdir)/contrib/devtools/security-check.py \
           $(top_srcdir)/contrib/devtools/utils.py

WINDOWS_PACKAGING = $(top_srcdir)/share/pixmaps/bitcoin.ico \
  $(top_srcdir)/share/pixmaps/nsis-header.bmp \
  $(top_srcdir)/share/pixmaps/nsis-wizard.bmp \
  $(top_srcdir)/doc/README_windows.txt

OSX_PACKAGING = $(OSX_DEPLOY_SCRIPT) $(OSX_INSTALLER_ICONS) \
  $(top_srcdir)/contrib/macdeploy/detached-sig-create.sh

COVERAGE_INFO = $(COV_TOOL_WRAPPER) baseline.info \
  test_bitcoin_filtered.info total_coverage.info \
  baseline_filtered.info functional_test.info functional_test_filtered.info \
  test_bitcoin_coverage.info test_bitcoin.info fuzz.info fuzz_filtered.info fuzz_coverage.info

dist-hook:
	-$(GIT) archive --format=tar HEAD -- src/clientversion.cpp | $(AMTAR) -C $(top_distdir) -xf -

if TARGET_WINDOWS
$(BITCOIN_WIN_INSTALLER): all-recursive
	$(MKDIR_P) $(top_builddir)/release
	STRIPPROG="$(STRIP)" $(INSTALL_STRIP_PROGRAM) $(BITCOIND_BIN) $(top_builddir)/release
	STRIPPROG="$(STRIP)" $(INSTALL_STRIP_PROGRAM) $(BITCOIN_QT_BIN) $(top_builddir)/release
	STRIPPROG="$(STRIP)" $(INSTALL_STRIP_PROGRAM) $(BITCOIN_TEST_BIN) $(top_builddir)/release
	STRIPPROG="$(STRIP)" $(INSTALL_STRIP_PROGRAM) $(BITCOIN_CLI_BIN) $(top_builddir)/release
	STRIPPROG="$(STRIP)" $(INSTALL_STRIP_PROGRAM) $(BITCOIN_TX_BIN) $(top_builddir)/release
	STRIPPROG="$(STRIP)" $(INSTALL_STRIP_PROGRAM) $(BITCOIN_WALLET_BIN) $(top_builddir)/release
	STRIPPROG="$(STRIP)" $(INSTALL_STRIP_PROGRAM) $(BITCOIN_UTIL_BIN) $(top_builddir)/release
	@test -f $(MAKENSIS) && echo 'OutFile "$@"' | cat $(top_builddir)/share/setup.nsi - | $(MAKENSIS) -V2 - || \
	  echo error: could not build $@
	@echo built $@

deploy: $(BITCOIN_WIN_INSTALLER)
endif

if TARGET_DARWIN
$(OSX_APP)/Contents/PkgInfo:
	$(MKDIR_P) $(@D)
	@echo "APPL????" > $@

$(OSX_APP)/Contents/Resources/empty.lproj:
	$(MKDIR_P) $(@D)
	@touch $@

$(OSX_APP)/Contents/Info.plist: $(OSX_PLIST)
	$(MKDIR_P) $(@D)
	$(INSTALL_DATA) $< $@

$(OSX_APP)/Contents/Resources/bitcoin.icns: $(OSX_INSTALLER_ICONS)
	$(MKDIR_P) $(@D)
	$(INSTALL_DATA) $< $@

$(OSX_APP)/Contents/MacOS/Bitcoin-Qt: all-recursive
	$(MKDIR_P) $(@D)
	STRIPPROG="$(STRIP)" $(INSTALL_STRIP_PROGRAM)  $(BITCOIN_QT_BIN) $@

$(OSX_APP)/Contents/Resources/Base.lproj/InfoPlist.strings:
	$(MKDIR_P) $(@D)
	echo '{	CFBundleDisplayName = "$(PACKAGE_NAME)"; CFBundleName = "$(PACKAGE_NAME)"; }' > $@

OSX_APP_BUILT=$(OSX_APP)/Contents/PkgInfo $(OSX_APP)/Contents/Resources/empty.lproj \
  $(OSX_APP)/Contents/Resources/bitcoin.icns $(OSX_APP)/Contents/Info.plist \
  $(OSX_APP)/Contents/MacOS/Bitcoin-Qt $(OSX_APP)/Contents/Resources/Base.lproj/InfoPlist.strings

osx_volname:
	echo $(OSX_VOLNAME) >$@

if BUILD_DARWIN
$(OSX_DMG): $(OSX_APP_BUILT) $(OSX_PACKAGING)
	$(PYTHON) $(OSX_DEPLOY_SCRIPT) $(OSX_APP) $(OSX_VOLNAME) -translations-dir=$(QT_TRANSLATION_DIR) -dmg

deploydir: $(OSX_DMG)
else !BUILD_DARWIN
APP_DIST_DIR=$(top_builddir)/dist

$(OSX_DMG): deploydir
	$(XORRISOFS) -D -l -V "$(OSX_VOLNAME)" -no-pad -r -dir-mode 0755 -o $@ $(APP_DIST_DIR) -- $(if $(SOURCE_DATE_EPOCH),-volume_date all_file_dates =$(SOURCE_DATE_EPOCH))

$(APP_DIST_DIR)/$(OSX_APP)/Contents/MacOS/Bitcoin-Qt: $(OSX_APP_BUILT) $(OSX_PACKAGING)
	INSTALL_NAME_TOOL=$(INSTALL_NAME_TOOL) OTOOL=$(OTOOL) STRIP=$(STRIP) $(PYTHON) $(OSX_DEPLOY_SCRIPT) $(OSX_APP) $(OSX_VOLNAME) -translations-dir=$(QT_TRANSLATION_DIR)

deploydir: $(APP_DIST_DIR)/$(OSX_APP)/Contents/MacOS/Bitcoin-Qt
endif !BUILD_DARWIN

deploy: $(OSX_DMG)
endif

$(BITCOIN_QT_BIN): FORCE
	$(MAKE) -C src qt/$(@F)

$(BITCOIND_BIN): FORCE
	$(MAKE) -C src $(@F)

$(BITCOIN_CLI_BIN): FORCE
	$(MAKE) -C src $(@F)

$(BITCOIN_TX_BIN): FORCE
	$(MAKE) -C src $(@F)

$(BITCOIN_UTIL_BIN): FORCE
	$(MAKE) -C src $(@F)

$(BITCOIN_WALLET_BIN): FORCE
	$(MAKE) -C src $(@F)

$(BITCOIN_NODE_BIN): FORCE
	$(MAKE) -C src $(@F)

$(BITCOIN_GUI_BIN): FORCE
	$(MAKE) -C src $(@F)

if USE_LCOV
LCOV_FILTER_PATTERN = \
	-p "/usr/local/" \
	-p "/usr/include/" \
	-p "/usr/lib/" \
	-p "/usr/lib64/" \
	-p "src/leveldb/" \
	-p "src/crc32c/" \
	-p "src/bench/" \
	-p "src/crypto/ctaes" \
	-p "src/minisketch" \
	-p "src/secp256k1" \
	-p "depends"

DIR_FUZZ_SEED_CORPUS ?= qa-assets/fuzz_seed_corpus

$(COV_TOOL_WRAPPER):
	@echo 'exec $(COV_TOOL) "$$@"' > $(COV_TOOL_WRAPPER)
	@chmod +x $(COV_TOOL_WRAPPER)

baseline.info: $(COV_TOOL_WRAPPER)
	$(LCOV) -c -i -d $(abs_builddir)/src -o $@

baseline_filtered.info: baseline.info
	$(abs_builddir)/contrib/filter-lcov.py $(LCOV_FILTER_PATTERN) $< $@
	$(LCOV) -a $@ $(LCOV_OPTS) -o $@

fuzz.info: baseline_filtered.info
	@TIMEOUT=15 test/fuzz/test_runner.py $(DIR_FUZZ_SEED_CORPUS) -l DEBUG
	$(LCOV) -c $(LCOV_OPTS) -d $(abs_builddir)/src --t fuzz-tests -o $@
	$(LCOV) -z $(LCOV_OPTS) -d $(abs_builddir)/src

fuzz_filtered.info: fuzz.info
	$(abs_builddir)/contrib/filter-lcov.py $(LCOV_FILTER_PATTERN) $< $@
	$(LCOV) -a $@ $(LCOV_OPTS) -o $@

test_bitcoin.info: baseline_filtered.info
	$(MAKE) -C src/ check
	$(LCOV) -c $(LCOV_OPTS) -d $(abs_builddir)/src -t test_bitcoin -o $@
	$(LCOV) -z $(LCOV_OPTS) -d $(abs_builddir)/src

test_bitcoin_filtered.info: test_bitcoin.info
	$(abs_builddir)/contrib/filter-lcov.py $(LCOV_FILTER_PATTERN) $< $@
	$(LCOV) -a $@ $(LCOV_OPTS) -o $@

functional_test.info: test_bitcoin_filtered.info
	@TIMEOUT=15 test/functional/test_runner.py $(EXTENDED_FUNCTIONAL_TESTS)
	$(LCOV) -c $(LCOV_OPTS) -d $(abs_builddir)/src --t functional-tests -o $@
	$(LCOV) -z $(LCOV_OPTS) -d $(abs_builddir)/src

functional_test_filtered.info: functional_test.info
	$(abs_builddir)/contrib/filter-lcov.py $(LCOV_FILTER_PATTERN) $< $@
	$(LCOV) -a $@ $(LCOV_OPTS) -o $@

fuzz_coverage.info: fuzz_filtered.info
	$(LCOV) -a $(LCOV_OPTS) baseline_filtered.info -a fuzz_filtered.info -o $@ | $(GREP) "\%" | $(AWK) '{ print substr($$3,2,50) "/" $$5 }' > coverage_percent.txt

test_bitcoin_coverage.info: baseline_filtered.info test_bitcoin_filtered.info
	$(LCOV) -a $(LCOV_OPTS) baseline_filtered.info -a test_bitcoin_filtered.info -o $@

total_coverage.info: test_bitcoin_filtered.info functional_test_filtered.info
	$(LCOV) -a $(LCOV_OPTS) baseline_filtered.info -a test_bitcoin_filtered.info -a functional_test_filtered.info -o $@ | $(GREP) "\%" | $(AWK) '{ print substr($$3,2,50) "/" $$5 }' > coverage_percent.txt

fuzz.coverage/.dirstamp: fuzz_coverage.info
	$(GENHTML) -s $(LCOV_OPTS) $< -o $(@D)
	@touch $@

test_bitcoin.coverage/.dirstamp:  test_bitcoin_coverage.info
	$(GENHTML) -s $(LCOV_OPTS) $< -o $(@D)
	@touch $@

total.coverage/.dirstamp: total_coverage.info
	$(GENHTML) -s $(LCOV_OPTS) $< -o $(@D)
	@touch $@

cov_fuzz: fuzz.coverage/.dirstamp

cov: test_bitcoin.coverage/.dirstamp total.coverage/.dirstamp

endif

dist_noinst_SCRIPTS = autogen.sh

EXTRA_DIST = $(DIST_SHARE) $(DIST_CONTRIB) $(WINDOWS_PACKAGING) $(OSX_PACKAGING) $(BIN_CHECKS)

EXTRA_DIST += \
    test/functional \
    test/fuzz

EXTRA_DIST += \
    test/util/test_runner.py \
    test/util/data/bitcoin-util-test.json \
    test/util/data/blanktxv1.hex \
    test/util/data/blanktxv1.json \
    test/util/data/blanktxv2.hex \
    test/util/data/blanktxv2.json \
    test/util/data/tt-delin1-out.hex \
    test/util/data/tt-delin1-out.json \
    test/util/data/tt-delout1-out.hex \
    test/util/data/tt-delout1-out.json \
    test/util/data/tt-locktime317000-out.hex \
    test/util/data/tt-locktime317000-out.json \
    test/util/data/tx394b54bb.hex \
    test/util/data/txcreate1.hex \
    test/util/data/txcreate1.json \
    test/util/data/txcreate2.hex \
    test/util/data/txcreate2.json \
    test/util/data/txcreatedata1.hex \
    test/util/data/txcreatedata1.json \
    test/util/data/txcreatedata2.hex \
    test/util/data/txcreatedata2.json \
    test/util/data/txcreatedata_seq0.hex \
    test/util/data/txcreatedata_seq0.json \
    test/util/data/txcreatedata_seq1.hex \
    test/util/data/txcreatedata_seq1.json \
    test/util/data/txcreatemultisig1.hex \
    test/util/data/txcreatemultisig1.json \
    test/util/data/txcreatemultisig2.hex \
    test/util/data/txcreatemultisig2.json \
    test/util/data/txcreatemultisig3.hex \
    test/util/data/txcreatemultisig3.json \
    test/util/data/txcreatemultisig4.hex \
    test/util/data/txcreatemultisig4.json \
    test/util/data/txcreatemultisig5.json \
    test/util/data/txcreateoutpubkey1.hex \
    test/util/data/txcreateoutpubkey1.json \
    test/util/data/txcreateoutpubkey2.hex \
    test/util/data/txcreateoutpubkey2.json \
    test/util/data/txcreateoutpubkey3.hex \
    test/util/data/txcreateoutpubkey3.json \
    test/util/data/txcreatescript1.hex \
    test/util/data/txcreatescript1.json \
    test/util/data/txcreatescript2.hex \
    test/util/data/txcreatescript2.json \
    test/util/data/txcreatescript3.hex \
    test/util/data/txcreatescript3.json \
    test/util/data/txcreatescript4.hex \
    test/util/data/txcreatescript4.json \
    test/util/data/txcreatescript5.hex \
    test/util/data/txcreatescript6.hex \
    test/util/data/txcreatesignsegwit1.hex \
    test/util/data/txcreatesignv1.hex \
    test/util/data/txcreatesignv1.json \
    test/util/data/txcreatesignv2.hex \
    test/util/rpcauth-test.py

CLEANFILES = $(OSX_DMG) $(BITCOIN_WIN_INSTALLER)

DISTCHECK_CONFIGURE_FLAGS = --enable-man

doc/doxygen/.stamp: doc/Doxyfile FORCE
	$(MKDIR_P) $(@D)
	$(DOXYGEN) $^
	$(AM_V_at) touch $@

if HAVE_DOXYGEN
docs: doc/doxygen/.stamp
else
docs:
	@echo "error: doxygen not found"
endif

clean-docs:
	rm -rf doc/doxygen

clean-local: clean-docs
	rm -rf coverage_percent.txt test_bitcoin.coverage/ total.coverage/ fuzz.coverage/ test/tmp/ cache/ $(OSX_APP)
	rm -rf test/functional/__pycache__ test/functional/test_framework/__pycache__ test/cache share/rpcauth/__pycache__
	rm -rf osx_volname dist/

test-security-check:
if TARGET_DARWIN
	$(AM_V_at) CC='$(CC)' CFLAGS='$(CFLAGS)' CPPFLAGS='$(CPPFLAGS)' LDFLAGS='$(LDFLAGS)' $(PYTHON) $(top_srcdir)/contrib/devtools/test-security-check.py TestSecurityChecks.test_MACHO
	$(AM_V_at) CC='$(CC)' CFLAGS='$(CFLAGS)' CPPFLAGS='$(CPPFLAGS)' LDFLAGS='$(LDFLAGS)' $(PYTHON) $(top_srcdir)/contrib/devtools/test-symbol-check.py TestSymbolChecks.test_MACHO
endif
if TARGET_WINDOWS
	$(AM_V_at) CC='$(CC)' CFLAGS='$(CFLAGS)' CPPFLAGS='$(CPPFLAGS)' LDFLAGS='$(LDFLAGS)' $(PYTHON) $(top_srcdir)/contrib/devtools/test-security-check.py TestSecurityChecks.test_PE
	$(AM_V_at) CC='$(CC)' CFLAGS='$(CFLAGS)' CPPFLAGS='$(CPPFLAGS)' LDFLAGS='$(LDFLAGS)' $(PYTHON) $(top_srcdir)/contrib/devtools/test-symbol-check.py TestSymbolChecks.test_PE
endif
if TARGET_LINUX
	$(AM_V_at) CC='$(CC)' CFLAGS='$(CFLAGS)' CPPFLAGS='$(CPPFLAGS)' LDFLAGS='$(LDFLAGS)' $(PYTHON) $(top_srcdir)/contrib/devtools/test-security-check.py TestSecurityChecks.test_ELF
	$(AM_V_at) CC='$(CC)' CFLAGS='$(CFLAGS)' CPPFLAGS='$(CPPFLAGS)' LDFLAGS='$(LDFLAGS)' $(PYTHON) $(top_srcdir)/contrib/devtools/test-symbol-check.py TestSymbolChecks.test_ELF
endif
5
6
7
8
9
10
11
12
13
14
15
16
17
18
# This code sample uses the 'requests' library:
# http://docs.python-requests.org
import requests

url = "https://api.trello.com/1/boards/{id}/{field}"

query = {
  'key': 'APIKey',
  'token': 'APIToken'
}

response = requests.request(
   "GET",
   url,
   params=query
)

print(response.text)

# .2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
# This code sample uses the 'requests' library:
# http://docs.python-requests.org
import requests

url = "https://api.trello.com/1/boards/{id}"

query = {
  'key': 'APIKey',
  'token': 'APIToken'
}

response = requests.request(
   "DELETE",
   url,
   params=query
)

print(response.text)
Responses


#$https://developer.atlassian.com/cloud/trello/rest/api-group-boards/#api-boards-id-memberships-get-example
#$https://developer.atlassian.com/cloud/trello/rest/api-group-boards/#api-boards-id-memberships-get-example
$https://trello.com/b/n44AxJQW/foundationscipnet-corporate-board
int bsdChecksumFromFile(FILE *fp) /* The file handle for input data */
{
    int checksum = 0;             /* The checksum mod 2^16. */

    for (int ch = getc(fp); ch != EOF; ch = getc(fp)) {
        checksum = (checksum >> 1) + ((checksum & 1) << 15);
        checksum += ch;
        checksum &= 0xffff;       /* Keep it within bounds. */
    }
    return checksum;
}
Description of the algorithm
As mentioned above, this algorithm computes a checksum by segmenting the data and adding it to an accumulator that is circular right shifted between each summation. To keep the accumulator within return value bounds, bit-masking with 1's is done.
Example: Calculating a 4-bit checksum using 4-bit sized segments (big-endian) 
Input: 101110001110 -> three segments: 1011, 1000, 1110.
Iteration 1:

 segment: 1011        checksum: 0000        bitmask: 1111
a) Apply circular shift to the checksum:
 0000 -> 0000
b) Add checksum and segment together, apply bitmask onto the obtained result:
 0000 + 1011 = 1011 -> 1011 & 1111 = 1011
Iteration 2:

 segment: 1000        checksum: 1011        bitmask: 1111
a) Apply circular shift to the checksum:
 1011 -> 1101
b) Add checksum and segment together, apply bitmask onto the obtained result:
 1101 + 1000 = 10101 -> 10101 & 1111 = 0101
Iteration 3:

 segment: 1110        checksum: 0101        bitmask: 1111
a) Apply circular shift to the checksum:
 0101 -> 1010
b) Add checksum and segment together, apply bitmask onto the obtained result:
 1010 + 1110 = 11000 -> 11000 & 1111 = 1000
Final checksum: 1000
References
 sum(1) — manual pages from GNU coreutils
 sum(1) – FreeBSD General Commands Manual
Sources
official FreeBSD sum source code
official GNU sum manual page
GNU sum source code
Category: Checksum algorithms

#int bsdChecksumFromFile(FILE *fp) /* The file handle for input data */
{
    int checksum = 0;             /* The checksum mod 2^16. */

    for (int ch = getc(fp); ch != EOF; ch = getc(fp)) {
        checksum = (checksum >> 1) + ((checksum & 1) << 15);
        checksum += ch;
        checksum &= 0xffff;       /* Keep it within bounds. */
    }
    return checksum;
}
Description of the algorithm
As mentioned above, this algorithm computes a checksum by segmenting the data and adding it to an accumulator that is circular right shifted between each summation. To keep the accumulator within return value bounds, bit-masking with 1's is done.
Example: Calculating a 4-bit checksum using 4-bit sized segments (big-endian) 
Input: 101110001110 -> three segments: 1011, 1000, 1110.
Iteration 1:

 segment: 1011        checksum: 0000        bitmask: 1111
a) Apply circular shift to the checksum:
 0000 -> 0000
b) Add checksum and segment together, apply bitmask onto the obtained result:
 0000 + 1011 = 1011 -> 1011 & 1111 = 1011
Iteration 2:

 segment: 1000        checksum: 1011        bitmask: 1111
a) Apply circular shift to the checksum:
 1011 -> 1101
b) Add checksum and segment together, apply bitmask onto the obtained result:
 1101 + 1000 = 10101 -> 10101 & 1111 = 0101
Iteration 3:

 segment: 1110        checksum: 0101        bitmask: 1111
a) Apply circular shift to the checksum:
 0101 -> 1010
b) Add checksum and segment together, apply bitmask onto the obtained result:
 1010 + 1110 = 11000 -> 11000 & 1111 = 1000
Final checksum: 1000
References
 sum(1) — manual pages from GNU coreutils
 sum(1) – FreeBSD General Commands Manual
Sources
official FreeBSD sum source code
official GNU sum manual page
GNU sum source code
Category: Checksum algorithms

https://github.com/bitcoin/bips/blob/master/bip-0032/derivation.png?raw=true