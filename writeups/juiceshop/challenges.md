# OWASP Juice Shop v13.0.2

I suck at markdown so I apologize in advance if this document looks hideous.

## Difficulty 1

### Score Board

Go to main page. Open Developer Tools, go to the Sources tab. Look into the main.js file.
Search for "score". Eventually you'll find a line

	path: "score-board"
	
Then, go to `http://your.juice.shop.ip:port/#/score-board`


### Confidential Document

Brute-force directories using your directory brute-force tool of choice. In my case, I used [dirsearch](https://github.com/maurosoria/dirsearch)

	python3 dirsearch.py -u "http://your.juice.shop.ip:port" -w /path/to/dictionary
	...
	Found "200 -   11KB - /ftp"

Then, go to `http://your.juice.shop.ip:port/#/ftp`

*Alternatively, you may also find the /ftp directory by going to `http://your.juice.shop.ip:port/robots.txt*`

Several files can be found here. Server only allows to download .md and .pdf files.
Open the file `acquisitions.md`


### Exposed metrics

Happened to solve this one while trying to solve "Confidential Document (1*)"

Brute-force directories using your directory brute-force tool of choice.  
In my case, I used I used [dirsearch](https://github.com/maurosoria/dirsearch)

	python3 dirsearch.py -u "http://your.juice.shop.ip:port" -w /path/to/dictionary
	...
	Found "200 - 22KB - /metrics"`

Then, go to `http://your.juice.shop.ip:port/#/metrics`


### Privacy Policy

Log into your Juice Shop account. If you don't have one, make one.  
Go to the Account menu in the top bar. Select "Privacy & Security"  
Select "Privacy Policy"

Alternatively, go to `http://your.juice.shop.ip:port/#/privacy-security/privacy-policy`

### Zero Stars

For this one I used BurpSuite since that's what's comfortable for me.
Open the left menu and go to "Customer Feedback"
Type whatever you want under the "comment" field.
Select any rating, solve the captcha.

Open BurpSuite, turn on Intercept Mode.
Submit the feedback, see a request being sent to `POST /api/Feedbacks/`

You're gonna see a JSON body. The important part lies on the "rating" field.

`{"UserId":xxxx,"captchaId":xxxx,"captcha":"xxxx","comment":"xxxx","rating":1}`

Change the value of "rating" to 0. Forward the request.

Additionally, see the server response for it

`{"message":"internal error","errors":["SQLITE_CONSTRAINT: FOREIGN KEY constraint failed"]}`

Might be useful for later.

### DOM XSS

You need to use the payload provided by the challenge's description.

	<iframe src="javascript:alert(`xss`)">
	
Paste it in the search bar on top.

### Bonus Payload

You need to paste the provided payload in the same place as the challenge "DOM XSS"
That'd be the search bar on top

`<iframe width="100%" height="166" scrolling="no" frameborder="no" allow="autoplay" src="https://w.soundcloud.com/player/?url=https%3A//api.soundcloud.com/tracks/771984076&color=%23ff5500&auto_play=true&hide_related=false&show_comments=true&show_user=true&show_reposts=false&show_teaser=true"></iframe>`

Enjoy!

### Bully Chatbot

Open the left menu and select "Support Chat"
Or simply go to `http://your.juice.shop.ip:port/#/chatbot`

Ask for a discount coupon, bot's gonna tell you no. Keep trying until the bot gives up and sends you one.

```{"action":"response","body":"Oooookay, if you promise to stop nagging me here's a 10% coupon code for you: o*IVjga+jm"}```

## Difficulty 2

### Login Admin

This one is about injection so let's try to find the admin's email so we can attempt to log into their account.
One way to find it is by looking through product reviews. You'll quickly find "admin@juice-sh.op"

Go to the Login Page.
Input "admin@juice-sh.op" into the user field, and whatever you want into the password field.
Add "' --" at the end of admin@juice-sh.op

	admin@juice-sh.op' --
	
Attempt to log in.

This is a simple authentication bypass by SQL injection. The problem comes from unsanitized input being included into an SQL query.
It could look like somethiing like `SELECT * FROM users WHERE email = '@email' AND password = '@password'`

By putting `admin@juice-sh.op' --` into the username field, the query will then look like
`SELECT * FROM users WHERE email = 'admin@juice-sh.op' -- ' AND password = @password`

Everything after `--` will be commented, so the final query will look like
`SELECT * FROM users WHERE email = 'admin@juice-sh.op' --`
Allowing us to log in without knowing the password.

### View Basket

Add any item to your cart, go to your basket.
You'll immediately notice a request being made - `GET /rest/basket/someID`

Simply change the number value and you'll be able to see other users' baskets.

## Difficulty 3

### Manipulate Basket

For this one I had to add an item to someone else's basket
So I went to the main page, added an item to the cart and saw the request `POST /api/BasketItems/` being sent.
It included JSON data in the request body 

	{"ProductId":1,"BasketId":"6","quantity":1}

I have no idea if it's the intended solution, but I tried sending `BasketId` twice to see if that'd do anything. (HTTP Parameter Pollution)

	{"ProductId":4,"BasketId":"6","BasketId":"1","quantity":1}

I actually had to try with different product IDs, some of them showed me nasty SQL errors (1, 2, and 3).
Perhaps those products were already in that basket. Even with the errors the challenge was solved but I kept trying until I didn't see one.

## Difficulty 4

### Poison Null Byte

For this one I was trying to find a way to download files in `/ftp`
There's a filter that only allows you to download .md or .pdf files
So what we need to do is for the filename parser to think it's an .md file, but find a way to get the file we want anyway.
To do this we use a null byte, which is used to terminate strings.

`GET /ftp/file.ext%2500.md`

You'll need to double-encode it for it to work. Hence %2500 and not just %00.
Read about poison null bytes to understand how this works better.

### Easter Egg

You'll need to solve "Poison Null Byte" for this one.
Inject a null byte followed by a .md extension to be able to download this file.

`GET /ftp/eastere.gg%2500.md`

### Nested Easter Egg

You'll need to solve "Easter Egg" for this one.
Open the eastere.gg file, inside you'll find some text saying the real easter eggs lies here:
`L2d1ci9xcmlmL25lci9mYi9zaGFhbC9ndXJsL3V2cS9uYS9ybmZncmUvcnR0L2p2Z3V2YS9ndXIvcm5mZ3JlL3J0dA==`

It's very evident the string is encoded in Base 64, so after decoding it we get:

`/gur/qrif/ner/fb/shaal/gurl/uvq/na/rnfgre/rtt/jvguva/gur/rnfgre/rtt`

Looking at it for a bit, and after figuring out it's not some directory it's noticeable how some of the "words" are repeated.
You can see "gur", "rnfgre", "rtt" repeated one or more times. This gave me a feeling it could be some alphabet shift.
Tried shifting the alphabet from a->b all the way to a->n (hey, ROT-13!)

And I got `/the/devs/are/so/funny/they/hid/an/easter/egg/within/the/easter/egg}`

I had no idea what to do with it so I tried sending it as feedback or complaint, but it didn't work.
So I just tried to see, again, if it's a directory. And yup, it sure was.

	http://your.juice.shop.ip:port/the/devs/are/so/funny/they/hid/an/easter/egg/within/the/easter/egg