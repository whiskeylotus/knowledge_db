# Active Directory

## Tools

netexec
impacket
ldap tools
enumerate AD without netexec / bloodhound
FIXME

Look for complex environments feedback

## Kerberos

**Three main parts**
- User/client (aka principle - User/Service principal)
- KDC - Key Distribution Center
	- Authentication server (AS)
	- Ticket-granting server (TGS)
- service/application (aka resource)

Client/User and Authentication Server (AS)
    Query: An access request is sent from the user via a secret key to the authentication server (AS) in the key distribution center (KDC).
    Response: The AS uses the secret key to authenticate the user (by comparing their credentials in the database). Once authenticated, the AS sends its own secret key along with a time-limited token back to the user. The user accepts the key and the time-limited token from the AS.

Client/User and Ticket-granting Service (TGS)
    Query: The user takes the key and time-limited token received from the AS and sends them to the TGS along with a request to access the application.
    Response: When the TGS gets the request, it decrypts it with the secret key that was shared with the AS and issues the client a token, which is encrypted with another secret key.

Client/User and Application/Server
    Query: The client takes the token they got from the TGS and sends it with a secret key to the application they want to access. (Note: Once the client possesses the token from the TGS, the other services can be assured that the client is authenticated.
    Response: When the application receives the request, it encrypts the token with a secret key and shares it back to the client and the TGS.
        Access Granted to User: The application allows the user access for a certain period of time according to the limitations of the token.


## Tickets - asymmetric encryption

- TGT - Ticket Granting Ticket

**Authentication process**

Client --- AS_REQ (request TGT) ---> KDC
Client <-- AS_REP (receive TGT) ---- KDC

Client can use the TGT to request access to the network resources

Client --- TGS-REQ ---> KDC
Client <-- ST      ---- KDC

Client gets a Service-Ticket (TGS ticket) from TGS (Ticket Granting Service)

Client --- AP_REQ ---> KDC
Client <-- AP_REP ---- KDC

## Attacks

- AS-REP Roasting : Sending AS-REQ on behalf of any user and crack hash offline
- Kerberoasting : Stealing hash from service account and crack them offline
- Silver ticket : forged service ticket that enables access to resources on the target service
- Golden ticket : Forged KRBTGT ticket allowing arbitrary domain access, signed by the krbtgt account's password hash

**Kerberos Delegations**

Allows a service to impersonate a user to access another resource. Authentication is delegated, and the final resource responds to the service as if it had the first user's rights. 

- Unconstrained delegation allows a service to impersonate a user when accessing any other service.
- Constrained delegation is a“more restrictive” version of unconstrained delegation. In this case, a service has the right to impersonate a user to a well-defined list of services.
- Resource-based constrained delegation reverses the responsibilities and shifts delegation management to the final resource. It is no longer at the service level that we list the resources to which we can delegate, but at the resource level, a trust list is established. 

**Defense**

- Strong password policy
- Don't set Kerberos pre-authentication disable
- Disable unconstrained delegation
- Use Protected Users Group
- Limit admin accounts and use MFA
- Monitor events 


## Persistence methods

FIXME
