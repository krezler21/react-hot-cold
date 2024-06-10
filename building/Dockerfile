FROM node as react-hot-cold

ARG TOKEN

RUN apt-get update
RUN apt-get upgrade -y 
RUN apt-get install -y git

WORKDIR /react-hot-cold

COPY . .

RUN npm i
RUN npm run build

CMD ls build
