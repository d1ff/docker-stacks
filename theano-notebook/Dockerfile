FROM jupyter/scipy-notebook
MAINTAINER Vladislav Bortnikov <bortnikov.vladislav@e-sakha.ru>

ENV CUDA_ROOT=/usr/local/cuda/bin

USER root

RUN apt-get update && apt-get install -y cmake && apt-get clean \
     && rm -rf /var/lib/apt/lists/*

USER $NB_USER

RUN cd /tmp && \
    git clone https://github.com/Theano/libgpuarray.git && \
    cd libgpuarray && \
    mkdir Build && \
    cd Build && \
    cmake .. -DCMAKE_BUILD_TYPE=Release && \
    make

USER root

RUN cd /tmp/libgpuarray/Build && make install && \
    ldconfig

USER $NB_USER

RUN cd /tmp/libgpuarray && \
    $CONDA_DIR/bin/python3 setup.py build && \
    $CONDA_DIR/bin/python3 setup.py install

USER root
RUN rm -rf /tmp/libgpuarray
USER $NB_USER

RUN $CONDA_DIR/bin/pip install --upgrade --no-deps git+git://github.com/Theano/Theano.git
# install jupyter vim binding
RUN mkdir -p $(jupyter --data-dir)/nbextensions && \
    cd $(jupyter --data-dir)/nbextensions && \
    git clone https://github.com/lambdalisue/jupyter-vim-binding vim_binding && \
    jupyter nbextension enable vim_binding/vim_binding

RUN conda install --quiet --yes 'nose=1.3*' && conda clean -tipsy
# Set up .theanorc for CUDA
RUN echo "[global]\ndevice=cuda\nfloatX=float32\noptimizer_including=cudnn\n[lib]\ncnmem=0.1\n[nvcc]\nfastmath=True\n[cuda]\nroot=/usr/local/cuda\n[dnn]\nlibrary_path=/usr/local/cuda/lib64\ninclude_path=/usr/local/cuda/include" > /home/$NB_USER/.theanorc
