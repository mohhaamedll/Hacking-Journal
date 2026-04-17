# What SOP :

Will SOP is short for Same Origin Policy's , they are implemented by browsers like (Fire Fox , chrome , etc..) for preventing websites from stealing other websites data Via fetching(Read).

-------------------------------------------------------
# How SOP works :

Look every website has an origin that the browsers read and if the website-1 requesting data from website-2 and they both same origin then it allows the request to reach the website-2, but if web-1 origin differ from web-2 origin the browser sent the request normally and the web-2 response but the browser blocks the response from web1** 
so it blocks the requesting page from reading the response.

*NOTE: Same Origin Policy restricts reading cross-origin responses, But it's not related to execution request-responses, Due to that its not protecting against attacks like (XSS,CSRF) but more of data reading requests.*

-------------------------------------------------------
# How is SOP determined :

Same Origin is determined by three things (Scheme , Domain , Port)
- `ex: (https:|example.com:443)`
- The Scheme(https) , Domain(example.com) , Port(443)

-------------------------------------------------------
# SOP Exceptions : 

SOP is a very Strict Policy that if all none origin requests are prevented the websites just won't work so they allow 3 types of requests

- Image requests -> So if a website is Hosting an image on his website.

- Scripts -> So if a website is Hosting an scripts on his website like maps. 

- I-frames -> For embedding Youtube videos or others. 
