FROM node

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

EXPOSE 80

ENV MONGODB_USERNAME=root
ENV MONGODB_PASSWORD=root
ENV MONGODB_URL=mongodb

CMD [ "npm", "run", "start" ]