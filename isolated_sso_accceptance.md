# Acceptance criterias for "isolated sso-session" clients

### Actors:

- A: a client participanting in the joint/common circle-of-trust 
- A: a client participanting in the joint/common circle-of-trust 
- I1: a client flagged for isolated-sso functionality  
- I2: a client flagged for isolated-sso functionality

Start conditions:  Clean situation, there are no existing active sessions / cookies present for the user.
All test steps must happen before any session inactity timers expire


### Basic behavior for normal, common SSO-sessions:

1:
```
GIVEN user is logged in to service A 
WHEN the user tries to log in to service B 
THEN the user must be automatically logged in to service B without needing to re-authenticate
```

2:
```
GIVEN user is logged in at level "significant" on service A
WHEN the user tries to log in at level "high" on service B
THEN the eID selector at high level must be displayed and the user must re-authenticate
```

### Isolated SSO session
(isolated SSO session behaves largely the same as a common SSO session)

3:
```
GIVEN user is logged in to service I1
WHEN the user tries to log in to service I1 one more time
THEN the user will be logged in automatically without re-authentication
```

4:
(adhere to normal oidc behaviour):
```
GIVEN user is logged in to service I1
WHEN the user tries to log in to service I1 one more time with prompt=login
THEN the eID selector must be displayed and the user must re-authenticate
```

5:
(session upgrade also within own SSO session):
```
GITT user is logged in at level "significant" on service I1
WHEN the user tries to log in at level "high" on service I1
THEN the eID selector on high level must be displayed and the user must re-authenicate
```
 

### Interactions between common session and isolated sessions:
(basically: re-authentication every time the user crosses a new "SSO boundary")

6:
```
GIVEN user is logged in to service A
WHEN the user tries to log in to service I1
THEN the eID selector must be displayed and the user must re-authenticate
```

7:
```
GIVEN user is logged in to service I1
WHEN the user tries to log in to service A
THEN the eID selector must be displayed and the user must re-authenticate
```

8:
```
GIVEN user is logged in to service I1
WHEN the user tries to log in to service I2 
THEN the eID selector must be displayed and the user must re-authenticate
```


### Crossing the SSO boundary shall not destroy already active sessions:

9:
```
GIVEN user is logged in to service A 
AND user has then logged in to service I1
WHEN the user tries to log in to service B 
THEN the user will be logged in automatically to service B
```

