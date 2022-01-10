#Functional Requirements :

a)Service can generate a short url for a given long url.
b)On hitting the short url the service should redirect it to the original long url.


#Non-Functional Requirements:

a)System must be highly available.
b)System should support read heavy operations.
c)Redirection should occur very fast.

#Load Predictions:

a)URL shortening activities per month=100 million
b)URL shortening activities per second=100 million/30*24*60*60=38.6=39
c)Read/Redirection per month=100*100=10 billion
d)Read/Redirection per second=10 billion/30*24*60*60=3860

#Storage Required:

Lets suppose we expire a url after 10 years.So
a)Total urls stored in 10 years=100 million*12*10=12 billion.
b)Storage required for a single url=100 bytes
c)Total storage needed=12 billion*100=1.2 TB

#APIs Required:

a)GET API
b)UPDATE API
c)CREATE API
d)EXPIRE API

#Database Design:

a)Support read heavy system
b)We can use nosql database as it will be highly scalable and available to support such a large amount of traffic.
c)Just need 2 document:
USER{                         
    user_id(primary key),
	name,
	email,
	password,
}
URL{
    short_url(primary key),
    original_url,
    user_id(foreign key),
	created_date,
    expiration_date
}

#Shortened URL Generation:

a)Calculate a unique hash for the long url(8 characters long)
b)Using a base 64 encoding, an 8 letter long key will be used making total of 64^8=281 trillion possible strings , which satisfies our requirement.
c)A catch here would be if multiple users shorten the same url,they will get ame shortened url,which is not acceptable.We can overcome this using any of the below methods:
(i)Append an increasing sequence of numbers at the beginning or end of the each input url,than generate hash.
(ii)If user needs to sign,append user_id at the beginning or end of the each input url,than generate hash.

#Caching:

a)We can cache the urls generating most traffic(lets suppose 20% cause most traffic)
b)Daily read/redirection of url=3860*60*60*24=0.33billion
c)Cache memory=0.2*0.33*100bytes=6.6GB
d)Least recently used(LRU) caching policy can be used.

#Load balancer can also be used to make system more scalable and available.Round robin algorithm can be used.

#A seperate cleanup cron service would also run to remove the expired links when the expired time is reached.



#System APIs(psuedo code):

a)createURL(user_id,original_url,user_name(optional),expiration_date(optional)) : creates a shortened url ->add to db->returns it.
If expiration date is present , we set the expiration_date to given value else the default value of the expiration date is created_date+10 years.

current_date=Date()
if(isEmpty(expiration_date)){
	expiration_date=current_date+10 years
}
var short_url=createShortUrl(original_url)
addEntryToDb(user_id,original_url,short_url,created_date,expiration_date)
return short_url



b)updateURL(user_id,original_url,user_name(optional),expiration_date(optional)) : creates a shortened url ->update the db->returns it.

current_date=Date()
if(isEmpty(expiration_date)){
	expiration_date=current_date+10 years
}
updateEntryToDb(user_id,original_url,created_date,expiration_date)



c)getOriginalURL(user_id,short_url):
validates the user access to the short url->fetch the long version of the url from db->checks for expiration->redircts to the original long url.

currentDate=Date()
if(isUserHasAccessToUrl(user_id,short_url)){
	var urlObject=getURLObjectLFromDB(user_id,short_url)
	if(urlObject.expiration_date<currentDate){
	return error message
	}
	else{
	redirect(urlObject.original_url)
	}
}



d)expireURL(user_id,short_url):
validates the user access to the short url->remove the url from db.

currentDate=Date()
if(isUserHasAccessToUrl(user_id,short_url)){
	deleteURLObject(user_id,short_url)
}

e)signup(email,password,name):
Registers the user


f)login(email,password):
authenticates the user->returns user object.


#Function Description:

a)createShortUrl(original_url) => returns a short url from original url using the algorithm defined above.

b)addEntryToDb(user_id,original_url,short_url,created_date,expiration_date)=>adds a new url object to URL document in db.

c)updateEntryToDb(user_id,original_url,created_date,expiration_date)=>updates the url object present in URL document in db. 

d)getURLObjectLFromDB(user_id,short_url)=>fetch the url Object from URL document and returns it.

e)redirect(original_url)=>redirects to a given url.




#High Level Diagram:

USER------>signup/login----->short url request----->SERVER-------->encoding with appending user_id-------->ECODING ALGORITHM--------------->DATABASE--------->success/failure---------->USER



