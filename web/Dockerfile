FROM node:16

# Create app directory (mount volume to this directory in docker compose)
WORKDIR /app

ADD package.json ./

RUN npm install

COPY . .

# Start de node server
CMD [ "npm", "run", "dev" ]