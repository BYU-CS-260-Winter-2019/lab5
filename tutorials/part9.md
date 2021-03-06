## Part 9: Photo Page

Modify the application so that when you click on a photo you see a single page
with the photo on it. The requirements for this part are:

- The photo should not show any link decoration (outline, border, etc). It is
  not required but you can show an effect when you hover over a photo, like the
  photo getting slightly larger, or showing a shadow around the photo.

- The photo page should show only a single photo, along with the title, user
  name, date, and description.

- The photo page should have its own view in `src/views` and its out route in
  `src/router.js`

- The photo page should dispatch an action to Vuex to fetch the photo, and
  Vuex should fetch the photo from the server using axios.

- The server should have a new REST API for fetching a single photo. This API
  should use Mongoose to get the photo from the database.
  
## Hints
  
To setup a route for an individual photo, you can use a configuration like this:
  
```
{
  path: '/photo/:id',
  name: 'photo',
  component: Photo
}
```
  
Then inside the component, you can use `this.$route.params.id` to get the id that is passed in from the `:id` in the route.
  
If you want to link to this component, use:
  
```
<router-link :to="{ name: 'photo', params: { id: photo._id }}"><img :src="photo.path" /></router-link>
```

  Go to [Part 10](/tutorials/part10.md).
