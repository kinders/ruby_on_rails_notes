Devise works mostly out of the box with BackboneJS models.
 The only thing that you need to take note of is that Devise's update and delete routes do not use the standard BackboneJS '/users/:id/', instead it is just '/users'.
 Devise looks at the currently logged in user to determine which user the action is related to.
 To make the routes match just add to your model:

`url: '/users'`

Note: you are setting url not urlRoot.

Update: This will not give you a show route.
