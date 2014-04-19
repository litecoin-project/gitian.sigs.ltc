This repository contains gitian signatures for Litecoin.  It should be updated on each release.
Random members of the public are encouraged to participate in this process in order to better secure the integrity of the Gitian builds.

## Generating Gitian Sigs

 Checkout litecoin, gitian-builder and gitian.sigs.ltc

	mkdir workdir
	cd workdir
	git clone https://github.com/devrandom/gitian-builder.git
	git clone https://github.com/litecoin-project/litecoin.git
	git clone https://github.com/litecoin-project/gitian.sigs.ltc.git

 Create Gitian Base VM Images

	cd ./gitian-builder
        bin/make-base-vm --arch i386
        bin/make-base-vm --arch amd64
        bin/make-base-vm --arch amd64 --suite precise

 Fetch source inputs: (first time, or when dependency versions change)

	mkdir -p inputs; cd inputs/
        wget 'http://miniupnp.free.fr/files/download.php?file=miniupnpc-1.9.20140401.tar.gz' -O 'miniupnpc-1.9.20140401.tar.gz'
        wget 'http://www.openssl.org/source/openssl-1.0.1g.tar.gz'
        wget 'http://download.oracle.com/berkeley-db/db-4.8.30.NC.tar.gz'
        wget 'http://zlib.net/zlib-1.2.8.tar.gz'
        wget 'ftp://ftp.simplesystems.org/pub/libpng/png/src/history/libpng16/libpng-1.6.8.tar.gz'
        wget 'http://fukuchi.org/works/qrencode/qrencode-3.4.3.tar.bz2'
        wget 'http://downloads.sourceforge.net/project/boost/boost/1.55.0/boost_1_55_0.tar.bz2'
	wget 'https://svn.boost.org/trac/boost/raw-attachment/ticket/7262/boost-mingw.patch'
	mv boost-mingw.patch boost-mingw-gas-cross-compile-2013-03-03.patch
        wget 'http://download.qt-project.org/official_releases/qt/4.8/4.8.5/qt-everywhere-opensource-src-4.8.5.tar.gz'
	cd ..

 Build intermediate dependencies and move them into inputs.

	./bin/gbuild ../litecoin/contrib/gitian-descriptors/boost-win32.yml
	mv build/out/inputs/boost-*.zip inputs/
	./bin/gbuild ../litecoin/contrib/gitian-descriptors/deps-win32.yml
	mv build/out/bitcoin*.zip inputs/
	./bin/gbuild ../litecoin/contrib/gitian-descriptors/qt-win32.yml
	mv build/out/qt-*zip inputs/

 Choose your GPG identity and git tag VERSION that you wish to build.

	export SIGNER=(your gitian key, ie bluematt, sipa, etc)
	export VERSION=0.8.7

 Build litecoind and litecoin-qt on Linux32, Linux64, and Win32:

	./bin/gbuild --commit litecoin=v${VERSION} ../litecoin/contrib/gitian-descriptors/gitian.yml
	./bin/gsign --signer $SIGNER --release ${VERSION} --destination ../gitian.sigs.ltc/ ../litecoin/contrib/gitian-descriptors/gitian.yml
	./bin/gbuild --commit litecoin=v${VERSION} ../litecoin/contrib/gitian-descriptors/gitian-win32.yml
	./bin/gsign --signer $SIGNER --release ${VERSION}-win32 --destination ../gitian.sigs.ltc/ ../litecoin/contrib/gitian-descriptors/gitian-win32.yml

 It is customary to rename your directories in gitian.sigs.ltc to match your github username.  Submit a Pull Request against this repo.  If you are uncertain, please see previous commits for examples.
