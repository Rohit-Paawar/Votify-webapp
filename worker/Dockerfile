# -----------------------
# Build Stage
# -----------------------
FROM mcr.microsoft.com/dotnet/sdk:7.0 AS build

# Print build info for debugging
ARG TARGETPLATFORM
ARG TARGETARCH
ARG BUILDPLATFORM
RUN echo "Building on $BUILDPLATFORM for target $TARGETPLATFORM ($TARGETARCH)"

WORKDIR /source

# Copy and restore dependencies
COPY *.csproj ./
RUN dotnet restore

# Copy the remaining source code and build
COPY . ./
RUN dotnet publish -c Release -o /app --self-contained false --no-restore

# -----------------------
# Runtime Stage
# -----------------------
FROM mcr.microsoft.com/dotnet/runtime:7.0 AS runtime

WORKDIR /app

# Copy the published output from the build stage
COPY --from=build /app .

# Expose ports if necessary (optional)
# EXPOSE 80

# Run the worker
ENTRYPOINT ["dotnet", "Worker.dll"]
