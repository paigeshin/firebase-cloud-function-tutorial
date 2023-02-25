# EVENTS YOU CAN HANDLE WITH CLOUD FUNCTIONS

### Background Triggers

- Datebase Events
- Authentication Events
- Storage Events
- Analytics Events

### HTTP Triggers

- endpoint request
- callable

# HTTP FUNCTIONS

```jsx
// HTTP FUNCTIONS

const functions = require("firebase-functions");

exports.randomNumber = functions.https.onRequest((request, response) => {
  const number = Math.round(Math.random() * 100);
  response.send(number.toString());
});

exports.toTheDojo = functions.https.onRequest((request, response) => {
  response.redirect("https://www.thenetninja.co.uk");
});
```

# HTTP CALLABEL

@args

- HTTP Callalbes can only be invoked from the client
- data, context
- data returns the data sent from the client
- context returns the user's id, token etc.. (if authenticated)

```jsx
// http callable functions
// @args data, context
// data returns the data sent from the client
// context returns the user's id, token, etc. (if authenticated)
exports.sayHello = functions.https.onCall((data, context) => {
  const name = data.name;
  return `hello, ${name}`;
});

exports.addRequest = functions.https.onCall((data, context) => {
  // check request is made by an authenticated user
  if (!context.auth) {
    throw new functions.https.HttpsError([
      "unauthenticated",
      "only authenticated users can add requests",
    ]);
  }
  if (data.text.length > 30) {
    throw new functions.https.HttpsError([
      "invalid-argument",
      "request must be no more than 30 characters long",
    ]);
  }
  return admin.firestore().collection("requests").add({
    text: data.text,
    upvotes: 0,
  });
});

// // upvote callable function
exports.upvote = functions.https.onCall(async (data, context) => {
  // check auth state
  if (!context.auth) {
    throw new functions.https.HttpsError(
      "unauthenticated",
      "only authenticated users can vote up requests"
    );
  }
  // get refs for user doc & request doc
  const user = admin.firestore().collection("users").doc(context.auth.uid);
  const request = admin.firestore().collection("requests").doc(data.id);

  const doc = await user.get();
  // check thew user hasn't already upvoted
  if (doc.data().upvotedOn.includes(data.id)) {
    throw new functions.https.HttpsError(
      "failed-precondition",
      "You can only vote something up once"
    );
  }

```

# Auth Trigger

- Triggered when user is created or deleted

```jsx
// auth trigger (new user signup)
exports.newUserSignup = functions.auth.user().onCreate((user) => {
  // for background triggers you must return a value/promise
  admin.firestore().collection("users").doc(user.uid).set({
    email: user.email,
    upvotedOn: [],
  });
});

// auth trigger (user deleted)
exports.userDeleted = functions.auth.user().onDelete((user) => {
  // for background triggers you must return a value/promise
  const doc = admin.firestore().collection("users").doc(user.uid);
  return doc.delete();
});
```

# Firestore Trigger

- Triggered when database changes

```jsx
// firestore trigger for tracking activity
exports.logActivities = functions.firestore
  .document("/{collection}/{id}")
  .onCreate((snap, context) => {
    console.log(snap.data());

    const activities = admin.firestore().collection("activities");
    const collection = context.params.collection;

    if (collection === "requests") {
      return activities.add({ text: "a new tutorial request was added" });
    }
    if (collection === "users") {
      return activities.add({ text: "a new user signed up" });
    }

    return null;
  });
```
