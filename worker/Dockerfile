# -----------------------
# Build Stage
# -----------------------
FROM --platform=linux/arm64 mcr.microsoft.com/dotnet/sdk:7.0 AS build

ARG TARGETPLATFORM
ARG TARGETARCH
ARG BUILDPLATFORM
RUN echo "Building on $BUILDPLATFORM for target $TARGETPLATFORM ($TARGETARCH)"

WORKDIR /source

COPY *.csproj ./
RUN dotnet restore

COPY . ./
RUN dotnet publish -c Release -o /app --self-contained false --no-restore

# -----------------------
# Runtime Stage
# -----------------------
FROM --platform=linux/arm64 mcr.microsoft.com/dotnet/runtime:7.0 AS runtime

WORKDIR /app
COPY --from=build /app .
ENTRYPOINT ["dotnet", "Worker.dll"]
