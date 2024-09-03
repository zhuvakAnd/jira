FROM node:12.22.10-alpine AS build

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

RUN npm run build

FROM nginx:stable-alpine

COPY --from=build /app/build /var/www/html/build/

COPY /home/andrii/nginx.conf /etc/nginx/nginx.conf
COPY /home/andrii/site.conf  /etc/nginx/conf.d/site.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
