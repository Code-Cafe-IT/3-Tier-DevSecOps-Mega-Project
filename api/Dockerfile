FROM node:23.1.0 AS build

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY . . 

FROM node:23.1.0-alpine

COPY --from=build /app/node_modules ./node_modules

COPY --from=build /app .

EXPOSE 5000

CMD [ "node", "app.js" ]
