brew install swig

mkdir project

cd project
git clone https://github.com/cmusphinx/pocketsphinx-ios-demo.git
git clone https://github.com/cmusphinx/sphinxbase.git
git clone https://github.com/cmusphinx/pocketsphinx.git

which autoconf
which automake

cd sphinxbase
./autogen.sh
make install

cd pocketsphinx
./autogen.sh
make install

cp build_iphone.sh ../sphinxbase/build_iphone.sh
cp build_iphone.sh ../pocketsphinx/build_iphone.sh

make distclean
./build_iphone.sh
