FROM node:lts-alpine

WORKDIR /app

COPY package*.json yarn.lock /app/

RUN yarn install

COPY . .

EXPOSE 8080

CMD ["yarn", "dev"]