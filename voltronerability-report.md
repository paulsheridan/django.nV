#Voltronerability Report
Authors Paul and AJ
Date: April 26

##Voltronerability 1: Injection
###Exposure
We found an instance of vulnerability to injection by typing the following into the "new image" field: testPic',(select password from auth_user where username='admin'),8);--.  Clicking the picture after that point provided a hashed version of the admin credentials.
###Repair
We no longer inject sql directly into the database, but instead we use the ORMs built in features to sanitize the inputs prior to entry.
##Voltronerability 2: Broken Auth
###Exposure
We were able to set the is_admin field by manually editing the new user form and swapping the 'email' spot for is_admin.  We were also able to do the same thing for is_superuser.
###Repair
To fix the problem we changed the 'exclude' field to an 'include' specifying only the fields that we want the user to have access to, such as first name, last name, email and password
##Voltronerability 3: XSS
###Exposure
We were able to successfully place the following JavaScript code into the username field when changing the username after we were logged in:
`<script> alert('javascript injection')</script>`
When we hit save and navigated to another page that displayed the username, our script was executed and we saw the alert.
###Repair
Anywhere in the templates we changed the following:
 `{{ user.username|safe }}`
 to
 `{{ user.username }}`
 By eliminated the `|safe` we were able to ensure that django does not assume that the text is safe.
##Voltronerability 4: Insecure Direct Object Reference(IDOR)
###Exposure
We were able to input the following URL to delete a project when not logged in:
`http://0.0.0.0:8000/taskManager/7/project_delete/`
###Repair
In order to repair this voltronerability we looked to our project_delete function as well as all other functions that should require authentication and we added the following:

`if request.user.has_perm('project_delete'):
    run the code to delete the project.`
