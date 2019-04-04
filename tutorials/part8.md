# Part 8: Home Page

The last step is to display everyone's photos on the home page.

## Back End

On the back end, we need an API endpoint that will return all the photos in
the system. In `server/photos.js`, add the following endpoint:

```
// get all photos
router.get("/all", async (req, res) => {
  try {
    let photos = await Photo.find().sort({
      created: -1
    }).populate('user');
    return res.send(photos);
  } catch (error) {
    console.log(error);
    return res.sendStatus(500);
  }
});
```

We have to use a different path for this endpoint, since we already have a GET
endpoint for getting the photos for the current user. We do _not_ validate the
token or the user's account because anyone can see the photos.

For this query, we add `populate('user')` to have Mongo fill in the user
information for each photo. This way we can use `photo.user.name` to get the
name of the person who took the photo.

## Vuex

In `Vuex`, we need to add a new method to get all the photos from the back end:

```
    async getAllPhotos(context) {
      try {
        let response = await axios.get("/api/photos/all");
        context.commit('setPhotos', response.data);
        return "";
      } catch (error) {
        return "";
      }
    },
```

## Image Gallery

We need to make some small changes to the Image Gallery to allow it to display
the name of the person who uploaded the photo. Modify the line where we display
the date as follows:

```
<p class="photoDate">
  <span v-if="photo.user.name">{{photo.user.name}}, </span>
  {{formatDate(photo.created)}}
</p>
```

This will display both the user name and the date, but only if the user name is
available. This way the `MyPhotos` page won't display the name for each photo.

## Home Page

Now we modify the home page to use the Image Gallery component. The `template`
section needs just this:

```
<template>
<div class="home">
  <image-gallery :photos="photos" />
</div>
</template>
```

The `script` section needs to import this component, name it as a component,
fetch the photos, and use a computed property to load the photos from Vuex.

```
<script>
// @ is an alias to /src
import ImageGallery from '@/components/ImageGallery.vue'

export default {
  name: 'home',
  components: {
    ImageGallery
  },
  computed: {
    photos() {
      return this.$store.state.photos;
    }
  },
  async created() {
    await this.$store.dispatch("getAllPhotos");
  },
}
</script>
```

## Results

You can now see the photos from all users on the home page.

![home page](/screenshots/homepage.png)

Be sure to use the Network tab on the Developer Tools of your browser to see the
difference between what is returned for a user's photos versus all photos. You
should see just a user ID in the first case and a complete user record in the
second.

![populated user record](/screenshots/populated.png)

Go to [Part 9](/tutorials/part9.md).
