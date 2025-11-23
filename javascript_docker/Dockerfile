# Use an official Node.js runtime with a newer version as the base image
FROM node:20

# Set the working directory in the container
WORKDIR /app

# --- FIX 1: Install system dependencies and immediately CLEAN UP ---
# This single layer installs dependencies AND cleans the apt cache, 
# significantly reducing final image size.
RUN apt-get update && apt-get install -y \
    build-essential \
    python3 \
    libsqlite3-dev \
    # The backslash allows the command to continue on the next line
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Copy package.json and package-lock.json (or npm-shrinkwrap.json)
COPY package*.json ./

# --- FIX 2: Install dependencies and immediately CLEAN UP npm cache ---
# This reduces image size by clearing the npm cache after installation.
RUN npm install \
    && npm cache clean --force

# Set environment variables
ENV AWS_REGION="us-east-1"

# --- FIX 3: Use the non-root 'node' user for security (if possible) ---
# Running as a non-root user prevents potential attackers from gaining root access 
# if the application is compromised.
USER node

# Copy the rest of the application source code
COPY . .

# Define the command to run the app (ensure 'node' user has permission)
CMD ["node", "alt-text.js"]
