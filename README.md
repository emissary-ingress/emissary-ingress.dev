# emissary-ingress.dev

The [emissary-ingress.dev](https://emissary-ingress.dev) website, built using [Hugo][] and hosted on [Netlify][].

## Local build

To build and serve the site, you'll need the latest [LTS release][] of **Node**.
Like Netlify, we use **[nvm][]**, the Node Version Manager, to install and
manage Node versions:

```console
$ nvm install --lts
```

You must also install [hugo](https://gohugo.io/getting-started/installing/) to build the site.

### Setup

 1. Clone this repo.
 2. From a terminal window, change to the cloned repo directory.
 3. Get NPM packages and git submodules, including the the [Docsy][] theme:
    ```console
    $ npm install
    ```

### Build or serve the site

To locally serve the site at [localhost:1313][], run the following command:

```console
$ make run
```

[Docsy]: https://www.docsy.dev
[Hugo]: https://gohugo.io
[localhost:1313]: http://localhost:1313
[LTS release]: https://nodejs.org/en/download/
[Netlify]: https://netlify.com
[nvm]: https://github.com/nvm-sh/nvm/blob/master/README.md#installing-and-updating
