#Functional Requirements :

->Service can generate a short url for a given long url.
->On hitting the short url the service should redirect it to the original long url.


#Non-Functional Requirements:

->System must be highly available.
->System should support read heavy operations.
->Redirection should occur very fast.

#Load Predictions:

->URL shortening activities per month=100 million
->URL shortening activities per second=100 million/30*24*60*60=38.6=39
->Read/Redirection per month=100*100=10 billion
->Read/Redirection per second=10 billion/30*24*60*60=3860

#Storage Required:

Lets suppose we expire a url after 10 years.So
->Total urls stored in 10 years=100 million*12*10=12 billion.
->Storage required for a single url=100 bytes
->Total storage needed=12 billion*100=1.2 TB

#APIs Required:

->GET API
->UPDATE API
->CREATE API
->EXPIRE API

#Database Design:

->Support read heavy system
->We can use nosql database as it will be highly scalable and available to support such a large amount of traffic.
->Just need 2 document:
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

->Calculate a unique hash for the long url(8 characters long)
->Using a base 64 encoding, an 8 letter long key will be used making total of 64^8=281 trillion possible strings , which satisfies our requirement.
->A catch here would be if multiple users shorten the same url,they will get ame shortened url,which is not acceptable.We can overcome this using any of the below methods:
(i)Append an increasing sequence of numbers at the beginning or end of the each input url,than generate hash.
(ii)If user needs to sign,append user_id at the beginning or end of the each input url,than generate hash.

#Caching:

->We can cache the urls generating most traffic(lets suppose 20% cause most traffic)
->Daily read/redirection of url=3860*60*60*24=0.33billion
->Cache memory=0.2*0.33*100bytes=6.6GB
->Least recently used(LRU) caching policy can be used.

#Load balancer can also be used to make system more scalable and available.

#A seperate cleanup cron service would also run to remove the expired links when the expired time is reached.



#System APIs:

a)createURL(user_id,original_url,user_name(optional),expiration_date(optional)) : creates a shortened url ->add to db->returns it.
If expiration date is present , we set the expiration_date to given value else the default value of the expiration date is created_date+10 years.



b)updateURL(user_id,original_url,user_name(optional),expiration_date(optional)) : creates a shortened url ->update the db->returns it.



c)getOriginalURL(user_id,short_url):
validates the user access to the short url->fetch the long version of the url from db->checks for expiration->redircts to the original long url.



d)expireURL(user_id,short_url):
validates the user access to the short url->remove the url from db.


e)signup(email,password,name):
Registers the user


f)login(email,password):
authenticates the user->returns user object.



#High Level Diagram:

USER------>signup/login----->short url request----->SERVER-------->encoding with appending user_id-------->ECODING ALGORITHM--------------->DATABASE--------->success/failure---------->USER



