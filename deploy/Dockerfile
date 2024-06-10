FROM node

WORKDIR /app

COPY ./artefakty .

RUN npm install -g serve
CMD serve -s build
