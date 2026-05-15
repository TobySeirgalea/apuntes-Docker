FROM node:22-alpine
WORKDIR /app
COPY ./package.json .
RUN yarn install --production
COPY . . 
#Copiamos todo de /app en nuestra carpeta a la WORKDIR del container
CMD ["node", "./src/index.js"]
