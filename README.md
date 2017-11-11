# DataMoviz 🎥

DataMoviz is a data visualization website about movies...

## Installation

```bash
cd client/
npm install
cd ../server
npm install
```

You also need a populated MongoDB database, see the [server](https://github.com/quentinus95/datamoviz/tree/master/server) readme.

## Running

```bash
cd client/
npm start &
cd ../server
npm start &
```

Access to the server at the following url: http://127.0.0.1:8080.

## Deploying

Compile the client:

```bash
cd client/
npm run deploy
```

You can export the `build` folder to your server.

## Licence

MIT
