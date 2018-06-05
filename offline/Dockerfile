FROM javierggt/externals

LABEL maintainer="Javier Gonzalez <jgonzalez@icecube.wisc.edu>"

# Set when building on Travis so that certain long-running build steps can
# be skipped to shorten build time.
ARG TEST_ONLY_BUILD

USER root

# create a directory for icetray
RUN mkdir /opt/icetray && chown $NB_UID:$NB_GID /opt/icetray

ADD ipython /home/$NB_USER/.ipython
ENV STUPID_FILENAME "/home/$NB_USER/.config/The IceCube Collaboration"
ADD icecube_config ${STUPID_FILENAME}
ENV STUPID_FILENAME "/home/$NB_USER/.config/The IceCube Collaboration"
RUN chown $NB_USER "${STUPID_FILENAME}"

USER $NB_UID
WORKDIR /opt/icetray

# check out icetray/offline-software/trunk
ARG password
RUN mkdir -p /opt/icetray/offline-software && mkdir /opt/icetray/offline-software/build && \
    svn co http://code.icecube.wisc.edu/svn/meta-projects/offline-software/trunk \
           /opt/icetray/offline-software/src \
           --username=icecube --password=$password --no-auth-cache

# prepare icetray build
WORKDIR /opt/icetray/offline-software/build
RUN cmake /opt/icetray/offline-software/src \
      -DCMAKE_BUILD_TYPE=Release \
      -DCMAKE_SHARED_LINKER_FLAGS="-Wl,-rpath,/opt/conda/lib" \
      -DCMAKE_MODULE_LINKER_FLAGS="-Wl,-rpath,/opt/conda/lib" \
      -DCMAKE_EXE_LINKER_FLAGS="-Wl,-rpath,/opt/conda/lib" \
      -DCMAKE_CXX_FLAGS="-D_GLIBCXX_USE_CXX11_ABI=0"

# build icetray
RUN make -j`nproc`

# build icetray test binaries
# RUN make test-bins -j`nproc`

# change back to the home directory
WORKDIR /home/$NB_USER

# Configure container startup (with icetray)
ENTRYPOINT ["tini", "--", "/opt/icetray/offline-software/build/env-shell.sh"]
CMD ["steamshovel"]
