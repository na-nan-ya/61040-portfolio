**Concept Questions**

1. *Contexts*

The specification of the context is to organise different sets of nonces in a way that allows a shortened URL to be reused without creating a conflict or overwriting an association in the database managed by the UrlShortening concept. In the URL shortening app, a context could end up being the user-specific domain that is being used to shorten the URL. For example, both users A and B are trying to access pinterest.com/dogs on different URL shortening websites, so both of them can be given a short URL with the same suffix string and a different base (e.g. user A gets shorturl.com/abc123 and user B gets tinyurl.com/abc123), ensuring that the uniqueness of the association is preserved. 

2. *Storing used strings*

*NonceGeneration* must store sets of used strings so that it doesn’t generate a nonce that’s already in use for that context. A counter could be used instead of storing the entire set of used strings if the strings were generated in a systematic, incremental way. For example, treating a six-character string like aaa111 as the starting point and having subsequent strings be aaa112, aaa13, and so on would let each new string be created by incrementing the counter. In other words, the concept could have the unique strings themselves serve as the counter, forming a compressed representation of all used strings so far without storing them explicitly.

3. *Words as nonces*

As the question states, using common dictionary words would increase the memorability of  a short URL, making it more user-friendly. However, due to the ultimately limited nature of the set, there is a much higher chance for collisions and possibly a lack of appropriate, available words that haven’t already been used in that context. The *NonceGeneration* concept could be modified as follows, with the nonce strings being a randomly selected word from the set of unused words, where a word is removed when it is used.

 **concept** NonceGeneration \[Context\]  
  **state**  
&nbsp;&nbsp;&nbsp;&nbsp;a set of Contexts with  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;***an unused set of Words***
<br />
<br />
**Synchronizations for URL Shortening**

1. *Partial matching*

This is because the first sync is responsible for creating the suffix ‘nonce’ string, which only requires the shortUrlBase so it can ensure that the string is unique in the context of that domain. However, the second sync is where the association between the shortened URL and the longer destination URL are actually stored, so the targetUrl is required.

2. *Omitting names*

This convention can only be used when the variable name is clearly analogous to the actual value it represents (e.g. *targetUrl* actually represents the longer, target URL). But, it can be confusing or unnecessary in cases where no such compact, self-explanatory representation exists. For example, in the third sync *setExpiry*, the argument *seconds* in the action *setExpiry* had to be explicitly assigned the value 3600, because there is no efficient or meaningful way to substitute a  specific, large scalar with a variable name. The variable name *seconds* doesn’t naturally represent a value that could be inferred from its name, so the actual value needed to be noted for clarity at the cost of succinctness.

3. *Inclusion of request*

This is because the actions ‘requested’ in the first two syncs are client-side actions, i.e. they are invoked by the user who needs them to generate the shortened URL. On the other hand, the third sync *setExpiry* is completely determined by the URL shortening service and users should not be able to change or tamper with that for both data management and security concerns.

4. *Fixed domain*

First sync *generate*: Remove the argument *shortUrlBase* from the *shortenUrl* action and the context argument from the *generate* action.  
Second sync *register*: Remove the shortUrlBase argument from both the *shortenUrl* and *register* actions.  
Third sync *setExpiry:* No change.

5. *Adding a sync*

**sync** expire  
**when** ExpiringResource.expireResource(): (resource: Resource)  
**then** UrlShortening.delete(shortUrl: Resource)
<br />
<br />
**Extending the design**

1. *Additional concept designs*

**concept** UrlAnalyticsAccessToken  
**state**  
&nbsp;&nbsp;&nbsp;&nbsp;a set of short Urls with  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;a unique token String  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;an Expiration time  
**actions**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;generateToken(shortUrl: ShortUrl, expiry: Expiration): (token: String)  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**requires** the shortened URL must exist  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**effects** generates and returns a string that expires at the same time as the shortUrl
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;registerToken(shortUrl: ShortUrl, token: String)  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**requires** a shortening exists for shortUrl  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**effects** saves the token and associates it with the shortUrl

**concept** UpdateUrlAnalytics  
**state**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;a set of shortened Urls with  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;an access count Number  
**actions**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;incrementCount(shortUrl: ShortUrl):   
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**requires** a shortening exists for shortUrl  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**effects** increments the count associated with the shortUrl

**concept** ViewUrlAnalytics  
**state**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;a set of shortened Urls with  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;a unique token String  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;an access count Number  
**actions**  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;viewCount(shortUrl: ShortUrl, token: String): (count: Number)  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**requires** a shortening exists for shortUrl and the token must match the shortUrl  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**effects** returns the count associated with shortUrl

2. *3 essential synchronizations with new concepts*

**sync** registerToken  
**when**   
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;UrlShortening.register(shortUrlSuffix: nonce, shortUrlBase, targetUrl): (shortUrl)  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ExpiringResource.setExpiry(resource: shortUrl, seconds: 3600\)  
**then**   
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;UrlAnalyticsAccessToken.generateToken(shortUrl, expiry: 3600): (token)  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;UrlAnalyticsAccessToken.registerToken(shortUrl, token)

**sync** incrementCount  
**when** UrlShortening.lookup(shortUrl): (targetUrl)  
**then** UrlAnalytics.incrementCount(shortUrl)

**sync** viewAnalytics  
**when** Request.viewCount(shortUrl, token)  
**then** ViewUrlAnalytics.viewCount(shortUrl, token)

3. *Outline features*

- Undesirable feature: On a primitive or undeveloped level, this could result in many collisions. If this was made into a username-like system where the user is informed if their selected URL is already chosen and prompted to choose another one, this could cause a lot of frustration on the user’s part. The onus is now on them to guess an unused value, turning what should be a quick task into a collision-prone trial-and-error process. It essentially shifts the problem of the uniqueness of the nonce onto the user’s shoulders.

- Echoing the design addition in Part 1 Q3, instead of maintaining a set of used strings, there could be a set of approved, unused words. Words are removed from the set when used for a shortened URL and re-added via a sync when the shortened URL expires and is deleted.

- UpdateUrlAnalytics would associate a count with the *targetUrl* instead of the shortUrl and the increment() action would sync with UrlShortening.lookup() to find the targetUrl associated with the shortUrl given as an argument and update the count.

- The method of generating a nonce would be pseudo-random and also create significantly longer nonces.

- This is already done with the additional concepts I implemented\! The user doesn’t need to register or have an account to access a shortUrl’s analytics, they just need the token for that specific URL which was provided to them when they generated it (which could be stored in a password manager).
