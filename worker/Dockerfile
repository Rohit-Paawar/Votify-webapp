# -----------------------
# Build Stage
# -----------------------
FROM --platform=linux/arm64 mcr.microsoft.com/dotnet/sdk:7.0 AS build

ARG TARGETPLATFORM
ARG TARGETARCH
ARG BUILDPLATFORM
RUN echo "Building on $BUILDPLATFORM for target $TARGETPLATFORM ($TARGETARCH)"

WORKDIR /source

# Copy csproj and restore dependencies
COPY *.csproj ./
RUN dotnet restore

# Copy remaining source and publish
COPY . ./
RUN dotnet publish -c Release -o /app --self-contained false --no-restore

# -----------------------
# Runtime Stage
# -----------------------
FROM --platform=linux/arm64 mcr.microsoft.com/dotnet/runtime:7.0 AS runtime

WORKDIR /app

# Copy published output from build stage
COPY --from=build /app .

# Expose port if needed (optional)
# EXPOSE 80

# Run the worker
ENTRYPOINT ["dotnet", "Worker.dll"]
