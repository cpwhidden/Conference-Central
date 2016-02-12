Conference Central
==================

# Session and Speaker model classes design rationale

Each Session object represents one session in a Conference.  Each Session has one Speaker who hosts the Session.  I chose to make this Speaker a separate entity.  My Session class includes a Key property for a speaker which is a unique identifier that points to the appropriate Speaker object.  The Speaker object for a given Session can be obtained using the `get()` function on this key.  A list of Session objects that the speaker is hosting can be obtained through the endpoint named 'getSessionsBySpeaker'.  Although it’s possible to just include a String property for the speaker's name in the model, having a key that points to the real Speaker object will enable better statistics.  For example, querying across multiple conferences to see how many times a speaker has hosted sessions.  The names might be the same for different speakers, so querying using the speaker's unique id won't erroneously overcount.  The Session class likewise has a Key property for the conference, and the Conference object is the parent of the Session, meaning there is a permanent relationship that binds Session and Speaker together.  All Sessions can be queried based on their parent, similar to how a filter works.

The Session class consists of a Key property called conferenceKey, which can contain a key object of type Conference.  This is useful for obtaining the Conference object for the conference at which the session is taking place.  I have a String property called name, which is the name of the Session.  This is a required property because it is perhaps the fundamental data point for end users.  Although keys ultimately make the record unique, without a name, the session would not be relatively identifiable for human beings.  Making the property requires assures users that there will always be that human-memorable property that generally identifies the conference.  There is a String property called highlights that is a place to flexibly describe what the session is about.  There is another key property called speaker, which is the key property that points to the Speaker entity who is the speaker for the session. By calling `get()` on this key, the associated Speaker object is retrieved.  The duration property is a Float property, which represents the duration for which the session lasts in hours.  This could have been an Integer property representing minutes, though.   I have the type of session in the Session class as an enum property.  There are a few different options for handling this property.  I could have just left it as a string property that leaves it up the session creator to define.  This gives a lot of flexibility to the session creator by enabling them to define whatever types they want.  However, if different session creators use a slightly different name (or even a different capitalization), then some queries might provide unexpected and inaccurate results.  I chose to create an enum property that allows for a set number of options.  This is good for queries, but doesn’t give good flexibility to session creators.  I think a good third option would be to enable conference creators to define their own set of session types that session makers would follow.  Conference creators would have the flexibility to define whatever session types they want, but session creators would have to follow those types, which would lead to more accurate queries.  However, that would add another level of complexity that I thought was out-of-scope for this project.  The Session class also has a date property, which represents the date on which the session will take place, which is sensibly a DateProperty.  The startTime property represents the time component, i.e. what time of day the session will start.  This is a TimeProperty, sensibly.

The SessionForm class matches the Session for the most part, but Forms do not have KeyField, so Key property are represented by simple StringFields.  Also, the typeOfSession property uses an EnumField to connect to choices for a session type from the SessionType class.  Date and time are also represented in StringField form.  Lastly, a websafeKey is a web-API safe string used to represent the key for the Session object, which is useful when using endpoints that necessarily reference a Session object, for example when using `addSessionToWishlist`, one uses the Session object's websafeKey find the desired Session to add to the wishlist.

The Speaker class has a name property which is a String.  This field is required for the same reason as Session name as mentioned above.  I also added some additional properties such as age (an Integer property) and emailAddress(a String property) just to demonstrate that the entity can collect many different types of data.

The SpeakerForm has the same fields as the Speaker class, except a websafeKey property, representing a web-API-safe version of the id of the Speaker object, is provided in order to access other possible endpoints, such as the `upcomingSessionsForSpeaker` endpoints, which takes a speaker urlsafe key and finds present and future sessions for a speaker.

One possible path through various endpoints that illustrates how these entities are related is as follows:
1.  Use the `queryConferences` endpoint to get a list of all conferences in the database.  Each entry will provide a `websafeKey`.  Choose one of them to explore this conference further.
2.  Use the Conference `websafeKey` at the `getConferenceSessions` endpoint.  The response will list all sessions for the conference.  Each entry has a `websafeKey` provided.  Copy one to look further at this session.
3.  Use the Session `websafeKey` at the `getSpeakerForSession` endpoint.  The response will be a single SpeakerForm object.  Get the `websafeKey` from this response.
4.  Use the Speaker `websafeKey` at the `getSessionsBySpeaker` endpoint.  This will list all sessions in all conferences that the speaker is speaking at.


# Added queries
The project needed a way to obtain the speaker for a session, so I added `getSpeakerForSession`.  I also added a way to delete all of the sessions in the wishlist at once through `deleteAllSessionsInWishlist`.  I added `upcomingSessionsForSpeaker` in order to retrieve all sessions that are today an in the future for a given speaker.  Lastly, I added 'deleteSession' in order to remove a session from a conference.

# Problem query
Querying for all conference sessions that are *not* of type “workshop” and before 7:00pm is a bit tricky because it involves two inequality filters on two different fields within a single query, which is not possible for Google App Engine.  After doing a bit of research, I found that you can do two `Session` queries separately (only fetching their keys) and then combine them with `set(firstQuery).intersection(secondQuery)`.  The full code is as follows:
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