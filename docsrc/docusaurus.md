# Docusaurus

## Installation nodejs on ubuntu

https://deb.nodesource.com/

```
wget --no-check-certificate https://deb.nodesource.com/setup_20.x
sudo bash ./setup_20.x
sudo apt install nodejs
node -v
```

## Installation of docusaurus

```
npx create-docusaurus@latest my-website classic
Need to install the following packages:
create-docusaurus@3.5.2
Ok to proceed? (y) y
```

## Getting started

```
[INFO] Inside that directory, you can run several commands:

  `npm start`
    Starts the development server.

  `npm run build`
    Bundles your website into static files for production.

  `npm run serve`
    Serves the built website locally.

  `npm run deploy`
    Publishes the website to GitHub pages.

We recommend that you begin by typing:

  `cd my-website`
  `npm start`

Happy building awesome websites!
```

Client starten mit Adresse: `localhost:3000`

Project structure: 

```
my-website
├── blog
│   ├── 2019-05-28-hola.md
│   ├── 2019-05-29-hello-world.md
│   └── 2020-05-30-welcome.md
├── docs
│   ├── doc1.md
│   ├── doc2.md
│   ├── doc3.md
│   └── mdx.md
├── src
│   ├── css
│   │   └── custom.css
│   └── pages
│       ├── styles.module.css
│       └── index.js
├── static
│   └── img
├── docusaurus.config.js
├── package.json
├── README.md
├── sidebars.js
└── yarn.lock
``` 

# Github Pages Integration

## Config

vi docusaurus.config.js
```
export default {
  // ...
  url: 'https://endiliey.github.io', // Your website URL
  baseUrl: '/',
  projectName: 'endiliey.github.io',
  organizationName: 'endiliey',
  trailingSlash: false,
  // ...
};
```

## Github

* In github kann man die sourcen im main branch haben
* per default wird mit `npm run deploy` in ein gh-pages branch gepuscht
* Im Setting des Repos ist das anzugeben: 
  setting -> pages -> branch: gh-pages -> root -> save


## Deploymentprozess

```
npm run build
npm run serve
npm run deploy
```



