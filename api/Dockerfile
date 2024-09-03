FROM node:12.22.10-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

RUN npm run build

RUN npm install -g pm2

EXPOSE 3000

CMD ["npm", "run", "start:production"]
