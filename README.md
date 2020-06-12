# Code test for Github search and report

### Features

 - Search repos through topic or programming languages.
 - Generate a PDF report based on the search result ( for Admin purposes )

### Test unit

Link :  [here](https://github-search-report.herokuapp.com/)

### Instruction

- Type any repository that you want to search.
- Type topic / languages that is related to that said repository
- Click the button which is based on topic / languages that you input just now
- After the information pops up, there's a button where you can generate a PDF file and you can click it to get the report from there.

### Compromises/Shotcuts

- Basically due to time limit given, a simple table is used to reach the requirement given.
- The design of the content inside of PDF is very basic at the moment which U think can be improved overtime. 
- Nuxt.js and vuetify default theme has been used instead for faster deployment.
- Instead of Lambda ( Amazon ), I used heroku to make the deployment online even faster due to easier configuration overall.

## Build Setup

```bash
# install dependencies
$ npm install

# serve with hot reload at localhost:3000
$ npm run dev

# build for production and launch server
$ npm run build
$ npm run start

# generate static project
$ npm run generate
```

For detailed explanation on how things work, check out [Nuxt.js docs](https://nuxtjs.org).
