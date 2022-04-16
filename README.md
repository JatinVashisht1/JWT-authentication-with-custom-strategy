# Implementing a Custom JWT Strategy
- although passport is good for other login options like sign in by google, etc.
- It is not making much difference in case of jwt authentication
- So we can also write our own **custom jwt strategy**
- We sign in with the private key and we verify it with the public key

## Pre-requisites
- make sure to have knowledge about how jwt authentication works
- make sure you have watched/followed my guide on passport-jwt [passport-jwt-authentication-guide](https://github.com/JatinVashisht1/passport-jwt-authentication-guide)

## Files to be changed:
- `/config/passport.js`
    - The `ExtractJwt.fromAuthHeaderAsBearerToken()` function is provided by passport to us so we have to write its custom implementation.
- 

## Lets refactor the code
### Removing passport middleware from our routes
- goto `/routes/users.js` and in the `protected` route delete the  passport.authenticate middleware implementation
- goto `/lib/utils.js` file and add new function `authMiddleware` with the followint signature
```
### Creating our own middleware
function authMiddleware(req, res, next)
```
- export the authMiddleware
```
module.export.authMiddleware = authMiddleware
```

#### `authMiddleware` definition
```
function authMiddleware(req, res, next) {
    const tokenParts = req.headers.authorization.split(' ');
    // console.log(tokenParts)
    if (tokenParts[0] === "Bearer" && tokenParts[1].match(/\S+\.\S+\.\S+/) !== null) {
        try {
            const verification = jsonwebtoken.verify(tokenParts[1], PUB_KEY, {
                algorithms: ['RS256']
            })
            console.log('verification is ', verification)
            req.jwt = verification;
            next();
        } catch (error) {
            console.log('error occured', error.message)
            return res.status(401).json({ success: false, msg: "you are not authorized to visit this route" });
        }
    }else{
        return res.status(401).json({ success: false, msg: "you are not authorized to visit this route" });
    }
}
```
- les go through the above code snippet and see what's going on
- Firstly we are accessing the `authorization` part from our `headers` because it is where our token is stored
- The pattern of the authorization header is something like this `Bearer *Our token*`
- We can see that `Bearer` keyword and `token` is seperated with a space, so we will split them up and store the array into a variable named `tokenParts`
- next we will validate our `tokenParts`
    - Firstly we are checking if the keyword `Bearer` is attached with the header,
    - Next we are matching our `token` with a pattern by using `match` function of js and checking validating that it is not null
- If the if condition returns true then we are confirmed that the request is valid
- Next we will verify the token with our public key and also providing which alogrithm we used to encrypt the token
- if the token is not correct then it will throw an exception with error message `JsonWebTokenError: invalid token`
    - in this case we will return `401` status code which means user is not authorized
- if the user is valid we can attach new variable to the request with name `jwt` 
    - and attach our token from `verification` variable and simply call `next` function
- if the if condition turns out to be false then we will send the same response as we send earlier in the catch block

### Updating routes
- goto `/routes/user.js` and update the `protected` get route with the below code snippet
```
router.get('/protected', utils.authMiddleware,(req, res, next)=>{
    // will land user here if user is valid
    console.log(req.jwt)
    res.status(200).json({success: true, msg: 'you are auhorized'})
});
```
- here we are now using our own made middleware to authenticate the jwt/user
- if the user is authenticated we will receive the jwt variable in the request variable

### Removing passport from our code
- in `/routes/users.js` file remove all the passport import statements
- in `/app.js` file also remove all `passport` related code
- delete the `/config/passport.js` file
- your app should be working same as before! 

