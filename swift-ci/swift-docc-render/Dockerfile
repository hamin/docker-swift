FROM node:14.17.4

ARG SWIFT_DOCC_RENDER_BRANCH=main

ARG BUILD_USER_ID=998

RUN groupadd -g ${BUILD_USER_ID} build-user && \
    useradd -m -r -u ${BUILD_USER_ID} -g build-user build-user

USER build-user

WORKDIR /home/build-user

RUN git clone https://github.com/apple/swift-docc-render.git
RUN cd swift-docc-render && git checkout $SWIFT_DOCC_RENDER_BRANCH
RUN /home/build-user/swift-docc-render/build-script-helper.py build --verbose
RUN /home/build-user/swift-docc-render/build-script-helper.py test --verbose
