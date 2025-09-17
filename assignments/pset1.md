**Problem Set 1**

**Exercise 1: Reading Concepts**

1. *Invariants*

The two invariants of the state could be:

- The number of purchases for a request cannot exceed the actual count requested.  
- A request cannot be deleted by the owner if the item has already been purchased partially or completely.

I believe that the main purpose of a gift registry is to prevent gift givers from wasting money on gifts that the receiver does not actually want, so the second invariant is more important. 

2. *Fixing an action*

This invariant is broken by the removeItem action, because there is no requirement that prevents it from deleting a partially or completely fulfilled request. This could be fixed by adding a requirement that the item count must not be decremented from the original count saved when the request was initially created.

3. *Inferring behaviour*

Yes, a registry can be opened and closed repeatedly based on the given concept actions. A possible reason to allow this is to give users flexibility in controlling the accessibility of the registry. They may want to make edits to an existing registry for givers to be able to fulfil more accurate requests. It could also be for record-keeping purposes, as the users may want to build a gift wishlist of sorts over time, until they want to make it active in the future and actually allow people to fulfil the requests.

4. *Registry deletion*

Yes and no. From an operational standpoint, not being able to delete a registry may not really matter in practice since there is an option to simply make a registry inactive by closing it and prevent givers from purchasing items for any of the requests in it. However, some users may feel restricted by the lack of privacy as they may want more control over the visibility of past registries, even after inactivation. Although there is a workaround by removing each request individually, this process could be time-consuming and lead to user frustration.

5. *Queries*

An example of a query from a registry owner is to know how many items from the registry have been purchased in total. An example of a query from a gift-giver is to know which requests have already been fulfilled.

6. *Hiding purchases*

I believe this feature would depend on whether the user wants all the purchases to be surprises or an individual giver wants their purchase to be a surprise. In the user’s case, I would add the following action:

surpriseMe(registry: Registry):  
   **requires** the registry exists   
   **effects** hides the item count for all requests

In the gift giver’s case, I would modify the purchase action as follows:  
purchase (purchaser: User, registry: Registry, item: Item, count: Number, ***surpriseOwner: Boolean***)  
   **requires** registry exists, is active and has a request for this item with at least count  
   **effects** create a new purchase for this purchaser, item and count  
and decrement the count in the matching request;  
***hides the count from the registry owner if surpriseOwner \== True.***

7. *Generic types*

Just as the example says, the Item type might be populated by SKU codes or  organised by the Item’s name and supplier information. These are superfluous attributes or irrelevant metadata at this early stage of design. It’s important that the concept is generalisable and not too mired by details that don’t affect the actual state in any way, which is why generic parameters that are relevant to the core conceptual identity are preferred.

**Exercise 2: Extending Concepts**

1\. **state**   
	a set of Users with  
	   a username String  
	   a password String

2\. **actions**  
	register(username: String, password: String): (user: User)  
	   **requires** username does not exist  
	   **effects** creates a new user

	authenticate(username: String, password: String): (user: User)  
	   **requires** username exists and password matches username  
	   **effects** logs in user to previously created account

3\. No two users can have the same username. It is preserved by the register action which requires that a username given by a new user does not already exist, i.e. it is not already associated with a different user.

*4\. email*

**concept** PasswordAuthentication  
**purpose** limit access to known users  
**principle** after a user registers with a username, a password, and email address,  
	they can authenticate with that same username and password,  
	and be treated each time as the same user  
**state**   
	a set of Users with  
	   an Email address  
	   a unique Username  
	   a Password  
	    
**actions**  
	register(username: String, password: String, email: String): (user: User, token: Token)  
	   **requires** email is not already registered and username does not exist  
	   **effects** emails a secret token to the user’s email address

	confirm(username: String, token: Token): (user: User)  
	   **requires** the token provided by the user matches the token sent by email  
	   **effects** registers user by creating a new account 

	authenticate(username: String, password: String): (user: User)  
	   **requires** username exists and password matches username  
	   **effects** logs in user to previously created account

**Exercise 3: Comparing Concepts**

**concept** PersonalAccessToken  
**purpose** reduce user authentication vulnerabilities with short-term, limited access password equivalents  
**principle** 

**state**   
	a set of Users with  
	   a unique username String  
	   a set of personal access Tokens

	a set of Tokens with  
	   a name String  
	   an Expiration date  
	   a set of access Permissions  
	

**actions**  
	createToken(username: String, name: String, date: Expiration, access: Permissions): (token: Token)  
	   **requires** an existing account associated with the username  
	   **effects** generates and returns a new token associated with username

	deleteToken(token: Token)  
	   **effects** removes Token from set of Tokens associated with username

When the article says ‘Treat your access tokens like passwords’, I think the author intends to emphasise the need to keep them safe and private the way we know how to handle passwords; not that they have the same purpose or operational principle. From my understanding, passwords are used for user authentication and account access, but they are not very secure as they're rarely updated and grant a third party unrestricted access to the user’s entire account if compromised. On the other hand, Personal Access Tokens (PATs) are much more secure since they can be limited to specific actions (like API access or certain repositories), and have fixed expiration dates which can prevent long-term damage if leaked to malicious third parties.

I think the Github page could elaborate more on why  a PAT is a much more secure way of managing your online activity, as opposed to only having a password protecting your entire account in the ‘About personal access tokens’ section. Just mentioning them as ‘an alternative to using passwords for authentication’ is very misleading and appears to conflate the two concepts when they operate very differently, despite having the general purpose of security.

**Exercise 4: Defining Concepts**

1. Billable Hours Tracking

**concept** BillablesTracking  
**purpose** track software usage on an hourly basis to calculate billable hours  
**principle** an employee selects an existing project of their company’s,  
	and initiates a billable session to work with the software  
	by recording the start time of the session;  
the company can customise how long a session can run uninterrupted  
	and the session is paused when that limit is reached;  
	the employee can resume the session if they are still using the software;  
	finally the employee records the end time to conclude the session.  
		  
**state**   
	a set of Clients with  
	   a set of Users  
	   a set of Projects  
	   an inactivity duration Limit

	a set of Projects with  
	   a set of Sessions  
	  
	a set of Sessions with  
	   a Start time  
	   an End time  
	   a description String 

**actions**  
	startSession(project: Project, time: Start, description: String): (session: Session)  
	   **requires** the selected project exists  
	   **effects** creates and returns a new session associated with the project 

	endSession(session: Session, time: End time): (session: Session)  
	   **requires** the selected session has a start time  
	   **effects** returns an existing session with an end time

	pauseSession(session: Session, duration: Limit): (time: End time): (session: Session)  
	   **requires** the session has a start time but no end time and has reached the company’s duration limit  
	   **effects** returns an existing session with an end time

	resumeSession(session: Session, time: End time): (session: Session)  
	   **requires** the session has a start time and end time  
	   **effects** deletes the session’s end time and returns it  
	

2. Conference Room Booking

**concept** RoomReservation  
**purpose** eliminating scheduling conflicts when reserving conference rooms  
**principle** the department makes a set of slots available per room for the semester  
and users can reserve a room for future use at a particular date and time;  
the department can create and delete slots if they are unreserved;  
users can edit reservations to reflect changes instead of having to cancel them.  
	  
**state**   
	a Department with  
	   a set of Rooms  
	   an Administrator  
	  
a set of Rooms with  
	   an occupancy Limit  
	   a set of Slots

	a set of Slots with  
	   a Date  
	   a Time

	a set of Reservations with  
	   a User  
	   a party Size  
	   a purpose String  
	  
**actions**  
	reserve(user: User, room: Room, slot: Slot, count: Size, purpose: String): (r: Reservation)  
	   **requires** the slot must be unreserved and the count must be at most the occupancy limit of the room  
	   **effects** creates and returns a new reservation associated with the user and the slot

	editReservation(user: User, r: Reservation, count: Size, purpose: String): (r: Reservation)  
	   **requires** the reservation must exist and the new count must still be within the room’s occupancy limit  
	   **effects** the reservation is updated to reflect a new headcount or purpose and returned

	cancelReservation(user: User, r: Reservation):  
	   **requires** there must be an existing reservation associated with the user  
	   **effects** the slot associated with that reservation is made available again

	createSlot(room: Room, d: Date, t: Time): (slot: Slot)  
	   **effects** creates a new slot associated with the room, date, and time

	deleteSlot(slot: Slot):  
	   **requires** no existing reservation in this slot    
   **effects** removes Slot from the set of slots

3. Electronic Boarding Pass

**concept** eBoardingPassCreation  
**purpose** create a mobile version of boarding pass to reflect real-time changes in travel details  
**principle**  
	  
**state**   
	a set of Users with  
   a set of boarding Passes

a set of boarding Passes with  
   a passenger name String  
   a Seat assignment  
   a boarding Time  
   a Gate number  
   a Flight

a Flight with  
   a Departure airport  
   a Destination airport  
   a flight Code  
   a Time range   
   a flight Status  
   	  
**actions**

