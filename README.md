# Developing a Single Page App with FastAPI and Vue.js and Vuetify

### Want to learn how to build this?

If you know what's good for you, check the parent
[post](https://testdriven.io/blog/developing-a-single-page-app-with-fastapi-and-vuejs).

## Want to use this project?

Build the images and spin up the containers:

```sh
$ docker-compose up -d --build
```

Apply the migrations:

```sh
$ docker-compose exec backend aerich upgrade
```

Ensure [http://localhost:5000](http://localhost:5000), [http://localhost:5000/docs](http://localhost:5000/docs), and [http://localhost:8080](http://localhost:8080) work as expected.
