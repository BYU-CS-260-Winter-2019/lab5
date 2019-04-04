# Part 7: Uploading Photos

We are now ready to have users upload photos.

# Back End

On the back end, we're going to use a library called
[multer](https://github.com/BYU-CS-260-Winter-2019/lab4/blob/master/tutorials)
to upload images.

First, create a file in `server/photos.js` that has the following:

```
const mongoose = require('mongoose');
const express = require("express");
const router = express.Router();
const auth = require("./auth.js");

// Configure multer so that it will upload to '/public/images'
const multer = require('multer')
const upload = multer({
  dest: '../public/images/',
  limits: {
    fileSize: 10000000
  }
});
```

This imports the necessary libraries and configures multer. Images will be
stored in `../public/images`.

Next, import the User model:

```
const users = require("./users.js");
const User = users.model;
```

Then, define a Photo schema and model:

```
const photoSchema = new mongoose.Schema({
  user: {
    type: mongoose.Schema.ObjectId,
    ref: 'User'
  },
  path: String,
  title: String,
  description: String,
  created: {
    type: Date,
    default: Date.now
  },
});

const Photo = mongoose.model('Photo', photoSchema);
```

Notice that the Photo schema uses a reference to a User. This is what we'll use
to ensure that photos belong to a particular user. We can use this reference to
look up all the photos belonging to a user or ensure that a user can only
delete their own photos.

Notice also that both the `user` and `created` fields in the document use an
extended syntax to include both the type and other fields -- a reference for
the user and a default for the date the photo was created.

Now we need to define the API. First, uploading a photo:

```
// upload photo
router.post("/", auth.verifyToken, User.verify, upload.single('photo'), async (req, res) => {
  // check parameters
  if (!req.file)
    return res.status(400).send({
      message: "Must upload a file."
    });

  const photo = new Photo({
    user: req.user,
    path: "/images/" + req.file.filename,
    title: req.body.title,
    description: req.body.description,
  });
  try {
    await photo.save();
    return res.sendStatus(200);
  } catch (error) {
    console.log(error);
    return res.sendStatus(500);
  }
});
```

We have to verify the token _and_ the user's account. Assuming the user has a
valid token and account, we let them upload a file. The request includes all of
the metadata for the photo, so we can create a photo document and save this to
the database. We don't need to set the date since it will default to the current
date.

Next, create a method to handle getting all the photos for a given user:

```
// get my photos
router.get("/", auth.verifyToken, User.verify, async (req, res) => {
  // return photos
  try {
    let photos = await Photo.find({
      user: req.user
    }).sort({
      created: -1
    });
    return res.send(photos);
  } catch (error) {
    console.log(error);
    return res.sendStatus(500);
  }
});
```

We again verify the token _and_ the user's account. Verifying the user's account
adds the `req.user` object. Then we find all the photos that belong to that user
by specifying to `find` that we want all photos with `{user: req.user}`. We sort
this list by the date they were created in descending order (most recent photo
first). We return this list in the response.

Finally, export the model and the routes:

```
module.exports = {
  model: Photo,
  routes: router,
}
```

The last step for the back end is to import the photos module in `server.js`:

```
const photos = require("./photos.js");
app.use("/api/photos", photos.routes);
```

# VueX

Now that have an API for uploading and getting photos, we need to add state,
mutations, and actions to Vuex for handling photos. In `src/store.js`, add
state for an array of photos:

```
  state: {
    user: null,
    photos: []
  },
```

Next, add the mutation for setting a new array of photos:

```
    setPhotos(state, photos) {
      state.photos = photos;
    }
```

Now add an action to upload a photo:

```
    async upload(context, data) {
      try {
        await axios.post('/api/photos', data);
        return "";
      } catch (error) {
        return error.response.data.message;
      }
    },
```

This uses `axios` to send a POST request for the `api/photos` endpoint.

Then add an action to get the photos for the current user:

```
    async getMyPhotos(context) {
      try {
        let response = await axios.get("/api/photos");
        context.commit('setPhotos', response.data);
        return "";
      } catch (error) {
        return "";
      }
    },
```

This uses `axios` to send a GET request to the `api/photos` endpoint.

## Uploader

Now we need to modify the `MyPage` view to upload and display photos for a user.
Instead of putting everything there, we are going to use several child
components. A component is a reusable piece of the application.

Create a file in `src/components/Uploader.vue` and add this to the `template`
section:

```
<template>
<transition v-if="show" name="modal">
  <div class="modal-mask">
    <div class="modal-wrapper">
      <div class="modal-container">
        <div class="modal-header">
          <h1 class="modal-title">Upload</h1>
        </div>
        <div class="modal-body">
          <p v-if="error" class="error">{{error}}</p>
          <form @submit.prevent="upload">
            <input v-model="title" placeholder="Title">
            <p></p>
            <textarea v-model="description" placeholder="Description"></textarea>
            <p></p>
            <input type="file" name="photo" @change="fileChanged">
            <p></p>
            <button type="button" @click="close" class="pure-button">Close</button>
            <button type="submit" class="pure-button pure-button-secondary">Upload</button>
          </form>
        </div>
      </div>
    </div>
  </div>
</transition>
</template>
```

This uses a transition to pop up a modal dialog for uploading a photo. We use a
regular form with Vue events to handle form submission and closing the form.

For the script section of this component, add the following:

```
<script>
export default {
  name: 'Uploader',
  props: {
    show: Boolean,
  },
  data() {
    return {
      title: '',
      description: '',
      file: null,
      error: '',
    }
  },
</script>
```

The main thing that is different here is that we define `props` for the component.
This provides a way for a parent view or component to pass its child component
data. We use props just like data properties or computed properties. Notice
that props are typed and Vue applies type checking to these.

Add the following methods to the `script` section:

```
methods: {
  fileChanged(event) {
    this.file = event.target.files[0]
  },
  close() {
    this.$emit('escape');
  },
```

The `fileChanged` event keeps track of the currently selected file. The close
method emits an event called `escape`. This is how child components can call
methods on their parent components.

Add one more method to the `script` section:

```
    async upload() {
      try {
        const formData = new FormData();
        formData.append('photo', this.file, this.file.name);
        formData.append('title', this.title);
        formData.append('description', this.description);
        this.error = await this.$store.dispatch("upload", formData);
        if (!this.error) {
          this.title = '';
          this.description = '';
          this.file = null;
          this.$emit('uploadFinished');
        }
      } catch (error) {
        console.log(error);
      }
```

This handles the file upload. We must use a `FormData` object to send the
file to the server. We can append additional metadata such as a title and
description. These are accessed as `req.body.title` and `req.body.description`
in the server.

If no error occurs, we close and reset the form and emit an `uploadFinished`
event to the parent.

Finally, add a `style` section to this component:

```
<style scoped>
input {
  width: 100%;
}

textarea {
  width: 100%;
  height: 100px
}

.pure-button-secondary {
  float: right;
}
</style>
```

## My Page

We need to use this component on the `MyPage` view. Start by modifying the
header in `src/views/MyPage.vue`:

```
<template>
<div>
  <div v-if="user">
    <div class="header">
      <div>
        <h1>{{user.name}}</h1>
      </div>
      <div>
        <p>
          <a @click="toggleUpload"><i class="far fa-image"></i></a>
          <a href="#" @click="logout"><i class="fas fa-sign-out-alt"></i></a>
        </p>
      </div>
    </div>
    <uploader :show="show" @escape="escape" @uploadFinished="uploadFinished" />
  </div>
  <div v-else>
    <p>If you would like to upload photos, please register for an account or login.</p>
    <router-link to="/register" class="pure-button">Register</router-link> or
    <router-link to="/login" class="pure-button">Login</router-link>
  </div>
</div>
</template>


```

We setup to links -- one for uploading photos and one for logging out. We use
icons from the `Font Awesome` library instead of buttons.

We also add the `Uploader` component right after the header. This will insert
the component for the Uploader at this point in the DOM. To pass data to the
`Uploader`, the `MyPage` view binds `show` to a local variable. This gets
passed as the `show` property in the component. To setup events that the
child component -- `Uploader` -- can call on the parent view -- `MyPage`,
we bind several event handlers.

Next, we setup the `script` component of `MyPage` to import and set up the
component:

```
<script>
import Uploader from '@/components/Uploader.vue'

export default {
  name: 'mypage',
  components: {
    Uploader,
  },
  data() {
    return {
      show: false,
    }
  },
```

Notice that the component in JavaScript is called `Uploader` and in HTML it
is called `uploader`.

We also create a data property called `show` that is bound as a property to
the `Uploader` component.

Add several methods to toggle the uploader:

```
    escape() {
      this.show = false;
    },
    toggleUpload() {
      this.show = true;
    },
```

Modify the style for this view:

```
<style scoped>
.header {
  display: flex;
}

.header a {
  padding-left: 50px;
  color: #222;
  font-size: 2em;
}

.header svg {
  margin-top: 12px;
}
</style>
```

## Image Gallery

We also need to show the photos for a user on their page. To do this, create
a file called `src/components/ImageGallery.vue`. This component has the
following `template` section:

```
<template>
<div>
  <div class="image" v-for="photo in photos" v-bind:key="photo._id">
    <img :src="photo.path" />
    <p class="photoTitle">{{photo.title}}</p>
    <p class="photoDate">{{formatDate(photo.created)}}</p>
    <p>{{photo.description}}</p>
  </div>
</div>
</template>
```

This creates a set of divs to hold the photos, which we'll display vertically.

Next add the following `script` section to this component:

```
<script>
import moment from 'moment';

export default {
  name: 'ImageGallery',
  props: {
    photos: Array
  },
}
</script>
```

This creates a prop named `photos`. The parent view will pass this component
a list of photos (either the user's photos or all photos) and then display them
on the screen.

Finally, add this method to the `script` section:

```
  methods: {
    formatDate(date) {
      if (moment(date).diff(Date.now(), 'days') < 15)
        return moment(date).fromNow();
      else
        return moment(date).format('d MMMM YYYY');
    }
  }
```

This creates a `formatDate` method that we use on the template to display the
date, using the Moment library. If the date is less than 15 days ago we display
the date using fromNow (1 day ago) otherwise we display a day, month, and year.

For this to work, we need to install the Moment library:

```
npm install moment
```

Here is the `style` section for this component:

```
<style scoped>
.photoTitle {
  margin: 0px;
  font-size: 1.2em;
}

.photoDate {
  margin: 0px;
  font-size: 0.9em;
  font-weight: normal;
}

p {
  margin: 0px;
}

.image {
  margin: 0 0 1.5em;
  display: inline-block;
  width: 100%;
}

.image img {
  max-width: 600px;
  max-height: 600px;
  image-orientation: from-image;
}
</style>
```

## My Page

To add this component to `MyPage`, add the component in the `template` section:

```
    <uploader :show="show" @escape="escape" @uploadFinished="uploadFinished" />
    <image-gallery :photos="photos" />
```

We pass the list of photos as a property.

In the script section import the component:

```
import Uploader from '@/components/Uploader.vue'
import ImageGallery from '@/components/ImageGallery.vue'
```

Add it to the component listing:

```
  components: {
    Uploader,
    ImageGallery
```

Define a computed property to get the list of photos from the store:

```
    photos() {
      return this.$store.state.photos;
    }
```

Dispatch the `getMyPhotos` action from Vuex when the view is created:

```
  async created() {
    await this.$store.dispatch("getUser");
    await this.$store.dispatch("getMyPhotos");
  },
```

Define the `uploadFinished` method, which is called when the Uploader emits
the `uploadFinished` event:

```
    async uploadFinished() {
      this.show = false;
      try {
        this.error = await this.$store.dispatch("getMyPhotos");
      } catch (error) {
        console.log(error);
      }
    },
```

This dispatches the `getMyPhotos` action from Vuex.

## Escape Key Handling

To have the system let the escape key close the uploader, define a third
component for this in `src/components/EscapeEvent.vue`. This component should
have:

```
<template>
<div></div>
</template>

<script>
export default {
  name: 'EscapeEvent',
  created() {
    window.addEventListener('keyup', this.handler);
  },
  beforeDestroy() {
    window.removeEventListener('keyup', this.handler);
  },
  methods: {
    handler(event) {
      if (event.keyCode == 27)
        this.$emit('escape');
    }
  }
}
</script>

<style scoped>
div {
  display: none;
}
</style>
```

This uses JavaScript to attach an event handler to the window to listen for
`keyup` JavaScript events and, when one occurs, emit the `escape` Vue event.

Back in `src/views/MyPage.vue`, add this component to the `template` section:

```
<escape-event @escape="escape"></escape-event>
<uploader :show="show" @escape="escape" @upload-finished="uploadFinished" />
<image-gallery :photos="photos" />
```

Import it into the `script` section:

```
import EscapeEvent from '@/components/EscapeEvent.vue'
import Uploader from '@/components/Uploader.vue'
import ImageGallery from '@/components/ImageGallery.vue'
```

And add it to the list of components in the `script` section:

```
  components: {
    EscapeEvent,
    Uploader,
    ImageGallery
  },
```

We have already defined the `escape` method that is emitted from the
`EscapeEvent` component.

## Results

Users that are logged in can now upload and see their own photos.

![uploaded photos](/screenshots/mypage4.png)

Go to [Part 8](/tutorials/part8.md).
