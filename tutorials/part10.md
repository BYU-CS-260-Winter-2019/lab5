Part 10: Comments

Modify the application so that users, if they are logged in, can comment on a
photo on its photo page. The requirements for this part are:

- The photo page should have a form for entering comments on the page.

- Comments are owned by a user. A user must be logged in to make comments on a
  photo. The form for adding comments is not displayed if the user is not logged
  in.

- Comments have a date.

- When comments are displayed, they show the user's name who made the comment
  and the date they made it. Format dates like we do for photos.

- When comments are added, they are displayed immediately on the page. You do
  not need to automatically display comments being added by a second user that
  is using the app at the same time (e.g. from a different computer). But you
  should be able to show the comments from the second user if you reload the
  page.

- The photo page should dispatch an action to Vuex to add the comment and to
  fetch comments. VueX should interact with the server using axios to add \*
  comments and fetch comments.

- The schema for comments will need to reference a User and a Photo.

- The server should have a new REST API for adding and getting comments for a
  photo. This API should use Mongoose to interact with the database. You will
  need to populate the database query for comments to fill in the user, like we
  did for photos.
