Conference Central
==================

# Session and Speaker model classes design rationale
My Session class has a property for a speaker which points to its own Speaker object.  Although it’s fine to just include the speaker as a String property, having a speaker will also for more accurate statistics.  For example, querying across multiple conferences to see how many times a speaker has hosted sessions.  Also, speakers with the same name can be differentiated because they each have a unique key.

I have the type of session in the Session class as an enum property.  There are a few different options for handling this property.  I could have just left it as a string property that leaves it up the session creator to define.  This gives a lot of flexibility to the session creator by enabling them to define whatever types they want.  However, if different session creators use a slightly different name (or even a different capitalization), then some queries might provide unexpected and inaccurate results.  I chose to create an enum property that allows for a set number of options.  This is good for queries, but doesn’t give good flexibility to session creators.  I think a good third option would be to enable conference creators to define their own set of session types that session makers would follow.  Conference creators would have the flexibility to define whatever session types they want, but session creators would have to follow those types, which would lead to more accurate queries.  However, that would add another level of complexity that I thought was out-of-scope for this project.


# Added queries
The project needed a way to obtain the speaker for a session, so I added `getSpeakerForSession`.  I also added a way to delete all of the sessions in the wishlist at once through `deleteAllSessionsInWishlist`.  Lastly, I added `upcomingSessionsForSpeaker` in order to retrieve all sessions that are today an in the future for a given speaker.

# Problem query
Querying for all conference sessions that are *not* of type “workshop” and before 7:00pm is a bit tricky because it involves two inequalities in a query, which is illegal for Google App Engine.  After doing a bit of research, I found that you can do two `Session` queries separately (only fetching their keys) and then combine them with `set(firstQuery).intersection(secondQuery)`.  The full code is as follows:
```
nonWorkshop = Session.query(Session.typeOfSession != ‘Workshop’).fetch(keys_only=True)
        before7 = Session.query(Session.startTime <= datetime.strptime(’19:00’, ‘%H:%M’).time()).fetch(keys_only=True)
        sessions = ndb.get_multi(set(nonWorkshop).intersection(before7))
```

# How to use
1.  You will need to get a [Google](developers.google.com) account to launch the app with Google App Engine.
2.  Add a web app to the Google developer [console](console.developers.google.com) and configure the consent screen for OAuth.
3.  Clone the repo to your local machine
4.  The complete files for this app are contained in the ConferenceCentral_Complete folder
5.  In app.yam replace the application id with the app id you receive from the Google console.
6.  In `settings.py` replace `WEB_CLIENT_ID` with the client id you receive in the Google console.  Likewise in `static/js/app.js` replace the `clients` value in the authentication method with your client id.
7.  Download the [Google App Engine Launcher for Python](https://cloud.google.com/appengine/downloads)
8.  Add the app to the launcher with File > Add Existing Application…
9.  Test the app on `localhost` by clicking Run.  Deploy to Google by clicking Deploy.
10.  Your app’s public URL will be:  `{app_id}.appspot.com`.
11.  Test APIs by navigating to `{app_id}.appspot.com/_ah/api/explorer`