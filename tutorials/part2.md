# Part 2: Navigation

We will setup a navigation system using [Pure.css](https://purecss.io/).
The `Pure.css` styles provide a simple way to create a grid, forms, buttons,
and menus without having to load JavaScript. If you want a responsive menu
system, see their website for a very small pure JavaScript file you can include.

Start by editing `public/index.html`. Add a link to the `Pure.css`
styles and change the title of the site:

```
  <link rel="stylesheet" href="https://unpkg.com/purecss@1.0.0/build/pure-min.css" integrity="sha384-nn4HPE8lTHyVtfCBi5yW9d20FjT8BJwUXyWZT9InLYax14RDjBj46LmSztkmNP9w" crossorigin="anonymous">
  <title>Photo Share</title>
```

At the bottom of the body, add the [font awesome](https://fontawesome.com/?from=io) library:

```
  <script src="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/5.8.1/js/all.js" integrity="sha256-FfgLgtUyCun3AtxuU4iXuVNSbOzW6p1ozrdO0PlV6qA=" crossorigin="anonymous"></script>
</body>
```

Now let's create a menu.

In `src/App.vue`, modify the `template` section so it looks like this:

```
<template>
<div id="app">
  <div class="pure-menu">
    <span class="pure-menu-heading">Photo Bomb</span>
    <ul class="pure-menu-list">
      <li class="pure-menu-item">
        <router-link to="/" class="pure-menu-link">Home</router-link>
      </li>
      <li class="pure-menu-item">
        <router-link to="/mypage" class="pure-menu-link">My Page</router-link>
      </li>
    </ul>
  </div>
  <div class="content">
    <router-view />
  </div>
</div>
</template>
```

The menu is contained in a `ul` element, with classes from `Pure.css`. We
use `router-link` to create the links.

Now add the `style` section:

```
<style>
/* https://color.adobe.com/Ventana-Azul-color-theme-2159606/?showPublished=true */
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  font-size: 18px;
  display: flex;
  min-height: 100%;
}

.pure-menu {
  /* To limit the menu width to the content of the menu: */
  /* display: inline-block; */
  /* Or set the width explicitly: */
  text-align: left;
  background: #000;
}

.pure-menu-heading {
  color: #fff;
  font-size: 1.2em;
  padding: 20px 20px;
  background-color: #F2385A;
  margin-bottom: 10px;
}

.pure-menu-link {
  color: #fff;
  padding: 10px 20px;
  font-weight: 800;
}

.pure-menu-link:hover {
  background: #333;
}

.pure-menu-link.router-link-exact-active {
  background: #fff;
  color: #F2385A;
}

.content {
  margin: 50px 100px;
}

html {
  height: 100%;
  box-sizing: border-box;
}

body {
  height: 100%;
}

*,
*:before,
*:after {
  box-sizing: inherit;
  /* https://css-tricks.com/box-sizing/ */
}

.error {
  color: #F2385A;
}

.pure-button-primary {
  background-color: #36B1BF;
}

/* Modals */
.modal-mask {
  position: fixed;
  z-index: 9998;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, .5);
  display: table;
  transition: opacity .3s ease;
}

.modal-wrapper {
  display: table-cell;
  vertical-align: middle;
}

.modal-container {
  width: 500px;
  margin: 0px auto;
  padding: 20px 30px;
  background-color: #fff;
  border-radius: 2px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, .33);
  transition: all .3s ease;
  font-family: Helvetica, Arial, sans-serif;
}

.modal-header h1 {
  margin-bottom: 30px;
  font-size: 1.5em;
}

.modal-body {
  margin: 0;
}

.modal-body input {
  margin-bottom: 20px;
  height: 30px;
}

.modal-footer {
  margin-top: 20px;
  display: flex;
  justify-content: space-between;
}

.modal-default-button {
  float: right;
}

/*
  * The following styles are auto-applied to elements with
  * transition="modal" when their visibility is toggled
  * by Vue.js.
  *
  * You can easily play with the modal transition by editing
  * these styles.
  */
.modal-enter {
  opacity: 0;
}

.modal-leave-active {
  opacity: 0;
}

.modal-enter .modal-container,
.modal-leave-active .modal-container {
  -webkit-transform: scale(1.1);
  transform: scale(1.1);
}
</style>
```

## Results

Run the front end with:

```
npm run serve
```

You should see the menu for this site:

![menu](/screenshots/menu.png)

Go to [Part 3](/tutorials/part3.md).
