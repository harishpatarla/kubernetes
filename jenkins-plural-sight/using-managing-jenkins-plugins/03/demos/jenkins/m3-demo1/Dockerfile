FROM sixeyed/psod-jenkins:2.204.4

# .NET Core deps
RUN apk add --no-cache \
    ca-certificates \
    krb5-libs \
    libgcc \
    libintl \
    libssl1.1 \
    libstdc++ \
    zlib

# .NET Core
RUN wget https://dot.net/v1/dotnet-install.sh  && \
    chmod +x dotnet-install.sh && \
    ./dotnet-install.sh

ENV DOTNET_SYSTEM_GLOBALIZATION_INVARIANT="true" \
    PATH="${PATH}:/root/.dotnet"