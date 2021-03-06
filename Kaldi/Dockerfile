FROM ubuntu:16.04

ARG OPENFST_VERSION=1.6.1
ARG NUM_BUILD_CORES=8
ENV OPENFST_VERSION ${OPENFST_VERSION}
ENV NUM_BUILD_CORES ${NUM_BUILD_CORES}

RUN ln -s -f bash /bin/sh
RUN apt-get update && apt-get install -y \
	gcc g++ checkinstall python-dev libz-dev \
	wget g++ make automake autoconf bzip2 libtool subversion \
	python3 libatlas3-base zlib1g-dev sox graphviz git \
	swig python-setuptools gawk libsox-fmt-mp3
	
RUN easy_install pip
RUN pip install numpy jupyter

# Build OpenFST
RUN wget http://www.openfst.org/twiki/pub/FST/FstDownload/openfst-${OPENFST_VERSION}.tar.gz
RUN tar -xvf openfst-${OPENFST_VERSION}.tar.gz -C /
RUN mkdir -p /openfst-deb
WORKDIR /openfst-${OPENFST_VERSION}
RUN ./configure \
        --enable-static=yes \
        --enable-shared=no \
        --with-pic=yes \
        --enable-far \
        --enable-python
RUN make -j${NUM_BUILD_CORES}
RUN checkinstall -Dy --install=yes

# Install Kaldi
RUN git clone https://github.com/kaldi-asr/kaldi /kaldi --origin golden
RUN cd /kaldi/tools/; make; cd ../src; ./configure --shared \
	&& make clean && make depend -j 20 \
	&& make -j 20
# RUN cd /kaldi/src && ./configure --shared && make clean && make depend -j 20 \
# 	&& make -j 20
RUN cd /kaldi/tools && rm *.tar.gz *.bz2

# Install SRILM
COPY srilm.tgz /kaldi/tools
RUN cd /kaldi/tools && ./install_srilm.sh 

RUN cd /kaldi/tools && ./extras/install_sequitur.sh \
	&& mv /kaldi/tools/sequitur-g2p /kaldi/tools/sequitur

ENV KALDI_ROOT=/kaldi
ENV PATH=$KALDI_ROOT/utils/:$KALDI_ROOT/src/bin:$KALDI_ROOT/tools/openfst/bin:$KALDI_ROOT/src/fstbin/:$KALDI_ROOT/src/gmmbin/:$KALDI_ROOT/src/featbin/:$KALDI_ROOT/src/lm/:$KALDI_ROOT/src/sgmmbin/:$KALDI_ROOT/src/sgmm2bin/:$KALDI_ROOT/src/fgmmbin/:$KALDI_ROOT/src/latbin/:$PWD:$PATH
ENV SEQUITUR="/kaldi/tools/sequitur-g2p"
ENV PATH="$PATH:${SEQUITUR}/bin"
ENV PYTHONPATH="${PYTHONPATH:-}:$SEQUITUR/./lib/python2.7/site-packages"
ENV LIBLBFGS=/kaldi/tools/liblbfgs-1.10
ENV LD_LIBRARY_PATH=${LD_LIBRARY_PATH:-}:${LIBLBFGS}/lib/.libs
ENV SRILM=/kaldi/tools/srilm
ENV PATH=${PATH}:${SRILM}/bin:${SRILM}/bin/i686-m64

COPY cmd.sh /kaldi/egs/commonvoice/s5/cmd.sh

WORKDIR /kaldi/egs/commonvoice/s5
CMD ["jupyter", "notebook", "--ip=0.0.0.0", "--allow-root"]