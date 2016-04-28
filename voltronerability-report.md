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

##Voltronerability 5: Misconfig
###Exposure
Debug mode is still activated for this site, which allowed us to see the hashed admin password in the first voltronerability as well as let me find the means of deleting a project that my user wasn't authenticated to do.
###Repair
To fix this, we set DEBUG to False and updated ALLOWED_HOSTS.  This broke the CSS, but fixed the error.

##Voltronerability 6: Exposure
###Exposure
Using the same SQL injection as Voltronerability 1, we revealed the hashed password and using the MD5PasswordHasher, we were able to unhash it.  This exposes user information (in this case, admin) to serious risk of account compromise.
###Repair
We updated the PASSWORD_HASHERS field in settings.py to the stock setting:
`PASSWORD_HASHERS = [
    'django.contrib.auth.hashers.PBKDF2PasswordHasher',
    'django.contrib.auth.hashers.PBKDF2SHA1PasswordHasher',
    'django.contrib.auth.hashers.BCryptSHA256PasswordHasher',
    'django.contrib.auth.hashers.BCryptPasswordHasher',
    'django.contrib.auth.hashers.SHA1PasswordHasher',
    'django.contrib.auth.hashers.MD5PasswordHasher',
    'django.contrib.auth.hashers.CryptPasswordHasher',
]`
Apparently, we could have also put:
`PASSWORD_HASHERS = ['django.contrib.auth.hashers.PBKDF2SHA1PasswordHasher']`

##Voltronerability 7: Access
###Exposure
Since the view only checks to see that the user is authenticated, not that they have permissions to see things like the manage_groups page, we were able to, just by going through the list of URLs and plugging them in, get access to the manage_groups view and change Seth's access to admin
###Repair
This was fixed by changing the user.is_authenticated() check to
`user.is_authenticated and user.has_perm('can_change_group')`

##Voltronerability 8: CSRF
###Exposure
Simply searching in atom for 'csrf' exposed like 6 '@csrf_exempt' decorators above a handful of important views.  I'm not exactly sure what we can write to expose this but it's pretty clear that you'd have to go out of your way not to protect yourself from this.
###Repair
We removed those exempt decorators and verified that the CSRF middleware was being imported in settings.  For posterity, we even moved it to the top of the list, as the documentation says that it should be imported before other middleware classes.
