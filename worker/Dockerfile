# -----------------------
# Build Stage
# -----------------------
FROM --platform=linux/amd64 mcr.microsoft.com/dotnet/sdk:7.0 AS build

# Display platform info (optional, for debugging)
ARG TARGETPLATFORM
ARG TARGETARCH
ARG BUILDPLATFORM
RUN echo "Building on $BUILDPLATFORM for target $TARGETPLATFORM ($TARGETARCH)"

# Set working directory
WORKDIR /source

# Copy project files and restore dependencies
COPY *.csproj ./
RUN dotnet restore

# Copy the rest of the source code
COPY . ./

# Publish the app
RUN dotnet publish -c Release -o /app --self-contained false --no-restore

# -----------------------
# Runtime Stage
# -----------------------
FROM --platform=linux/amd64 mcr.microsoft.com/dotnet/runtime:7.0 AS runtime

WORKDIR /app

# Copy published app from build stage
COPY --from=build /app .

# Set entrypoint
ENTRYPOINT ["dotnet", "Worker.dll"]