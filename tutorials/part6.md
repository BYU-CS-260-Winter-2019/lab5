# Part 6: Login

Our next step is to create a login page.

## Login Form

Create a new file called `src/views/Login.vue`. It should have a `template` section:

```
<template>
<div>
  <h1>Login to your account</h1>
  <form @submit.prevent="login" class="pure-form pure-form-aligned">
    <fieldset>
      <p class="pure-form-message-inline">All fields are required.</p>

      <div class="pure-control-group">
        <label for="username">Username</label>
        <input v-model="username" type="text" placeholder="Username">
      </div>

      <div class="pure-control-group">
        <label for="password">Password</label>
        <input v-model="password" type="password" placeholder="Password">
      </div>

      <div class="pure-controls">
        <button type="submit" class="pure-button pure-button-primary">Submit</button>
      </div>
    </fieldset>
  </form>
  <p v-if="error" class="error">{{error}}</p>
</div>
</template>
```

This contains a form to enter a username and password. It is nearly identical
to the registration form.

Next, put the following in the `script` section:

```
<script>
export default {
  name: 'login',
  data() {
    return {
      username: '',
      password: '',
      error: '',
    }
  },
  methods: {
    async login() {
      try {
        this.error = await this.$store.dispatch("login", {
          username: this.username,
          password: this.password
        });
        if (this.error === "")
          this.$router.push('mypage');
      } catch (error) {
        console.log(error);
      }
    }
  }
}
</script>
```

This uses some variables that are bound to the registration form, plus a `login`
method. This method dispatches the `login` action on the store and, if no error
occurs, loads the `MyPage` view.

We'll use the same styles as the registration form:

```
<style scoped>
form {
  border: 1px solid #ccc;
  background-color: #eee;
  border-radius: 4px;
  padding: 20px;
}

.pure-controls {
  display: flex;
}

.pure-controls button {
  margin-left: auto;
}
</style>
```

## Vuex

We need to add some new actions to our Vuex store to handle login and logout.
Start with a `login` action:

```
    async login(context, data) {
      try {
        let response = await axios.post("/api/users/login", data);
        context.commit('setUser', response.data);
        return "";
      } catch (error) {
        return error.response.data.message;
      }
    }
```

This uses `axios` to call the REST API for logging in. We commit the user state
if it is successful, otherwise return an error message.

Next, add the `logout` action.

```
    async logout(context) {
      try {
        await axios.delete("/api/users");
        context.commit('setUser', null);
        return "";
      } catch (error) {
        return error.response.data.message;
      }
    },
```

This uses `axios` to call the REST API for logging out. We delete the state for
the logged in user if it is successful.

Next, add the `getUser` action:

```
    async getUser(context) {
      try {
        let response = await axios.get("/api/users");
        context.commit('setUser', response.data);
        return "";
      } catch (error) {
        return "";
      }
    }
```

This uses `axios` to call the REST API for getting the current user, setting
the state for this user if succssful.

You will note that much of this code is similar to our user authentication
activity.

## My Page

We now need to redesign the `MyPage` view to add a logout button. If the user
is not logged in, then the page displays buttons to register or login.

Modify the template in `src/views/mypage.vue`:

```
<template>
<div>
  <div v-if="user" class="header">
    <div>
      <h2>{{user.name}}</h2>
    </div>
    <div class="button">
      <p><button @click=" logout" class="pure-button pure-button-primary">Logout</button></p>
    </div>
  </div>
  <div v-else>
    <p>If you would like to upload photos, please register for an account or login.</p>
    <router-link to="/register" class="pure-button">Register</router-link> or
    <router-link to="/login" class="pure-button">Login</router-link>
  </div>
</div>
</template>
```

This uses a `v-if` directive to display the user's name if they are logged in,
plus a logout button. Otherwise, it displays directions for registration or
login.

Next, add a `created` section and a `logout` method:

```
  created() {
    this.$store.dispatch("getUser");
  },
  methods: {
    async logout() {
      try {
        this.error = await this.$store.dispatch("logout");
      } catch (error) {
        console.log(error);
      }
    },
  }
```

The `created` section dispatches the `getUser` action on the store, which sets
the `user` variable if the user is logged in. The `logout` method dispatches
the `logout` action on the store.

We also use the following styles:

```
<style scoped>
.pure-button {
  margin: 0px 20px;
}

.header {
  display: flex;
}

.header .button {
  margin-left: 50px;
  order: 2;
}
</style>
```

## Routing

The last step is to hook up the new Login view. Start with adding an import
statement:

```
import Login from './views/Login.vue'
```

Then add the route:

```
   {
      path: '/login',
      name: 'login',
      component: Login
    }
```

## Results

You should now be able to login now. The cookie that is set will keep you logged
in even if you refresh the page.

![login](/screenshots/login.png)

![my page](/screenshots/mypage2.png)

![my page](/screenshots/mypage3.png)

Go to [Part 7](/tutorials/part7.md).
