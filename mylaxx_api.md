Mylaxx API

The Mylaxx networking daemon combines a forked version of IPFS, a Bitcoin wallet, and a Ricardian contracting system. This JSON API is the primary mechanism for controlling the node. It offers a RESTful api for building user interfaces. In this documentation you'll find a full list of API calls and example usages.

Getting started
Mylaxx is composed of two components: client and server. The client is a front-end desktop application and primary user interface to the server. For each user, their server is an individual 'node' on the peer-to-peer network.

You can download both client and server from https://Mylaxx.org/download.html

Installation
To interact with the APIs directly, the Mylaxx server will need to be installed either as a pre-packaged binary or from source. Instructions are available to install from source on Windows, macOS, and Linux.

Running Mylaxx
Firstly we need to setup the server:

go run Mylaxxd init
This will initialize the node; this only needs to be performed once. Next, you can run the server with the following command:

go run Mylaxxd start
For more options in running the node, please refer to this guide.

Authentication
When the JSON API is exposed on localhost (the default), the API is unauthenticated. If the JSON API is exposed on anything other than localhost (implying open internet access), then authentication becomes required and must be configured. There are two ways for a client to authenticate. The default (and preferred) is via an authentication cookie. On start up the server generates a random cookie and saves it in the data directory as a .cookie file. You need to add this cookie to the header of all requests.

For example:

cookie: Mylaxx_Auth_Cookie=2Yc7VZtG/pVKrH5Lp0mKRSEPC4xlm1dGpkbUXLehTUI
Alternatively, you can use basic authentication in the request header following RFC 2617. This can be enabled with the following command in the terminal which interactively asks for and sets your HTTP auth username and password:

go run Mylaxxd setapicreds
The username and SHA256 hash of the password are saved in the configuration file. The authenticaion should be passed along as a header:

Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==
Where your unique authentication payload are encoded as:

base64encode(username + ":" + password)
SSL
NEVER open the JSON API to the internet without also encrypting the connection with SSL. Without SSL enabled your authentication information be sent to the server in the clear allowing anyone who views your packets to access your server and potentially steal your funds.

For instructions on how to enable SSL, see here.

Cross-Origin Resource Sharing
CORS is turned off by default, meaning that you will not be able to make API calls from a browser to a running server on localhost. You can turn CORS on for a specific domain by setting an origin in the config file.

Use "*" to allow all domains. Keep in mind that running the server with CORS enabled for all domains is a potential secrity risk as it will allow any website you visit to make calls to your server. Authentication should be enabled along side CORS.

Using the APIs
To develop applications on top of Mylaxx, the APIs must be understood in context of the application architecture and the Mylaxx protocol. In short, Mylaxx is built on top of IPFS, which is a distributed file system hosted on its own peer-to-peer network. This allows content (i.e. listings, encrypted chat and purchase orders) to be persistently hosted by peers on the Mylaxx network.

Mylaxx is also a protocol for buying and selling any good or service with cryptocurrency. Using Ricardian contracts and multisignature escrow, buyers and sellers can transact safely with mechanisms to recover funds in the event of a dispute or accident.

When the Mylaxx server starts for the first time, it creates a public directory of data that is hosted or 'seeded' by other peers.


The first step is to create a public profile and set your private settings for the node. Creating a public facing profile will allow other users to see details such as display name, avatar, and header image when they visit your node. The addresss of your node or peerID can be retrieved by making a GET /ob/config call, or from the profile response. The settings allow you to set your preferred currency (to display prices), country, create a delivery etc; these setting are private and are not visible to the network.

Notes
HEAD requests
As of v0.13.3, all GET endpoints can also accept HEAD requests.

Order Flow
The order flow of Mylaxx transactions depends on whether the payment is sent directly to the seller ('direct'), or the funds are deposited in multisignature/smart contract escrow ('moderated'). Furthermore, the flow will also change depending if the seller is online at the time of purchase.

As the order flow progress, the order will change 'states' (blue boxes in the images below), while will determine how the order will be processed and whether or not funds can be released.


block

API calls that handling blocking a node.

POST Block a node 
http://localhost:4002/ob/blocknode/:peerId
Block a node from the network. Note that this subjective blocking in that it only affects your node.

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
Basic base64encode(username + ":" + password)

PATH VARIABLES
peerIdQmSkkZpNFzWubxE4p9Ejjo2CLqq61jxuRF26ZzCmCKaHcJ
(String) The peerId of the node you want to block.

_________________________________________________________________________________________________________________________________________________

DEL Unblock a node 
http://localhost:4002/ob/blocknode/:peerId
Unblock a blocked node.

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
Basic base64encode(username + ":" + password)

PATH VARIABLES
peerIdQmSkkZpNFzWubxE4p9Ejjo2CLqq61jxuRF26ZzCmCKaHcJ
(String) The peerId of the node you want to block.



__________________________________________________________________________________________________________________________________________________

chat

API calls for handling chat

GET Fetch chat conversations 
http://localhost:4002/ob/chatconversations
API call to retrieve chat conversations

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=

_______________________________________________________________________________________________________________________________________________

GET Fetch chat messages 
http://localhost:4002/ob/chatmessages/:peerid
API call to retrieve chat messages in a specific chat conversation.

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
Basic base64encode(username + ":" + password)

PATH VARIABLES
peerid

______________________________________________________________
  
GET Fetch group chat messages 
http://localhost:4002/ob/chatmessages?limit&offsetId&subject=
API call to retrieve group chat messages, specifically those related to an order.

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
Basic base64encode(username + ":" + password)

PARAMS
limit
offsetId
subject

_____________________________________________________________
POST Send a chat message 
http://localhost:4002/ob/chat
Send a chat message in a chat conversation.

Chat messages are limited to 20,000 characters and subjects are limited to 500 characters.

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
Basic base64encode(username + ":" + password)

BODY raw
{
  "subject": "",
  "message": "Hello",
  "peerId": "QmRGCvxohRX7NtAHJ2vziW6DTitCDJxSsJ3QG7GsJeLgpi"
}


_____________________________________________________________

POST Send a group chat message 
http://localhost:4002/ob/groupchat
Send a chat message in a chat conversation.

Chat messages are limited to 20,000 characters and subjects are limited to 500 characters.

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
Basic base64encode(username + ":" + password)

BODY raw
{
  "subject": "",
  "message": "",
  "peerIds": []
}


__________________________________________________________________

POST Mark chat message as read 
http://localhost:4002/ob/markchatasread/:peerID?subject=
Mark all messages in a conversation as read.

HEADERS
Content-Typeapplication/json
AuthorizationBasic ZHJ3YXNoby10ZXN0LTAwMzpwYXNzd29yZA==
PARAMS
subject
PATH VARIABLES
peerID

_______________________________________________________________

DEL Delete a chat message 
http://localhost:4002/ob/chatmessage/:messageId
Delete a chat message in a chat conversation

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
Basic base64encode(username + ":" + password)

PATH VARIABLES
messageId

_____________________________________________________________

DEL Delete a chat conversation 
http://localhost:4002/ob/chatconversation/:peerID
Delete a chat conversation

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
Basic base64encode(username + ":" + password)

PATH VARIABLES
peerID

___________________________________________________________
disputes

API calls for dispute resolution

GET Get Case History 
http://localhost:4002/ob/cases?limit=&offsetId=
API call to retrieve a history all of cases/disputes.

limit and offsetId work the same way as they do in chat messages.

The queryCount that is returned tells you how many cases are in the database. This is useful for knowning how many more cases are left if you're using the limit option.

HEADERS
AuthorizationBasic ZGVtbzpkZW1vZGVtbw==
PARAMS
limit
offsetId

_____________________________________________________________
GET Fetch case 
http://localhost:4002/ob/case/:caseId
API call to retrieve a specific dispute resolution case that you're the moderator for.

HEADERS
AuthorizationBasic ZGVtbzpkZW1vZGVtbw==
PATH VARIABLES
caseId

______________________________________________________________

POST Open a dispute 
http://localhost:4002/ob/opendispute
Open a dispute of a case you're involved in

HEADERS
Content-Typeapplication/json
BODY raw
{
	"orderId": "",
	"claim": ""
}


__________________________________________________________

POST Close a dispute 
http://localhost:4002/ob/closedispute
Close a dispute of a case you're involved in

buyerPercentage and vendorPercentage are integers between 0 and 100.

HEADERS
Content-Typeapplication/json
AuthorizationBasic ZGVtbzpkZW1vZGVtbw==
BODY raw
{
  "orderId": "",
  "buyerPercentage": 100,
  "vendorPercentage": 0,
  "resolution": ""
}


_______________________________________________________________

POST Release funds 
http://localhost:4002/ob/releasefunds
Release the funds from a multisignature address for a dispute of a case you're involved in

HEADERS
Content-Typeapplication/json
BODY raw
{
  "orderId": ""
}


_________________________________________________________
files

IPFS and IPNS API calls to retrieve files

GET Fetch file from IPFS 
http://localhost:4002/ipfs/:hash
This API call will query the DHT using the given hash to get a list of peers who are seeding the file. Depending on the size of the file it will then download it from one or more of them. The hash may be a directory. In which case this API returns an HTML directory view. To view the files inside the directory, you can add either add their filename or hash to the query. For example... ipfs/QmecpJrN9RJ7smyYByQdZUy5mF6aapgCfKLKRmDtycv9aG/some_image.jpg ipfs/QmecpJrN9RJ7smyYByQdZUy5mF6aapgCfKLKRmDtycv9aG/QmamudHQGtztShX7Nc9HcczehdpGGWpFBWu2JvKWcpELxr And of course nested directories are also supported, referenced again by a name or hash. ipfs/QmecpJrN9RJ7smyYByQdZUy5mF6aapgCfKLKRmDtycv9aG/my_directory/some_image.jpg ipfs/QmecpJrN9RJ7smyYByQdZUy5mF6aapgCfKLKRmDtycv9aG/QmamudHQGtztShX7Nc9HcczehdpGGWpFBWu2JvKWcpELxr/some_image.jpg

HEADERS
Authorization  Basic dXNlcm5hbWU6cGFzc3dvcmQ=
PATH VARIABLES
hash  zb2rhj2TpNPsjPsPNFXoaiaeBtKny1KxcFgFWFiEkeKkRJqJ8

_____________________________________________________________________


GET Fetch file from IPNS 
http://localhost:4002/ipns/:hash/:directory
The IPNS protocol allows for a user to map their peer ID to an IPFS hash in a cryptographically secure manner. When an IPNS query is made, the server first queries the DHT to resolve the peer ID into an IPFS hash, then proceeds to make an IPFS query given the resolved hash. For example, suppose we have the following mapping.. {peerId} = {hash} Then the following API calls are equal... ipfs/{hash}/some_image.jpg ipns/{peerId}/some_image.jpg

HEADERS
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
PATH VARIABLES
hash
directory

_________________________________________________________________________

followers/following

API calls for handling nodes you follow and who follow you

GET Get a list of your followers 
http://localhost:4002/ob/followers/?offsetId=&limit=
Returns a list of peers IDs that follow you

limit and offsetId work the same as with chat messages. Not using the limit option will return all the followers (or all after offsetId if you're using that).

HEADERS
AuthorizationBasic ZHJ3YXNoby10ZXN0LTAwMzpwYXNzd29yZA==
PARAMS
offsetId
limit

_________________________________________________________________

GET Get a list of following 
http://localhost:4002/ob/following?offsetId=&limit=
Returns a list of peer IDs you are following

limit and offsetId work the same as with chat messages. Not using the limit option will return all the following (or all after offsetId if you're using that).

HEADERS
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
PARAMS
offsetId
limit

______________________________________________________

GET Am I following <peer>? 
http://localhost:4002/ob/isfollowing/:peerId
Returns true if you're following the given peer

HEADERS
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
PATH VARIABLES
peerId
______________________________________________________

GET Does <peer> follow me? 
http://localhost:4002/ob/followsme/:peerId
Returns true if the given peer follows you

HEADERS
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
PATH VARIABLES
peerId

________________________________________________________
GET Get followers (external) 
http://localhost:4002/ob/followers/:peerID
Get a list of followers for a node given the peer ID.

HEADERS
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
PATH VARIABLES
peerID

__________________________________________________________

GET Get following (external) 
http://localhost:4002/ob/following/:peerID
Get a list of nodes following a node given the peer ID.

HEADERS
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
PATH VARIABLES
peerID
____________________________________________________________

POST Follow a peer 
http://localhost:4002/ob/follow
RPC call to follow another peer on the network.

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
BODY raw
{
	"id":""
}

_______________________________________________________________

POST Unfollow a peer 
http://localhost:4002/ob/unfollow
RPC call to unfollow another peer on the network.

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
BODY raw
{
	"id":""
}

_____________________________________________________________

images

API calls for handling general + avatar + header images

GET Get images (self) 
http://localhost:4002/ob/images/:imageHash
Get an image given its filename. Alternatively, you can use /ipfs/{hash}

HEADERS
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
PATH VARIABLES
imageHash
________________________________________________________

http://localhost:4002/ob/avatar/:peerID/:size
Get the avatar image of a node according to its size.

HEADERS
AuthorizationBasic ZHJ3YXNoby10ZXN0LTAwMzpwYXNzd29yZA==
PATH VARIABLES
peerID
size
____________________________________________________________


GET Get header (ob) 
http://localhost:4002/ob/header/:peerID/:size
Get the header image of a node according to its size.

HEADERS
AuthorizationBasic ZHJ3YXNoby10ZXN0LTAwMzpwYXNzd29yZA==
PATH VARIABLES
peerID
size
___________________________________________________________


POST Upload image 
http://localhost:4002/ob/images
Upload one or more images. Returns the IPFS hash of each image which can be used to fetch the image using the IPFS API call.

The Post body expects image to be a Base64 encoded string of the image itself with the filename indicating the name of the file without the path.

HEADERS
Content-Typeapplication/json
AuthorizationBasic ZHJ3YXNoby10ZXN0LTAwNjpwYXNzd29yZA==
BODY raw
[
	{
	"filename": "filename.png",
	"image": "base64encodedimagestring"
	}
]

________________________________________________________


POST Set the avatar 
http://localhost:4002/ob/avatar
Set the avatar image. This will also update the avatar hash in the profile.

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
BODY raw
{
	"avatar":""
}


__________________________________________________________

POST Set the header 
http://localhost:4002/ob/header
Set the header image. This will also update the header hash in the profile.

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
BODY raw
{
	"header":""
}


_____________________________________________________________
inventory

API calls for handling inventory

GET Get all inventory 
http://localhost:4002/ob/inventory
Get a list of all inventory counts

_____________________________________________________


POST Set inventory levels 
http://localhost:4002/ob/inventory
Set the inventory for a given item. The item format must be /slug/variant/...

HEADERS
Content-Typeapplication/json
BODY raw
{
	"slug": "",
	"quantity": 0
}


_______________________________________________________________

listings

API calls for handling listings

GET Get an array of own listings 
http://localhost:4002/ob/listings
Returns an array of node's own listings.

HEADERS
AuthorizationBasic ZHJ3YXNoby10ZXN0LTAwMzpwYXNzd29yZA==

_________________________________________________________

     
GET Get an array of external listings 
http://localhost:4002/ob/listings/:peerId
This returns an array of listings from an external node.

HEADERS
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
PATH VARIABLES
peerId
__________________________________________________________


GET Get own listing 
http://localhost:4002/ob/listing/:slug
Returns a specific listing with the inventory attached to the return.

HEADERS
AuthorizationBasic ZHJ3YXNoby10ZXN0LTA3MTpwYXNzd29yZA==
PATH VARIABLES
slugeth-physical-order-testing-w-options
_____________________________________________________________


GET Get an external listing via IPNS 
http://localhost:4002/ob/listing/:peerID/:slug
This returns a listing wrapped in a Ricardian Contract which contains the signatures needed to validate the listing.

This call fetches a listing via IPNS, which requires you to know the peerID of the publisher, and slug of the content. The IPNS call will scan the network to find the latest IPFS hash that matches the slug. It can take significantly longer than an IPFS call, but it will ensure that you'll get the latest version of that content.

HEADERS
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
PATH VARIABLES
peerID    QmTKu8AQ37o4sJquHrmvEW3tGW5bFHqhZ8f66oHFUc2m2H
slug      citizenfour
____________________________________________________________

GET Get an external listing via IPFS 
http://localhost:4002/ob/listing/ipfs/:hash
This returns a listing wrapped in a Ricardian Contract which contains the signatures needed to validate the listing.

This call fetches a listing via IPFS, which requires you to know the hash of the content. Retrieving the listing this way is significantly faster than an IPNS call, but there are no guarantees that the data retrieved is the latest version of that listing. If you try and purchase the listing based on this hash, and it's an older version of the listing, the seller may reject the purchase.

HEADERS
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
PATH VARIABLES
hash QmTeGATiiTEbNwkLosSRivSApstXLof45jWRjSAVoB1QUv

_____________________________________________________________

POST Create a new listing (testnet) 
http://localhost:4002/ob/listing
Create a new listing for testnet.

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
BODY raw
{
    "slug": "",
    "metadata": {
        "version": 1,
        "contractType": "DIGITAL_GOOD",
        "format": "FIXED_PRICE",
        "escrowTimeoutHours": 1080,
        "expiry": "2030-08-17T04:52:19.000Z",
        "acceptedCurrencies": [
            "TBTC",
            "TBCH",
            "TLTC",
            "TZEC",
            "TETH"
        ]
    },
    "item": {
        "title": "Citizenfour",
        "description": "A documentarian and a reporter travel to Hong Kong for the first of many meetings with Edward Snowden.",
        "processingTime": "1 confirmation",
        "priceCurrency": {
            "code": "USD",
            "divisibility": 2
        },
        "bigPrice": "100",
        "tags": [
            "documentary"
        ],
        "images": [
            {
                "tiny": "QmUAuYuiafnJRZxDDX7MuruMNsicYNuyub5vUeAcupUBNs",
                "small": "QmXSEqXLCzpCByJU4wqbJ37TcBEj77FKMUWUP1qLh56847",
                "medium": "QmYVXrKrKHDC9FobgmcmshCDyWwdrfwfanNQN4oxJ9Fk3h",
                "large": "QmUadX5EcvrBmxAV9sE7wUWtWSqxns5K84qKJBixmcT1w9",
                "original": "QmfVj3x4D6Jkq9SEoi5n2NmoUomLwoeiwnYz2KQa15wRw6",
                "filename": "citizenfour.jpg"
            }
        ],
        "categories": [
            "movies"
        ],
        "condition": "New",
        "skus": []
    },
    "coupons": [
        {
            "title": "DASCOUPON",
            "discountCode": "LETMEIN",
            "bigPriceDiscount": "10"
        }
    ],
    "taxes": [
        {
            "taxType": "Sales tax",
            "taxRegions": [
                "UNITED_STATES"
            ],
            "taxShipping": false,
            "percentage": 7
        }
    ],
    "moderators": [
        "Qmdpk5swhYvy5m81mFTm6wv23XRBcGETQutUma3G9CDciP"
    ],
    "termsAndConditions": "NA",
    "refundPolicy": "No refuns for you. All sales are final."
}

___________________________________________________________


POST Create a new listing (physical; w/ options) 
http://localhost:4002/ob/listing
Create a new listing and set its inventory

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
BODY raw
{
    "slug": "",
    "metadata": {
        "contractType": "PHYSICAL_GOOD",
        "format": "FIXED_PRICE",
        "escrowTimeoutHours": 1080,
        "expiry": "2037-12-31T05:00:00.000Z",
        "acceptedCurrencies": [
            "BTC",
            "BCH",
            "LTC",
            "ZEC",
            "ETH"
        ]
    },
    "item": {
        "title": "ETH physical order testing w/ options",
        "description": "This is a listing example for testing orders.",
        "processingTime": "3 days",
        "bigPrice": "100",
        "priceCurrency": {
            "code": "USD",
            "divisibility": 2
        },
        "tags": [
            "vintage dress"
        ],
        "images": [
            {
                "filename": "front",
                "tiny": "QmbjyAxYee4y3443kAMLcmRVwggZsRDKiyXnXus1qdJJWz",
                "small": "QmVsoT9iabv6GZhxhvtjSpQMJA6QyMivGTs6MmHJr6TBm9",
                "medium": "QmTJfeeapZwFM8EoZAuf16JsSJyxZtKaAR6hmWiMf4CTcF",
                "large": "QmfTKL3Z67mWKTKf9XKSCj1ptmDRaZLr5yjPS4JrVDgo5h",
                "original": "QmNexx7SaJCVCjyGGG3j2k7fenn3iVhtWdm9RvKvT7GTLq"
            },
            {
                "filename": "cream",
                "tiny": "QmU1cBgjyHpuzDYbEd4iDVuPzxgKM3CqhRhDJqkHWCKBXq",
                "small": "QmP3BVFuga7N4XEX8iU2MFYC7pc6mfTRQRrpZbKiVy2Csr",
                "medium": "QmQaSzaoHzp8raZLtPEFyCjTnwfXvDGKdXFM83STDVWG43",
                "large": "QmNsFdsX2LNALG2WBxw6E6FTPZWgJcRAcLHnKdWczrCNf9",
                "original": "QmTEUnCjuQPj1ggj5UL5vJujkgBiNYY4jkteugnogiCJny"
            },
            {
                "filename": "black",
                "tiny": "QmdA3Nmc8VnwSvt98Deo2RQztEiCsAkNLhron73bnBzARe",
                "small": "QmcADxUo89ZsEAWiYsuUk7hrgjWDMKXL1CtoA9sTNrQFFP",
                "medium": "QmZydpAJoLsJWbP5vmh59W6bW1kuiCV34yD62hq28AtP7b",
                "large": "QmXixGseetihe6vZiWcTw9N1pieok1YtRoxwvyd5d7jz6s",
                "original": "QmZsZ78FJwt281gfeUvGzDnsBW7WNjPWW3aJWDKskhpCRr"
            },
            {
                "filename": "other_red",
                "tiny": "QmbRFtxNWqACak1vvMJrrxUjzWjTJbMqi3vdUK5ZYvibgt",
                "small": "QmRdYph9YrfpdzMsaDnuySj6U4AY9dZhmjd8Cv2e6SscUG",
                "medium": "QmcD4pkp7SwCmN95pFnED2hz1LfsoYTPpynxeZbxCMoYPL",
                "large": "QmbSQZNAL3pZspUYWm6WNBD1oEQ6i9EnWPEsnk1DfdKnAv",
                "original": "QmZpgjK4jXmdqPg8Jt9YHGVmiuowVve3sbN2AZx7GXioDF"
            }
        ],
        "categories": [
            "👚 Apparel & Accessories"
        ],
        "condition": "New",
        "options": [
            {
                "name": "Color",
                "description": "Color of the dress.",
                "variants": [
                    {
                        "name": "Red",
                        "image": {
                            "filename": "front",
                            "tiny": "QmbjyAxYee4y3443kAMLcmRVwggZsRDKiyXnXus1qdJJWz",
                            "small": "QmVsoT9iabv6GZhxhvtjSpQMJA6QyMivGTs6MmHJr6TBm9",
                            "medium": "QmTJfeeapZwFM8EoZAuf16JsSJyxZtKaAR6hmWiMf4CTcF",
                            "large": "QmfTKL3Z67mWKTKf9XKSCj1ptmDRaZLr5yjPS4JrVDgo5h",
                            "original": "QmNexx7SaJCVCjyGGG3j2k7fenn3iVhtWdm9RvKvT7GTLq"
                        }
                    },
                    {
                        "name": "Cream",
                        "image": {
                            "filename": "cream",
                            "tiny": "QmU1cBgjyHpuzDYbEd4iDVuPzxgKM3CqhRhDJqkHWCKBXq",
                            "small": "QmP3BVFuga7N4XEX8iU2MFYC7pc6mfTRQRrpZbKiVy2Csr",
                            "medium": "QmQaSzaoHzp8raZLtPEFyCjTnwfXvDGKdXFM83STDVWG43",
                            "large": "QmNsFdsX2LNALG2WBxw6E6FTPZWgJcRAcLHnKdWczrCNf9",
                            "original": "QmTEUnCjuQPj1ggj5UL5vJujkgBiNYY4jkteugnogiCJny"
                        }
                    },
                    {
                        "name": "Black",
                        "image": {
                            "filename": "black",
                            "tiny": "QmdA3Nmc8VnwSvt98Deo2RQztEiCsAkNLhron73bnBzARe",
                            "small": "QmcADxUo89ZsEAWiYsuUk7hrgjWDMKXL1CtoA9sTNrQFFP",
                            "medium": "QmZydpAJoLsJWbP5vmh59W6bW1kuiCV34yD62hq28AtP7b",
                            "large": "QmXixGseetihe6vZiWcTw9N1pieok1YtRoxwvyd5d7jz6s",
                            "original": "QmZsZ78FJwt281gfeUvGzDnsBW7WNjPWW3aJWDKskhpCRr"
                        }
                    }
                ]
            },
            {
                "name": "Sizes",
                "description": "Size of the dress.",
                "variants": [
                    {
                        "name": "Small"
                    },
                    {
                        "name": "Medium"
                    },
                    {
                        "name": "Large"
                    },
                    {
                        "name": "Extra Large"
                    }
                ]
            }
        ],
        "skus": [
            {
                "variantCombo": [
                    0,
                    0
                ],
                "productID": "dress-red-small",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            },
            {
                "variantCombo": [
                    0,
                    1
                ],
                "productID": "dress-red-medium",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            },
            {
                "variantCombo": [
                    0,
                    2
                ],
                "productID": "dress-red-large",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            },
            {
                "variantCombo": [
                    0,
                    3
                ],
                "productID": "dress-red-xlarge",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            },
            {
                "variantCombo": [
                    1,
                    0
                ],
                "productID": "dress-cream-small",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            },
            {
                "variantCombo": [
                    1,
                    1
                ],
                "productID": "dress-cream-medium",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            },
            {
                "variantCombo": [
                    1,
                    2
                ],
                "productID": "dress-cream-large",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            },
            {
                "variantCombo": [
                    1,
                    3
                ],
                "productID": "dress-cream-xlarge",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            },
            {
                "variantCombo": [
                    2,
                    0
                ],
                "productID": "dress-black-small",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            },
            {
                "variantCombo": [
                    2,
                    1
                ],
                "productID": "dress-black-medium",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            },
            {
                "variantCombo": [
                    2,
                    2
                ],
                "productID": "dress-black-large",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            },
            {
                "variantCombo": [
                    2,
                    3
                ],
                "productID": "dress-black-xlarge",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            }
        ],
        "nsfw": false
    },
    "shippingOptions": [
        {
            "name": "Worldwide",
            "type": "FIXED_PRICE",
            "regions": [
                "ALL"
            ],
            "services": [
                {
                    "name": "Standard",
                    "bigPrice": "0",
                    "estimatedDelivery": "3 days",
                    "bigAdditionalItemPrice": "0"
                },
                {
                    "name": "Express",
                    "bigPrice": "100",
                    "estimatedDelivery": "3 days",
                    "bigAdditionalItemPrice": "100"
                }
            ]
        }
    ],
    "taxes": [
        {
            "taxType": "Sales tax",
            "taxRegions": [
                "AUSTRIA"
            ],
            "taxShipping": true,
            "percentage": 7
        }
    ],
    "coupons": [
        {
            "title": "DASCOUPON",
            "discountCode": "LETMEIN",
            "bigPriceDiscount": "10"
        },
        {
            "title": "DASCOUPONPERC",
            "discountCode": "LETMEIN",
            "percentDiscount": 10.5
        }
    ],
    "moderators": [
        "QmcdkKM2fWCKzTZdpjEY6abzRbDhRRJvNmqwNKVyiDtGky"
    ],
    "termsAndConditions": "These are my terms and conditions.",
    "refundPolicy": "This is my refund policy."
}

____________________________________________________________


POST Create a new listing (physical; no options) 
http://localhost:4002/ob/listing
Create a new listing and set its inventory

HEADERS
Content-Typeapplication/json
AuthorizationBasic ZGVtbzpkZW1vZGVtbw==
BODY raw
{
    "slug": "",
    "metadata": {
        "contractType": "PHYSICAL_GOOD",
        "format": "FIXED_PRICE",
        "escrowTimeoutHours": 1080,
        "expiry": "2037-12-31T05:00:00.000Z",
        "acceptedCurrencies": [
            "BTC",
            "BCH",
            "LTC",
            "ZEC",
            "ETH"
        ]
    },
    "item": {
        "title": "ETH physical order testing w/o options",
        "description": "This is a listing example for testing orders.",
        "processingTime": "3 days",
        "priceCurrency": {
            "code": "USD",
            "divisibility": 2
        },
        "bigPrice": "100",
        "tags": [
            "vintage dress"
        ],
        "images": [
            {
                "filename": "front",
                "tiny": "QmbjyAxYee4y3443kAMLcmRVwggZsRDKiyXnXus1qdJJWz",
                "small": "QmVsoT9iabv6GZhxhvtjSpQMJA6QyMivGTs6MmHJr6TBm9",
                "medium": "QmTJfeeapZwFM8EoZAuf16JsSJyxZtKaAR6hmWiMf4CTcF",
                "large": "QmfTKL3Z67mWKTKf9XKSCj1ptmDRaZLr5yjPS4JrVDgo5h",
                "original": "QmNexx7SaJCVCjyGGG3j2k7fenn3iVhtWdm9RvKvT7GTLq"
            },
            {
                "filename": "cream",
                "tiny": "QmU1cBgjyHpuzDYbEd4iDVuPzxgKM3CqhRhDJqkHWCKBXq",
                "small": "QmP3BVFuga7N4XEX8iU2MFYC7pc6mfTRQRrpZbKiVy2Csr",
                "medium": "QmQaSzaoHzp8raZLtPEFyCjTnwfXvDGKdXFM83STDVWG43",
                "large": "QmNsFdsX2LNALG2WBxw6E6FTPZWgJcRAcLHnKdWczrCNf9",
                "original": "QmTEUnCjuQPj1ggj5UL5vJujkgBiNYY4jkteugnogiCJny"
            },
            {
                "filename": "black",
                "tiny": "QmdA3Nmc8VnwSvt98Deo2RQztEiCsAkNLhron73bnBzARe",
                "small": "QmcADxUo89ZsEAWiYsuUk7hrgjWDMKXL1CtoA9sTNrQFFP",
                "medium": "QmZydpAJoLsJWbP5vmh59W6bW1kuiCV34yD62hq28AtP7b",
                "large": "QmXixGseetihe6vZiWcTw9N1pieok1YtRoxwvyd5d7jz6s",
                "original": "QmZsZ78FJwt281gfeUvGzDnsBW7WNjPWW3aJWDKskhpCRr"
            },
            {
                "filename": "other_red",
                "tiny": "QmbRFtxNWqACak1vvMJrrxUjzWjTJbMqi3vdUK5ZYvibgt",
                "small": "QmRdYph9YrfpdzMsaDnuySj6U4AY9dZhmjd8Cv2e6SscUG",
                "medium": "QmcD4pkp7SwCmN95pFnED2hz1LfsoYTPpynxeZbxCMoYPL",
                "large": "QmbSQZNAL3pZspUYWm6WNBD1oEQ6i9EnWPEsnk1DfdKnAv",
                "original": "QmZpgjK4jXmdqPg8Jt9YHGVmiuowVve3sbN2AZx7GXioDF"
            }
        ],
        "categories": [
            "👚 Apparel & Accessories"
        ],
        "condition": "New",
        "options": [],
        "skus": [],
        "nsfw": false
    },
    "shippingOptions": [
        {
            "name": "Worldwide",
            "type": "FIXED_PRICE",
            "regions": [
                "ALL"
            ],
            "services": [
                {
                    "name": "Standard",
                    "bigPrice": "0",
                    "estimatedDelivery": "3 days",
                    "bigAdditionalItemPrice": "0"
                },
                {
                    "name": "Express",
                    "bigPrice": "100",
                    "estimatedDelivery": "3 days",
                    "bigAdditionalItemPrice": "100"
                }
            ]
        }
    ],
    "taxes": [
        {
            "taxType": "Sales tax",
            "taxRegions": [
                "AUSTRIA"
            ],
            "taxShipping": true,
            "percentage": 7
        }
    ],
    "coupons": [
        {
            "title": "DASCOUPON",
            "discountCode": "LETMEIN",
            "bigPriceDiscount": "10"
        },
        {
            "title": "DASCOUPONPERC",
            "discountCode": "LETMEIN",
            "percentDiscount": 10.5
        }
    ],
    "moderators": [
        "QmcdkKM2fWCKzTZdpjEY6abzRbDhRRJvNmqwNKVyiDtGky"
    ],
    "termsAndConditions": "These are my terms and conditions.",
    "refundPolicy": "This is my refund policy."
}

__________________________________________________________

POST Create a new listing (digital; w/ options) 
http://localhost:4002/ob/listing
Create a new listing and set its inventory

HEADERS
Content-Typeapplication/json
AuthorizationBasic ZGVtbzpkZW1vZGVtbw==
BODY raw
{
    "slug": "",
    "metadata": {
        "contractType": "DIGITAL_GOOD",
        "format": "FIXED_PRICE",
        "escrowTimeoutHours": 1080,
        "expiry": "2037-12-31T05:00:00.000Z",
        "acceptedCurrencies": [
            "BTC",
            "BCH",
            "LTC",
            "ZEC",
            "ETH"
        ]
    },
    "item": {
        "title": "ETH digital order testing w/ options",
        "description": "This is a listing example for testing orders.",
        "processingTime": "3 days",
        "priceCurrency": {
            "code": "USD",
            "divisibility": 2
        },
        "bigPrice": "100",
        "tags": [
            "vintage dress"
        ],
        "images": [
            {
                "filename": "front",
                "tiny": "QmbjyAxYee4y3443kAMLcmRVwggZsRDKiyXnXus1qdJJWz",
                "small": "QmVsoT9iabv6GZhxhvtjSpQMJA6QyMivGTs6MmHJr6TBm9",
                "medium": "QmTJfeeapZwFM8EoZAuf16JsSJyxZtKaAR6hmWiMf4CTcF",
                "large": "QmfTKL3Z67mWKTKf9XKSCj1ptmDRaZLr5yjPS4JrVDgo5h",
                "original": "QmNexx7SaJCVCjyGGG3j2k7fenn3iVhtWdm9RvKvT7GTLq"
            },
            {
                "filename": "cream",
                "tiny": "QmU1cBgjyHpuzDYbEd4iDVuPzxgKM3CqhRhDJqkHWCKBXq",
                "small": "QmP3BVFuga7N4XEX8iU2MFYC7pc6mfTRQRrpZbKiVy2Csr",
                "medium": "QmQaSzaoHzp8raZLtPEFyCjTnwfXvDGKdXFM83STDVWG43",
                "large": "QmNsFdsX2LNALG2WBxw6E6FTPZWgJcRAcLHnKdWczrCNf9",
                "original": "QmTEUnCjuQPj1ggj5UL5vJujkgBiNYY4jkteugnogiCJny"
            },
            {
                "filename": "black",
                "tiny": "QmdA3Nmc8VnwSvt98Deo2RQztEiCsAkNLhron73bnBzARe",
                "small": "QmcADxUo89ZsEAWiYsuUk7hrgjWDMKXL1CtoA9sTNrQFFP",
                "medium": "QmZydpAJoLsJWbP5vmh59W6bW1kuiCV34yD62hq28AtP7b",
                "large": "QmXixGseetihe6vZiWcTw9N1pieok1YtRoxwvyd5d7jz6s",
                "original": "QmZsZ78FJwt281gfeUvGzDnsBW7WNjPWW3aJWDKskhpCRr"
            },
            {
                "filename": "other_red",
                "tiny": "QmbRFtxNWqACak1vvMJrrxUjzWjTJbMqi3vdUK5ZYvibgt",
                "small": "QmRdYph9YrfpdzMsaDnuySj6U4AY9dZhmjd8Cv2e6SscUG",
                "medium": "QmcD4pkp7SwCmN95pFnED2hz1LfsoYTPpynxeZbxCMoYPL",
                "large": "QmbSQZNAL3pZspUYWm6WNBD1oEQ6i9EnWPEsnk1DfdKnAv",
                "original": "QmZpgjK4jXmdqPg8Jt9YHGVmiuowVve3sbN2AZx7GXioDF"
            }
        ],
        "categories": [
            "👚 Apparel & Accessories"
        ],
        "condition": "New",
        "options": [
            {
                "name": "Color",
                "description": "Color of the dress.",
                "variants": [
                    {
                        "name": "Red",
                        "image": {
                            "filename": "front",
                            "tiny": "QmbjyAxYee4y3443kAMLcmRVwggZsRDKiyXnXus1qdJJWz",
                            "small": "QmVsoT9iabv6GZhxhvtjSpQMJA6QyMivGTs6MmHJr6TBm9",
                            "medium": "QmTJfeeapZwFM8EoZAuf16JsSJyxZtKaAR6hmWiMf4CTcF",
                            "large": "QmfTKL3Z67mWKTKf9XKSCj1ptmDRaZLr5yjPS4JrVDgo5h",
                            "original": "QmNexx7SaJCVCjyGGG3j2k7fenn3iVhtWdm9RvKvT7GTLq"
                        }
                    },
                    {
                        "name": "Cream",
                        "image": {
                            "filename": "cream",
                            "tiny": "QmU1cBgjyHpuzDYbEd4iDVuPzxgKM3CqhRhDJqkHWCKBXq",
                            "small": "QmP3BVFuga7N4XEX8iU2MFYC7pc6mfTRQRrpZbKiVy2Csr",
                            "medium": "QmQaSzaoHzp8raZLtPEFyCjTnwfXvDGKdXFM83STDVWG43",
                            "large": "QmNsFdsX2LNALG2WBxw6E6FTPZWgJcRAcLHnKdWczrCNf9",
                            "original": "QmTEUnCjuQPj1ggj5UL5vJujkgBiNYY4jkteugnogiCJny"
                        }
                    },
                    {
                        "name": "Black",
                        "image": {
                            "filename": "black",
                            "tiny": "QmdA3Nmc8VnwSvt98Deo2RQztEiCsAkNLhron73bnBzARe",
                            "small": "QmcADxUo89ZsEAWiYsuUk7hrgjWDMKXL1CtoA9sTNrQFFP",
                            "medium": "QmZydpAJoLsJWbP5vmh59W6bW1kuiCV34yD62hq28AtP7b",
                            "large": "QmXixGseetihe6vZiWcTw9N1pieok1YtRoxwvyd5d7jz6s",
                            "original": "QmZsZ78FJwt281gfeUvGzDnsBW7WNjPWW3aJWDKskhpCRr"
                        }
                    }
                ]
            },
            {
                "name": "Sizes",
                "description": "Size of the dress.",
                "variants": [
                    {
                        "name": "Small"
                    },
                    {
                        "name": "Medium"
                    },
                    {
                        "name": "Large"
                    },
                    {
                        "name": "Extra Large"
                    }
                ]
            }
        ],
        "skus": [
            {
                "variantCombo": [
                    0,
                    0
                ],
                "productID": "dress-red-small",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            },
            {
                "variantCombo": [
                    0,
                    1
                ],
                "productID": "dress-red-medium",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            },
            {
                "variantCombo": [
                    0,
                    2
                ],
                "productID": "dress-red-large",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            },
            {
                "variantCombo": [
                    0,
                    3
                ],
                "productID": "dress-red-xlarge",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            },
            {
                "variantCombo": [
                    1,
                    0
                ],
                "productID": "dress-cream-small",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            },
            {
                "variantCombo": [
                    1,
                    1
                ],
                "productID": "dress-cream-medium",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            },
            {
                "variantCombo": [
                    1,
                    2
                ],
                "productID": "dress-cream-large",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            },
            {
                "variantCombo": [
                    1,
                    3
                ],
                "productID": "dress-cream-xlarge",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            },
            {
                "variantCombo": [
                    2,
                    0
                ],
                "productID": "dress-black-small",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            },
            {
                "variantCombo": [
                    2,
                    1
                ],
                "productID": "dress-black-medium",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            },
            {
                "variantCombo": [
                    2,
                    2
                ],
                "productID": "dress-black-large",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            },
            {
                "variantCombo": [
                    2,
                    3
                ],
                "productID": "dress-black-xlarge",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            }
        ],
        "nsfw": false
    },
    "shippingOptions": [],
    "taxes": [],
    "coupons": [
        {
            "title": "DASCOUPON",
            "discountCode": "LETMEIN",
            "bigPriceDiscount": "10"
        },
        {
            "title": "DASCOUPONPERC",
            "discountCode": "LETMEIN",
            "percentDiscount": 10.5
        }
    ],
    "moderators": [
        "QmcdkKM2fWCKzTZdpjEY6abzRbDhRRJvNmqwNKVyiDtGky"
    ],
    "termsAndConditions": "These are my terms and conditions.",
    "refundPolicy": "This is my refund policy."
}
_______________________________________________________________

POST Create a new listing (digital; no options) 
http://localhost:4002/ob/listing
Create a new listing and set its inventory

HEADERS
Content-Typeapplication/json
AuthorizationBasic ZGVtbzpkZW1vZGVtbw==
BODY raw
{
    "slug": "",
    "metadata": {
        "contractType": "DIGITAL_GOOD",
        "format": "FIXED_PRICE",
        "escrowTimeoutHours": 1080,
        "expiry": "2037-12-31T05:00:00.000Z",
        "acceptedCurrencies": [
            "TBTC",
            "TBCH",
            "TLTC",
            "TZEC",
            "TETH"
        ]
    },
    "item": {
        "title": "TETH digital order testing w/o options",
        "description": "This is a listing example for testing orders.",
        "processingTime": "3 days",
        "priceCurrency": {
            "code": "USD",
            "divisibility": 2
        },
        "bigPrice": "100",
        "tags": [
            "vintage dress"
        ],
        "images": [
            {
                "filename": "front",
                "tiny": "QmbjyAxYee4y3443kAMLcmRVwggZsRDKiyXnXus1qdJJWz",
                "small": "QmVsoT9iabv6GZhxhvtjSpQMJA6QyMivGTs6MmHJr6TBm9",
                "medium": "QmTJfeeapZwFM8EoZAuf16JsSJyxZtKaAR6hmWiMf4CTcF",
                "large": "QmfTKL3Z67mWKTKf9XKSCj1ptmDRaZLr5yjPS4JrVDgo5h",
                "original": "QmNexx7SaJCVCjyGGG3j2k7fenn3iVhtWdm9RvKvT7GTLq"
            },
            {
                "filename": "cream",
                "tiny": "QmU1cBgjyHpuzDYbEd4iDVuPzxgKM3CqhRhDJqkHWCKBXq",
                "small": "QmP3BVFuga7N4XEX8iU2MFYC7pc6mfTRQRrpZbKiVy2Csr",
                "medium": "QmQaSzaoHzp8raZLtPEFyCjTnwfXvDGKdXFM83STDVWG43",
                "large": "QmNsFdsX2LNALG2WBxw6E6FTPZWgJcRAcLHnKdWczrCNf9",
                "original": "QmTEUnCjuQPj1ggj5UL5vJujkgBiNYY4jkteugnogiCJny"
            },
            {
                "filename": "black",
                "tiny": "QmdA3Nmc8VnwSvt98Deo2RQztEiCsAkNLhron73bnBzARe",
                "small": "QmcADxUo89ZsEAWiYsuUk7hrgjWDMKXL1CtoA9sTNrQFFP",
                "medium": "QmZydpAJoLsJWbP5vmh59W6bW1kuiCV34yD62hq28AtP7b",
                "large": "QmXixGseetihe6vZiWcTw9N1pieok1YtRoxwvyd5d7jz6s",
                "original": "QmZsZ78FJwt281gfeUvGzDnsBW7WNjPWW3aJWDKskhpCRr"
            },
            {
                "filename": "other_red",
                "tiny": "QmbRFtxNWqACak1vvMJrrxUjzWjTJbMqi3vdUK5ZYvibgt",
                "small": "QmRdYph9YrfpdzMsaDnuySj6U4AY9dZhmjd8Cv2e6SscUG",
                "medium": "QmcD4pkp7SwCmN95pFnED2hz1LfsoYTPpynxeZbxCMoYPL",
                "large": "QmbSQZNAL3pZspUYWm6WNBD1oEQ6i9EnWPEsnk1DfdKnAv",
                "original": "QmZpgjK4jXmdqPg8Jt9YHGVmiuowVve3sbN2AZx7GXioDF"
            }
        ],
        "categories": [
            "👚 Apparel & Accessories"
        ],
        "condition": "New",
        "options": [],
        "skus": [],
        "nsfw": false
    },
    "shippingOptions": [],
    "taxes": [
        {
            "taxType": "Sales tax",
            "taxRegions": [
                "AUSTRIA"
            ],
            "taxShipping": true,
            "percentage": 7
        }
    ],
    "coupons": [
        {
            "title": "DASCOUPON",
            "discountCode": "LETMEIN",
            "bigPriceDiscount": "10"
        },
        {
            "title": "DASCOUPONPERC",
            "discountCode": "LETMEIN",
            "percentDiscount": 10.5
        }
    ],
    "moderators": [
        "QmcdkKM2fWCKzTZdpjEY6abzRbDhRRJvNmqwNKVyiDtGky"
    ],
    "termsAndConditions": "These are my terms and conditions.",
    "refundPolicy": "This is my refund policy."
}
_________________________________________________________________


POST Create a new listing (service; w/ options) 
http://localhost:4002/ob/listing
Create a new listing and set its inventory

HEADERS
Content-Typeapplication/json
AuthorizationBasic ZGVtbzpkZW1vZGVtbw==
BODY raw
{
    "slug": "",
    "metadata": {
        "contractType": "SERVICE",
        "format": "FIXED_PRICE",
        "escrowTimeoutHours": 1080,
        "expiry": "2037-12-31T05:00:00.000Z",
        "acceptedCurrencies": [
            "BTC",
            "BCH",
            "LTC",
            "ZEC",
            "ETH"
        ]
    },
    "item": {
        "title": "ETH service order testing w/ options",
        "description": "This is a listing example for testing orders.",
        "processingTime": "3 days",
        "priceCurrency": {
            "code": "USD",
            "divisibility": 2
        },
        "bigPrice": "100",
        "tags": [
            "vintage dress"
        ],
        "images": [
            {
                "filename": "front",
                "tiny": "QmbjyAxYee4y3443kAMLcmRVwggZsRDKiyXnXus1qdJJWz",
                "small": "QmVsoT9iabv6GZhxhvtjSpQMJA6QyMivGTs6MmHJr6TBm9",
                "medium": "QmTJfeeapZwFM8EoZAuf16JsSJyxZtKaAR6hmWiMf4CTcF",
                "large": "QmfTKL3Z67mWKTKf9XKSCj1ptmDRaZLr5yjPS4JrVDgo5h",
                "original": "QmNexx7SaJCVCjyGGG3j2k7fenn3iVhtWdm9RvKvT7GTLq"
            },
            {
                "filename": "cream",
                "tiny": "QmU1cBgjyHpuzDYbEd4iDVuPzxgKM3CqhRhDJqkHWCKBXq",
                "small": "QmP3BVFuga7N4XEX8iU2MFYC7pc6mfTRQRrpZbKiVy2Csr",
                "medium": "QmQaSzaoHzp8raZLtPEFyCjTnwfXvDGKdXFM83STDVWG43",
                "large": "QmNsFdsX2LNALG2WBxw6E6FTPZWgJcRAcLHnKdWczrCNf9",
                "original": "QmTEUnCjuQPj1ggj5UL5vJujkgBiNYY4jkteugnogiCJny"
            },
            {
                "filename": "black",
                "tiny": "QmdA3Nmc8VnwSvt98Deo2RQztEiCsAkNLhron73bnBzARe",
                "small": "QmcADxUo89ZsEAWiYsuUk7hrgjWDMKXL1CtoA9sTNrQFFP",
                "medium": "QmZydpAJoLsJWbP5vmh59W6bW1kuiCV34yD62hq28AtP7b",
                "large": "QmXixGseetihe6vZiWcTw9N1pieok1YtRoxwvyd5d7jz6s",
                "original": "QmZsZ78FJwt281gfeUvGzDnsBW7WNjPWW3aJWDKskhpCRr"
            },
            {
                "filename": "other_red",
                "tiny": "QmbRFtxNWqACak1vvMJrrxUjzWjTJbMqi3vdUK5ZYvibgt",
                "small": "QmRdYph9YrfpdzMsaDnuySj6U4AY9dZhmjd8Cv2e6SscUG",
                "medium": "QmcD4pkp7SwCmN95pFnED2hz1LfsoYTPpynxeZbxCMoYPL",
                "large": "QmbSQZNAL3pZspUYWm6WNBD1oEQ6i9EnWPEsnk1DfdKnAv",
                "original": "QmZpgjK4jXmdqPg8Jt9YHGVmiuowVve3sbN2AZx7GXioDF"
            }
        ],
        "categories": [
            "👚 Apparel & Accessories"
        ],
        "condition": "New",
        "options": [
            {
                "name": "Color",
                "description": "Color of the dress.",
                "variants": [
                    {
                        "name": "Red",
                        "image": {
                            "filename": "front",
                            "tiny": "QmbjyAxYee4y3443kAMLcmRVwggZsRDKiyXnXus1qdJJWz",
                            "small": "QmVsoT9iabv6GZhxhvtjSpQMJA6QyMivGTs6MmHJr6TBm9",
                            "medium": "QmTJfeeapZwFM8EoZAuf16JsSJyxZtKaAR6hmWiMf4CTcF",
                            "large": "QmfTKL3Z67mWKTKf9XKSCj1ptmDRaZLr5yjPS4JrVDgo5h",
                            "original": "QmNexx7SaJCVCjyGGG3j2k7fenn3iVhtWdm9RvKvT7GTLq"
                        }
                    },
                    {
                        "name": "Cream",
                        "image": {
                            "filename": "cream",
                            "tiny": "QmU1cBgjyHpuzDYbEd4iDVuPzxgKM3CqhRhDJqkHWCKBXq",
                            "small": "QmP3BVFuga7N4XEX8iU2MFYC7pc6mfTRQRrpZbKiVy2Csr",
                            "medium": "QmQaSzaoHzp8raZLtPEFyCjTnwfXvDGKdXFM83STDVWG43",
                            "large": "QmNsFdsX2LNALG2WBxw6E6FTPZWgJcRAcLHnKdWczrCNf9",
                            "original": "QmTEUnCjuQPj1ggj5UL5vJujkgBiNYY4jkteugnogiCJny"
                        }
                    },
                    {
                        "name": "Black",
                        "image": {
                            "filename": "black",
                            "tiny": "QmdA3Nmc8VnwSvt98Deo2RQztEiCsAkNLhron73bnBzARe",
                            "small": "QmcADxUo89ZsEAWiYsuUk7hrgjWDMKXL1CtoA9sTNrQFFP",
                            "medium": "QmZydpAJoLsJWbP5vmh59W6bW1kuiCV34yD62hq28AtP7b",
                            "large": "QmXixGseetihe6vZiWcTw9N1pieok1YtRoxwvyd5d7jz6s",
                            "original": "QmZsZ78FJwt281gfeUvGzDnsBW7WNjPWW3aJWDKskhpCRr"
                        }
                    }
                ]
            },
            {
                "name": "Sizes",
                "description": "Size of the dress.",
                "variants": [
                    {
                        "name": "Small"
                    },
                    {
                        "name": "Medium"
                    },
                    {
                        "name": "Large"
                    },
                    {
                        "name": "Extra Large"
                    }
                ]
            }
        ],
        "skus": [
            {
                "variantCombo": [
                    0,
                    0
                ],
                "productID": "dress-red-small",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            },
            {
                "variantCombo": [
                    0,
                    1
                ],
                "productID": "dress-red-medium",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            },
            {
                "variantCombo": [
                    0,
                    2
                ],
                "productID": "dress-red-large",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            },
            {
                "variantCombo": [
                    0,
                    3
                ],
                "productID": "dress-red-xlarge",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            },
            {
                "variantCombo": [
                    1,
                    0
                ],
                "productID": "dress-cream-small",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            },
            {
                "variantCombo": [
                    1,
                    1
                ],
                "productID": "dress-cream-medium",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            },
            {
                "variantCombo": [
                    1,
                    2
                ],
                "productID": "dress-cream-large",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            },
            {
                "variantCombo": [
                    1,
                    3
                ],
                "productID": "dress-cream-xlarge",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            },
            {
                "variantCombo": [
                    2,
                    0
                ],
                "productID": "dress-black-small",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            },
            {
                "variantCombo": [
                    2,
                    1
                ],
                "productID": "dress-black-medium",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            },
            {
                "variantCombo": [
                    2,
                    2
                ],
                "productID": "dress-black-large",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            },
            {
                "variantCombo": [
                    2,
                    3
                ],
                "productID": "dress-black-xlarge",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            }
        ],
        "nsfw": false
    },
    "shippingOptions": [],
    "taxes": [
        {
            "taxType": "Sales tax",
            "taxRegions": [
                "AUSTRIA"
            ],
            "taxShipping": true,
            "percentage": 7
        }
    ],
    "coupons": [
        {
            "title": "DASCOUPON",
            "discountCode": "LETMEIN",
            "bigPriceDiscount": "10"
        },
        {
            "title": "DASCOUPONPERC",
            "discountCode": "LETMEIN",
            "percentDiscount": 10.5
        }
    ],
    "moderators": [
        "QmcdkKM2fWCKzTZdpjEY6abzRbDhRRJvNmqwNKVyiDtGky"
    ],
    "termsAndConditions": "These are my terms and conditions.",
    "refundPolicy": "This is my refund policy."
}

______________________________________________________________


POST Create a new listing (service; no options) 
http://localhost:4002/ob/listing
Create a new listing and set its inventory

HEADERS
Content-Typeapplication/json
AuthorizationBasic ZGVtbzpkZW1vZGVtbw==
BODY raw
{
    "slug": "",
    "metadata": {
        "contractType": "SERVICE",
        "format": "FIXED_PRICE",
        "escrowTimeoutHours": 1080,
        "expiry": "2037-12-31T05:00:00.000Z",
        "acceptedCurrencies": [
            "BTC",
            "BCH",
            "LTC",
            "ZEC",
            "ETH"
        ]
    },
    "item": {
        "title": "ETH service order testing w/o options",
        "description": "This is a listing example for testing orders.",
        "processingTime": "3 days",
        "priceCurrency": {
            "code": "USD",
            "divisibility": 2
        },
        "bigPrice": "100",
        "tags": [
            "vintage dress"
        ],
        "images": [
            {
                "filename": "front",
                "tiny": "QmbjyAxYee4y3443kAMLcmRVwggZsRDKiyXnXus1qdJJWz",
                "small": "QmVsoT9iabv6GZhxhvtjSpQMJA6QyMivGTs6MmHJr6TBm9",
                "medium": "QmTJfeeapZwFM8EoZAuf16JsSJyxZtKaAR6hmWiMf4CTcF",
                "large": "QmfTKL3Z67mWKTKf9XKSCj1ptmDRaZLr5yjPS4JrVDgo5h",
                "original": "QmNexx7SaJCVCjyGGG3j2k7fenn3iVhtWdm9RvKvT7GTLq"
            },
            {
                "filename": "cream",
                "tiny": "QmU1cBgjyHpuzDYbEd4iDVuPzxgKM3CqhRhDJqkHWCKBXq",
                "small": "QmP3BVFuga7N4XEX8iU2MFYC7pc6mfTRQRrpZbKiVy2Csr",
                "medium": "QmQaSzaoHzp8raZLtPEFyCjTnwfXvDGKdXFM83STDVWG43",
                "large": "QmNsFdsX2LNALG2WBxw6E6FTPZWgJcRAcLHnKdWczrCNf9",
                "original": "QmTEUnCjuQPj1ggj5UL5vJujkgBiNYY4jkteugnogiCJny"
            },
            {
                "filename": "black",
                "tiny": "QmdA3Nmc8VnwSvt98Deo2RQztEiCsAkNLhron73bnBzARe",
                "small": "QmcADxUo89ZsEAWiYsuUk7hrgjWDMKXL1CtoA9sTNrQFFP",
                "medium": "QmZydpAJoLsJWbP5vmh59W6bW1kuiCV34yD62hq28AtP7b",
                "large": "QmXixGseetihe6vZiWcTw9N1pieok1YtRoxwvyd5d7jz6s",
                "original": "QmZsZ78FJwt281gfeUvGzDnsBW7WNjPWW3aJWDKskhpCRr"
            },
            {
                "filename": "other_red",
                "tiny": "QmbRFtxNWqACak1vvMJrrxUjzWjTJbMqi3vdUK5ZYvibgt",
                "small": "QmRdYph9YrfpdzMsaDnuySj6U4AY9dZhmjd8Cv2e6SscUG",
                "medium": "QmcD4pkp7SwCmN95pFnED2hz1LfsoYTPpynxeZbxCMoYPL",
                "large": "QmbSQZNAL3pZspUYWm6WNBD1oEQ6i9EnWPEsnk1DfdKnAv",
                "original": "QmZpgjK4jXmdqPg8Jt9YHGVmiuowVve3sbN2AZx7GXioDF"
            }
        ],
        "categories": [
            "👚 Apparel & Accessories"
        ],
        "condition": "New",
        "options": [],
        "skus": [],
        "nsfw": false
    },
    "shippingOptions": [],
    "taxes": [
        {
            "taxType": "Sales tax",
            "taxRegions": [
                "AUSTRIA"
            ],
            "taxShipping": true,
            "percentage": 7
        }
    ],
    "coupons": [
        {
            "title": "DASCOUPON",
            "discountCode": "LETMEIN",
            "bigPriceDiscount": "10"
        },
        {
            "title": "DASCOUPONPERC",
            "discountCode": "LETMEIN",
            "percentDiscount": 10.5
        }
    ],
    "moderators": [
        "QmcdkKM2fWCKzTZdpjEY6abzRbDhRRJvNmqwNKVyiDtGky"
    ],
    "termsAndConditions": "These are my terms and conditions.",
    "refundPolicy": "This is my refund policy."
}

___________________________________________________________________


POST Create a crypto listing 
http://localhost:4002/ob/listing
Create a listing to sell crypto.

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
BODY raw
{
    "slug": "",
    "metadata": {
        "contractType": "CRYPTOCURRENCY",
        "format": "MARKET_PRICE",
        "expiry": "2019-12-02T15:04:05.999999999Z",
        "escrowTimeoutHours": 1080,
        "coinType": "HOT",
        "coinDivisibility": 8,
        "acceptedCurrencies": [
            "BCH"
        ]
    },
    "item": {
        "title": "HOT service order testing w/o options",
        "description": "This is a listing example for testing orders.",
        "processingTime": "3 days",
        "tags": [],
        "images": [
            {
                "filename": "front",
                "tiny": "QmbjyAxYee4y3443kAMLcmRVwggZsRDKiyXnXus1qdJJWz",
                "small": "QmVsoT9iabv6GZhxhvtjSpQMJA6QyMivGTs6MmHJr6TBm9",
                "medium": "QmTJfeeapZwFM8EoZAuf16JsSJyxZtKaAR6hmWiMf4CTcF",
                "large": "QmfTKL3Z67mWKTKf9XKSCj1ptmDRaZLr5yjPS4JrVDgo5h",
                "original": "QmNexx7SaJCVCjyGGG3j2k7fenn3iVhtWdm9RvKvT7GTLq"
            },
            {
                "filename": "cream",
                "tiny": "QmU1cBgjyHpuzDYbEd4iDVuPzxgKM3CqhRhDJqkHWCKBXq",
                "small": "QmP3BVFuga7N4XEX8iU2MFYC7pc6mfTRQRrpZbKiVy2Csr",
                "medium": "QmQaSzaoHzp8raZLtPEFyCjTnwfXvDGKdXFM83STDVWG43",
                "large": "QmNsFdsX2LNALG2WBxw6E6FTPZWgJcRAcLHnKdWczrCNf9",
                "original": "QmTEUnCjuQPj1ggj5UL5vJujkgBiNYY4jkteugnogiCJny"
            },
            {
                "filename": "black",
                "tiny": "QmdA3Nmc8VnwSvt98Deo2RQztEiCsAkNLhron73bnBzARe",
                "small": "QmcADxUo89ZsEAWiYsuUk7hrgjWDMKXL1CtoA9sTNrQFFP",
                "medium": "QmZydpAJoLsJWbP5vmh59W6bW1kuiCV34yD62hq28AtP7b",
                "large": "QmXixGseetihe6vZiWcTw9N1pieok1YtRoxwvyd5d7jz6s",
                "original": "QmZsZ78FJwt281gfeUvGzDnsBW7WNjPWW3aJWDKskhpCRr"
            },
            {
                "filename": "other_red",
                "tiny": "QmbRFtxNWqACak1vvMJrrxUjzWjTJbMqi3vdUK5ZYvibgt",
                "small": "QmRdYph9YrfpdzMsaDnuySj6U4AY9dZhmjd8Cv2e6SscUG",
                "medium": "QmcD4pkp7SwCmN95pFnED2hz1LfsoYTPpynxeZbxCMoYPL",
                "large": "QmbSQZNAL3pZspUYWm6WNBD1oEQ6i9EnWPEsnk1DfdKnAv",
                "original": "QmZpgjK4jXmdqPg8Jt9YHGVmiuowVve3sbN2AZx7GXioDF"
            }
        ],
        "categories": [],
        "options": [],
        "skus": [],
        "nsfw": false
    },
    "shippingOptions": [],
    "taxes": [],
    "coupons": [],
    "moderators": [],
    "termsAndConditions": "This is a test listing.",
    "refundPolicy": "This is a test listing."
}

__________________________________________________________________


PUT Edit a listing 
http://localhost:4002/ob/listing
Update a listing. The currentSlug field is used specify the listing being updated. If the listing contains a new slug, the listing will be renamed using the new slug.

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
BODY raw
{
    "slug": "eth-physical-order-testing-w-options",
    "metadata": {
        "contractType": "PHYSICAL_GOOD",
        "format": "FIXED_PRICE",
        "escrowTimeoutHours": 1080,
        "expiry": "2037-12-31T05:00:00.000Z",
        "acceptedCurrencies": [
            "BTC",
            "BCH",
            "LTC",
            "ZEC",
            "ETH"
        ]
    },
    "item": {
        "title": "ETH physical order testing w/ options",
        "description": "This is a listing example for testing orders.",
        "processingTime": "3 days",
        "bigPrice": "100",
        "priceCurrency": {
            "code": "USD",
            "divisibility": 2
        },
        "tags": [
            "vintage dress"
        ],
        "images": [
            {
                "filename": "front",
                "tiny": "QmbjyAxYee4y3443kAMLcmRVwggZsRDKiyXnXus1qdJJWz",
                "small": "QmVsoT9iabv6GZhxhvtjSpQMJA6QyMivGTs6MmHJr6TBm9",
                "medium": "QmTJfeeapZwFM8EoZAuf16JsSJyxZtKaAR6hmWiMf4CTcF",
                "large": "QmfTKL3Z67mWKTKf9XKSCj1ptmDRaZLr5yjPS4JrVDgo5h",
                "original": "QmNexx7SaJCVCjyGGG3j2k7fenn3iVhtWdm9RvKvT7GTLq"
            },
            {
                "filename": "cream",
                "tiny": "QmU1cBgjyHpuzDYbEd4iDVuPzxgKM3CqhRhDJqkHWCKBXq",
                "small": "QmP3BVFuga7N4XEX8iU2MFYC7pc6mfTRQRrpZbKiVy2Csr",
                "medium": "QmQaSzaoHzp8raZLtPEFyCjTnwfXvDGKdXFM83STDVWG43",
                "large": "QmNsFdsX2LNALG2WBxw6E6FTPZWgJcRAcLHnKdWczrCNf9",
                "original": "QmTEUnCjuQPj1ggj5UL5vJujkgBiNYY4jkteugnogiCJny"
            },
            {
                "filename": "black",
                "tiny": "QmdA3Nmc8VnwSvt98Deo2RQztEiCsAkNLhron73bnBzARe",
                "small": "QmcADxUo89ZsEAWiYsuUk7hrgjWDMKXL1CtoA9sTNrQFFP",
                "medium": "QmZydpAJoLsJWbP5vmh59W6bW1kuiCV34yD62hq28AtP7b",
                "large": "QmXixGseetihe6vZiWcTw9N1pieok1YtRoxwvyd5d7jz6s",
                "original": "QmZsZ78FJwt281gfeUvGzDnsBW7WNjPWW3aJWDKskhpCRr"
            },
            {
                "filename": "other_red",
                "tiny": "QmbRFtxNWqACak1vvMJrrxUjzWjTJbMqi3vdUK5ZYvibgt",
                "small": "QmRdYph9YrfpdzMsaDnuySj6U4AY9dZhmjd8Cv2e6SscUG",
                "medium": "QmcD4pkp7SwCmN95pFnED2hz1LfsoYTPpynxeZbxCMoYPL",
                "large": "QmbSQZNAL3pZspUYWm6WNBD1oEQ6i9EnWPEsnk1DfdKnAv",
                "original": "QmZpgjK4jXmdqPg8Jt9YHGVmiuowVve3sbN2AZx7GXioDF"
            }
        ],
        "categories": [
            "👚 Apparel & Accessories"
        ],
        "condition": "New",
        "options": [
            {
                "name": "Color",
                "description": "Color of the dress.",
                "variants": [
                    {
                        "name": "Red",
                        "image": {
                            "filename": "front",
                            "tiny": "QmbjyAxYee4y3443kAMLcmRVwggZsRDKiyXnXus1qdJJWz",
                            "small": "QmVsoT9iabv6GZhxhvtjSpQMJA6QyMivGTs6MmHJr6TBm9",
                            "medium": "QmTJfeeapZwFM8EoZAuf16JsSJyxZtKaAR6hmWiMf4CTcF",
                            "large": "QmfTKL3Z67mWKTKf9XKSCj1ptmDRaZLr5yjPS4JrVDgo5h",
                            "original": "QmNexx7SaJCVCjyGGG3j2k7fenn3iVhtWdm9RvKvT7GTLq"
                        }
                    },
                    {
                        "name": "Cream",
                        "image": {
                            "filename": "cream",
                            "tiny": "QmU1cBgjyHpuzDYbEd4iDVuPzxgKM3CqhRhDJqkHWCKBXq",
                            "small": "QmP3BVFuga7N4XEX8iU2MFYC7pc6mfTRQRrpZbKiVy2Csr",
                            "medium": "QmQaSzaoHzp8raZLtPEFyCjTnwfXvDGKdXFM83STDVWG43",
                            "large": "QmNsFdsX2LNALG2WBxw6E6FTPZWgJcRAcLHnKdWczrCNf9",
                            "original": "QmTEUnCjuQPj1ggj5UL5vJujkgBiNYY4jkteugnogiCJny"
                        }
                    },
                    {
                        "name": "Black",
                        "image": {
                            "filename": "black",
                            "tiny": "QmdA3Nmc8VnwSvt98Deo2RQztEiCsAkNLhron73bnBzARe",
                            "small": "QmcADxUo89ZsEAWiYsuUk7hrgjWDMKXL1CtoA9sTNrQFFP",
                            "medium": "QmZydpAJoLsJWbP5vmh59W6bW1kuiCV34yD62hq28AtP7b",
                            "large": "QmXixGseetihe6vZiWcTw9N1pieok1YtRoxwvyd5d7jz6s",
                            "original": "QmZsZ78FJwt281gfeUvGzDnsBW7WNjPWW3aJWDKskhpCRr"
                        }
                    }
                ]
            },
            {
                "name": "Sizes",
                "description": "Size of the dress.",
                "variants": [
                    {
                        "name": "Small"
                    },
                    {
                        "name": "Medium"
                    },
                    {
                        "name": "Large"
                    },
                    {
                        "name": "Extra Large"
                    }
                ]
            }
        ],
        "skus": [
            {
                "variantCombo": [
                    0,
                    0
                ],
                "productID": "dress-red-small",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            },
            {
                "variantCombo": [
                    0,
                    1
                ],
                "productID": "dress-red-medium",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            },
            {
                "variantCombo": [
                    0,
                    2
                ],
                "productID": "dress-red-large",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            },
            {
                "variantCombo": [
                    0,
                    3
                ],
                "productID": "dress-red-xlarge",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            },
            {
                "variantCombo": [
                    1,
                    0
                ],
                "productID": "dress-cream-small",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            },
            {
                "variantCombo": [
                    1,
                    1
                ],
                "productID": "dress-cream-medium",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            },
            {
                "variantCombo": [
                    1,
                    2
                ],
                "productID": "dress-cream-large",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            },
            {
                "variantCombo": [
                    1,
                    3
                ],
                "productID": "dress-cream-xlarge",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            },
            {
                "variantCombo": [
                    2,
                    0
                ],
                "productID": "dress-black-small",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            },
            {
                "variantCombo": [
                    2,
                    1
                ],
                "productID": "dress-black-medium",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            },
            {
                "variantCombo": [
                    2,
                    2
                ],
                "productID": "dress-black-large",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            },
            {
                "variantCombo": [
                    2,
                    3
                ],
                "productID": "dress-black-xlarge",
                "bigSurcharge": "100",
                "bigQuantity": "-1"
            }
        ],
        "nsfw": false
    },
    "shippingOptions": [
        {
            "name": "Worldwide",
            "type": "FIXED_PRICE",
            "regions": [
                "ALL"
            ],
            "services": [
                {
                    "name": "Standard",
                    "bigPrice": "0",
                    "estimatedDelivery": "3 days",
                    "bigAdditionalItemPrice": "0"
                },
                {
                    "name": "Express",
                    "bigPrice": "100",
                    "estimatedDelivery": "3 days",
                    "bigAdditionalItemPrice": "100"
                }
            ]
        }
    ],
    "taxes": [
        {
            "taxType": "Sales tax",
            "taxRegions": [
                "AUSTRIA"
            ],
            "taxShipping": true,
            "percentage": 7
        }
    ],
    "coupons": [
        {
            "title": "DASCOUPON",
            "discountCode": "LETMEIN",
            "bigPriceDiscount": "10"
        },
        {
            "title": "DASCOUPONPERC",
            "discountCode": "LETMEIN",
            "percentDiscount": 10.5
        }
    ],
    "moderators": [
        "QmcdkKM2fWCKzTZdpjEY6abzRbDhRRJvNmqwNKVyiDtGky"
    ],
    "termsAndConditions": "These are my terms and conditions.",
    "refundPolicy": "This is my refund policy."
}

______________________________________________________________


DEL Delete a listing 
http://localhost:4002/ob/listing/:slug
Delete a listing and associated inventory

HEADERS
Content-Typeapplication/json
AuthorizationBasic ZHJ3YXNoby10ZXN0LTAwNTpwYXNzd29yZA==
PATH VARIABLES
slugeth-physical-order-testing-w-options
___________________________________________________________

POST Bulk update accepted currencies for listings 
http://localhost:4002/ob/bulkupdatecurrency
Bulk updates what currencies all your listings will accept.

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
BODY raw
[
    "BTC",
    "BCH",
    "LTC",
    "ZEC"
]

_______________________________________________________________________


miscellaneous

Miscellaneous API calls

GET Get node configuration 
http://localhost:4002/ob/config
Returns the node's own peer ID and the type of cryptocurrency used.

HEADERS
AuthorizationBasic ZHJ3YXNoby10ZXN0LTAwMzpwYXNzd29yZA==
__________________________________________________________________


GET Resolve IPNS 
http://localhost:4002/ob/resolveipns
Resolve the root IPNS data.

HEADERS
AuthorizationBasic ZHJ3YXNoby10ZXN0LTAwMzpwYXNzd29yZA==
______________________________________________________________


GET Scan for offline messages 
http://localhost:4002/ob/scanofflinemessages
Scan for offline messages sent to the node.

HEADERS
AuthorizationBasic ZHJ3YXNoby10ZXN0LTAwMzpwYXNzd29yZA==
cookieMylaxx_Auth_Cookie=Token:B1B8511F-2ABE-465E-BBCB-9E579E9ED3E2
_______________________________________________________________


GET Get connected peers 
http://localhost:4002/ob/peers
Returns a list of IDs of the peers connected to this node.

HEADERS
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
_________________________________________________________________


GET Get peer online status 
http://localhost:4002/ob/status/:peerID
Pings the peer and returns if it is online or not

HEADERS
AuthorizationBasic ZHJ3YXNoby10ZXN0LTAwNTpwYXNzd29yZA==
PATH VARIABLES
peerID
____________________________________________________________

GET Get health check on node 
http://localhost:4002/ob/healthcheck
Pings the peer and returns if it is online or not

HEADERS
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
_____________________________________________________________

GET Get peer info 
http://localhost:4002/ob/peerinfo/:peerId
Get available transports for a peer.

HEADERS
AuthorizationBasic ZHJ3YXNoby10ZXN0LTAwNTpwYXNzd29yZA==
PATH VARIABLES
peerId
the peer ID to retrieve info about

_______________________________________________________________



GET Crawl for closest peers 
http://localhost:4002/ob/closestpeers/:peerId
Crawls the DHT and returns up to 20 of the closest peers to the given ID.

PATH VARIABLES
peerId  QmRBhyTivwngraebqBVoPYCh8SBrsagqRtMwj44dMLXhwn
______________________________________________________________

POST Publish changes to the network 
http://localhost:4002/ob/publish
Publish any changes made on your node to the network.

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
______________________________________________________________


POST Sign a message 
http://localhost:4002/ob/signmessage
Sign any message with the node's IPFS private key.

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
BODY raw
{
	"content": "test"
}

______________________________________________________________


POST Verify a message 
http://localhost:4002/ob/verifymessage
Verify any message using the signer's IPFS public key.

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
BODY raw
{
    "content": "test",
    "signature": "44f97b73bcc131f43546e3d39109f1cf730864b338d20f074b1a019374e6642531e98a243cb306fdd65044a1ebcbfc2539cfb410eced31783c8d4166957ba40c",
    "pubkey": "0801122038677bba7d783ec59759766c39369f8f2d9dc5f9cfeffbc7383e4f9b84ac9154",
    "peerId": "QmPyweNFHayJgBRqYaoBatmnfeonxtuVZvQjtWymzmwkav"
}

______________________________________________________________


POST Purge cache 
http://localhost:4002/ob/purgecache
Purge any data you are seeding on the network.

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
____________________________________________________________

POST Resend order message 
http://localhost:4002/ob/resendordermessage
Resend an order message that may have not been successful.

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
BODY raw
{
    "orderID": "QmUm92zCropp3dmN6JPdFfm38mjN5XnXTGdAPEpdvMfyDe",
    "messageType": "ORDER"
}
_______________________________________________________________

POST Shutdown the node 
http://localhost:4002/ob/shutdown
Shutdown your node safely.

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
__________________________________________________________________

moderators

API calls for handling moderators

GET Get a list of moderators 
http://localhost:4002/ob/moderators?async=&include=
Returns a list of moderator peer IDs. Since the server must do a network crawl to find the IDs, there are two ways make this query. The default is a simple long polling GET which returns after the crawl completes. If you want to receive the peer IDs asynchronously, add the async parameter to the request and it will return a random ID which can be used to receive the results over the websocket.

HEADERS
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
PARAMS
async
include
_____________________________________________________________________


PUT Set as moderator 
http://localhost:4002/ob/moderator
Make this node as a moderator and set the modertor's information.

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
BODY raw
{
    "description": "I am a moderator.",
    "termsAndConditions": "Will moderate anything and everything legal.",
    "languages": [
        "English",
        "Spanish"
    ],
    "fee": {
        "feeType": "FIXED_PLUS_PERCENTAGE",
        "percentage": 1,
        "fixedFee": {
            "bigAmount": "10",
            "amountCurrency": {
                "code": "USD",
                "divisibility": 2
            }
        }
    },
    "acceptedCurrencies": [
        "BTC",
        "BCH",
        "LTC",
        "ZEC",
        "ETH"
    ]
}

_____________________________________________________________


PUT Update moderator info 
http://localhost:4002/ob/moderator
Update the moderation info for this node

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
BODY raw
{
    "description": "I am a moderator.",
    "termsAndConditions": "Will moderate anything and everything legal.",
    "languages": [
        "English",
        "Spanish"
    ],
    "fee": {
        "feeType": "FIXED_PLUS_PERCENTAGE",
        "percentage": 1,
        "fixedFee": {
            "bigAmount": "12",
            "amountCurrency": {
                "code": "USD",
                "divisibility": 2
            }
        }
    },
    "acceptedCurrencies": [
        "BTC",
        "BCH",
        "LTC",
        "ZEC",
        "ETH"
    ]
}

___________________________________________________________________


DEL Unset moderator 
http://localhost:4002/ob/moderator/:peerID
Unset self as moderator and delete moderation info

HEADERS
Content-Typeapplication/json
PATH VARIABLES
peerID
_______________________________________________________________________


notifications

API calls handling notifications

GET Fetch notifications 
http://localhost:4002/ob/notifications?limit=&offsetId=
API call to retrieve notifications

HEADERS
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
PARAMS
limit
offsetId
_____________________________________________________________________


POST Mark notification as read 
http://localhost:4002/ob/marknotificationasread/:notificationId
Mark a notification as read

HEADERS
Content-Typeapplication/json
PATH VARIABLES
notificationId
_______________________________________________________________

POST Mark notifications as read (all) 
http://localhost:4002/ob/marknotificationsasread
Mark all notifications as read

HEADERS
Content-Typeapplication/json
_______________________________________________________________

DEL Delete a notification 
http://localhost:4002/ob/notifications/:notificationId
Delete a notification

HEADERS
Content-Typeapplication/json
PATH VARIABLES
notificationId
__________________________________________________________________

orders

API calls for handling orders.

GET Get an order 
http://localhost:4002/ob/order/:orderId
Get an order by ID

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
PATH VARIABLES
orderIdQmfYUVq8puk64Vc7FjDv4dE7XHGotJtij9yF6jejLnmrjg
_____________________________________________________________


GET Get purchase history 
http://localhost:4002/ob/purchases?limit=&offsetId=
Returns a list of all purchases that the node has made.

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
PARAMS
limit
offsetId
_____________________________________________________________


GET Get sales history 
http://localhost:4002/ob/sales?limit=&offsetId=
Returns a list of all sales that the node has made.

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
PARAMS
limit
offsetId
________________________________________________________________


GET Get cases history 
http://localhost:4002/ob/cases?limit=&offsetId=
Returns a list of all moderation cases that the node was involved in.

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
PARAMS
limit
offsetId
___________________________________________________________________


POST Get Estimate to Purchase a Listing 
http://localhost:4002/ob/estimatetotal
Find out how much a listing is going to cost.

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
BODY raw
{
    "shipTo": "Elwood Blues",
    "address": "1060 W Addison",
    "city": "Chicago",
    "state": "Illinois",
    "countryCode": "UNITED_STATES",
    "postalCode": "60613",
    "addressNotes": "",
    "items": [
        {
            "listingHash": "QmTjLBaqrunGmvCD9A3joXLXGudAwxnQvMtvcggQ5rahxN",
            "bigQuantity": "1",
            "options": [
                {
                    "name": "Sizes",
                    "value": "Small"
                },
                {
                    "name": "Color",
                    "value": "Red"
                }
            ],
            "shipping": {
                "name": "Worldwide",
                "service": "Standard"
            },
            "memo": "thanks!",
            "coupons": []
        }
    ],
    "moderator": "QmcdkKM2fWCKzTZdpjEY6abzRbDhRRJvNmqwNKVyiDtGky",
    "paymentCoin": "LTC"
}

___________________________________________________________________________

POST Send purchase order 
http://localhost:4002/ob/purchase
The purchase call can be made to a reachable or a unreachable vendor (offline or not able to receive incoming messages).

An order will be created in the AWAITING_PAYMENT state after this call.

If the total of the purchase is not more than 4X the current transaction fee, the purchase will be rejected (ie: if the fee is 0.0001, the total purchase must be more than 0.0004).

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
BODY raw
{
    "shipTo": "Elwood Blues",
    "address": "1060 W Addison",
    "city": "Chicago",
    "state": "Illinois",
    "countryCode": "UNITED_STATES",
    "postalCode": "60613",
    "addressNotes": "",
    "items": [
        {
            "listingHash": "QmTeGATiiTEbNwkLosSRivSApstXLof45jWRjSAVoB1QUv",
            "bigQuantity": "1",
            "options": [
                {
                    "name": "Sizes",
                    "value": "Small"
                },
                {
                    "name": "Color",
                    "value": "Red"
                }
            ],
            "shipping": {
                "name": "Worldwide",
                "service": "Standard"
            },
            "memo": "thanks!",
            "coupons": []
        }
    ],
    "moderator": "QmcdkKM2fWCKzTZdpjEY6abzRbDhRRJvNmqwNKVyiDtGky",
    "paymentCoin": "TLTC"
}

____________________________________________________________________


POST Send funds for an order 
http://localhost:4002/ob/orderspend
Used for sending money to another address for the express purpose of paying for an order within Mylaxx. This endpoint allows wallet implementations to properly associate a spend with the appropriate order.

currencyCode can be provided instead of the currency object for the default divisibility definition. Similarly, the currency object can be provided instead of currencyCode for custom divisibility.

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
Basic base64encode(username + ":" + password)

BODY raw
{
    "currencyCode": "TLTC",
    "currency": {
    	"code": "TLTC",
    	"divisibility": 8
    },
    "address": "tltc1qnwagagvyj5dsx6dz48n7zs6hpwryhaep8csrarwtulhdd4mcvsvqmdj7y5",
    "amount": "3559244",
    "feeLevel": "NORMAL",
    "requireAssociateOrder": true,
    "orderID": "QmfYUVq8puk64Vc7FjDv4dE7XHGotJtij9yF6jejLnmrjg"
}

____________________________________________________________________________


POST Confirm an order (offline) 
http://localhost:4002/ob/orderconfirmation
Online orders are confirmed instantly. This API call is to confirm an order sent to the vendor while he was offline.

HEADERS
Content-Typeapplication/json
BODY raw
{
	"orderId": "QmamudHQGtztShX7Nc9HcczehdpGGWpFBWu2JvKWcpELxr",
	"reject": false
}

_________________________________________________________________________________


POST Fulfill an order (physical good) 
http://localhost:4002/ob/orderfulfillment
Send an order fulfillment message to the buyer. Typically used to tell the buyer an item has been shipped.

HEADERS
Content-Typeapplication/json
BODY raw
{
    "orderId": "QmfYUVq8puk64Vc7FjDv4dE7XHGotJtij9yF6jejLnmrjg",
    "physicalDelivery": [
        {
            "shipper": "UPS",
            "trackingNumber": "1Z204E380338943508"
        }
    ]
}

_________________________________________________________________________________


POST Fulfill an order (digital good, service) 
http://localhost:4002/ob/orderfulfillment
Send an order fulfillment message to the buyer for a digital good or service.

HEADERS
Content-Typeapplication/json
BODY raw
{
    "orderId": "QmSJmnSgxfME35BiwedKvVMRBcFu4NA7MpB22vhsZ9GF2H",
    "digitalDelivery": [
        {
            "url": "http://example.com/download.mp3",
            "password": "letmein"
        }
    ]
}

__________________________________________________________________________


POST Complete an order 
http://localhost:4002/ob/ordercompletion
Send the order complete message (including the rating) to the vendor. If this is a moderated order, it will sign and release the funds to the vendor.

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
BODY raw
{
    "orderId": "QmfYUVq8puk64Vc7FjDv4dE7XHGotJtij9yF6jejLnmrjg",
    "ratings": [
        {
            "slug": "eth-physical-order-testing-w-options",
            "overall": 5,
            "quality": 5,
            "description": 5,
            "deliverySpeed": 5,
            "customerService": 5,
            "review": "This rocks!",
            "anonymous": false
        }
    ]
}

_________________________________________________________________________________


POST Resend order message 
http://localhost:4002/ob/resendordermessage
Triggr the server to resend order messages.

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
BODY raw
{
    "orderID": "QmSfSiJerLBJ5Ba86E1BECFgKsrj9NAZFMDX5S7WbZmu6P",
    "messageType": "ORDER_COMPLETION"
}

____________________________________________________________________________


POST Refund an order 
http://localhost:4002/ob/refund
Refund the order. If it's a moderated order, it will release the funds back to the buyer. If it's direct it will send the coins from your wallet.

HEADERS
Content-Typeapplication/json
BODY raw
{
	"orderId": "QmamudHQGtztShX7Nc9HcczehdpGGWpFBWu2JvKWcpELxr"
}

__________________________________________________________________________

POST Cancel an offline order 
http://localhost:4002/ob/ordercancel
Cancel an outstanding offline order. It will move the bitcoins back into your wallet.

HEADERS
Content-Typeapplication/json
BODY raw
{
	"orderId": "QmamudHQGtztShX7Nc9HcczehdpGGWpFBWu2JvKWcpELxr"
}

_______________________________________________________________________

POST Release funds from timed-out escrow 
http://localhost:4002/ob/releaseescrow
If the order is in the FULFILLED state, and 45 days have passed since the order was funded, the seller can call this API. The time is based on the number of blocks since the payment was confirmed, multiplied by the average time for a block to be written.

If the order is in the DISPUTED state, and 45 days have passed since the dispute was started, the seller can call this API. The time in this case is based on the timestamp of the dispute.

HEADERS
Content-Typeapplication/json
BODY raw
{
	"orderId": "QmUUPmgb8S5ibWHYvkgBj8VCC6CtBe2HZZVb7S2ukZCsMH"
}

__________________________________________________________________________________


posts

API calls for handling listings

GET Get an array of own posts 
http://localhost:4002/ob/posts
Returns an array of node's own posts.

HEADERS
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
_____________________________________________________________________________

GET Get an array of external posts 
http://localhost:4002/ob/posts/:peerId
This returns an array of posts from an external node.

HEADERS
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
PATH VARIABLES
peerId
_________________________________________________________________________________

GET Get own post 
http://localhost:4002/ob/post/:slug
Returns a specific listing with the inventory attached to the return.

HEADERS
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
PATH VARIABLES
slug
_____________________________________________________________________________

GET Get an external post 
http://localhost:4002/ob/post/:peerID/:slug
This returns a listing wrapped in a Ricardian Contract which contains the signatures needed to validate the listing. Alternatively, you can fetch a listing using /ipfs/{listing_hash}

HEADERS
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
PATH VARIABLES
peerID
slug
___________________________________________________________________________

POST Create a new post 
http://localhost:4002/ob/post
Create a new post

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
BODY raw
{
    "status": "testy",
    "longForm": "This is a test post dawg.",
    "images": [
        {
            "filename": "cat",
            "large": "zb2rhmBUB9i7UkfmeD3obJYK3FFS5K8N8QHaUanG8UWLVBHiY",
            "medium": "zb2rhaFhqziCWk1zo5tMRxQEUchfvJFaGG4DY1anEoR4GnYrN",
            "original": "zb2rhe2o6WbHqcER5VUKsMUbQrmpCC6ihg8qZ4JS9wVgKz9wm",
            "small": "zb2rhbDCeEiTTunugWPaRRKFCfNKUaB7aCR53nrPnMa9usZXY",
            "tiny": "zb2rhgqJDbshwAgPjs7X2h4mDm3V3BpLbp4tFGqkg1LNkg9yV"
        }
    ],
    "tags": [
        "Yo"
    ],
    "channel": [],
    "postType": "POST",
    "reference": ""
}

______________________________________________________________________________


POST Create a comment 
http://localhost:4002/ob/post
Create a comment

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
BODY raw
{
    "status": "This is a comment",
    "longForm": "",
    "images": [
        {
            "filename": "cat",
            "large": "zb2rhmBUB9i7UkfmeD3obJYK3FFS5K8N8QHaUanG8UWLVBHiY",
            "medium": "zb2rhaFhqziCWk1zo5tMRxQEUchfvJFaGG4DY1anEoR4GnYrN",
            "original": "zb2rhe2o6WbHqcER5VUKsMUbQrmpCC6ihg8qZ4JS9wVgKz9wm",
            "small": "zb2rhbDCeEiTTunugWPaRRKFCfNKUaB7aCR53nrPnMa9usZXY",
            "tiny": "zb2rhgqJDbshwAgPjs7X2h4mDm3V3BpLbp4tFGqkg1LNkg9yV"
        }
    ],
    "tags": [],
    "channel": [],
    "postType": "COMMENT",
    "reference": "QmdQBWA75xQSMZpTibQ2G83enNdriz2v14tetGvNrpr5KB/this-is-a-social-post"
}

____________________________________________________________________________________


POST Create a repost 
http://localhost:4002/ob/post
Create a repost

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
BODY raw
{
    "status": "This is a comment",
    "longForm": "",
    "images": [
        {
            "filename": "cat",
            "large": "zb2rhmBUB9i7UkfmeD3obJYK3FFS5K8N8QHaUanG8UWLVBHiY",
            "medium": "zb2rhaFhqziCWk1zo5tMRxQEUchfvJFaGG4DY1anEoR4GnYrN",
            "original": "zb2rhe2o6WbHqcER5VUKsMUbQrmpCC6ihg8qZ4JS9wVgKz9wm",
            "small": "zb2rhbDCeEiTTunugWPaRRKFCfNKUaB7aCR53nrPnMa9usZXY",
            "tiny": "zb2rhgqJDbshwAgPjs7X2h4mDm3V3BpLbp4tFGqkg1LNkg9yV"
        }
    ],
    "tags": [],
    "channel": [],
    "postType": "REPOST",
    "reference": "QmdQBWA75xQSMZpTibQ2G83enNdriz2v14tetGvNrpr5KB/this-is-a-social-post"
}

_____________________________________________________________________________


PUT Update a post 
http://localhost:4002/ob/post
Update a post. The currentSlug field is used specify the listing being updated. If the listing contains a new slug, the listing will be renamed using the new slug.

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
BODY raw
{
	"status": "testy",
	"longForm": "This is a test post dawg.",
	"images": [
		{
			"filename": "cat",
			"large": "zb2rhmBUB9i7UkfmeD3obJYK3FFS5K8N8QHaUanG8UWLVBHiY",
			"medium": "zb2rhaFhqziCWk1zo5tMRxQEUchfvJFaGG4DY1anEoR4GnYrN",
			"original": "zb2rhe2o6WbHqcER5VUKsMUbQrmpCC6ihg8qZ4JS9wVgKz9wm",
			"small": "zb2rhbDCeEiTTunugWPaRRKFCfNKUaB7aCR53nrPnMa9usZXY",
			"tiny": "zb2rhgqJDbshwAgPjs7X2h4mDm3V3BpLbp4tFGqkg1LNkg9yV"
		}
	],
	"tags": [
		"Yo",
		"Yo yo"
	]
}

_____________________________________________________________________________________--


DEL Delete a post 
http://localhost:4002/ob/post/:slug
Delete a post.

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
PATH VARIABLES
slug
______________________________________________________________________________________


profile

API calls for manging your profile and retrieving other profiles

GET Get your profile (ob) 
http://localhost:4002/ob/profile?async=false
Get your profile.

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
Basic base64encode(username + ":" + password)

PARAMS
asyncfalse
(Boolean) Return the profile via websockets.

_____________________________________________________________________________________________



GET Get external profile (ob) 
http://localhost:4002/ob/profile/:peerID?usecache=true
Get an external profile.

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
Basic base64encode(username + ":" + password)

PARAMS
usecachetrue
(Boolean) Use the last fetched profile data stored in cache

PATH VARIABLES
peerIDQmSkkZpNFzWubxE4p9Ejjo2CLqq61jxuRF26ZzCmCKaHcJ
(String) The peerId of the target node.


___________________________________________________________________________


POST Set the profile 
http://localhost:4002/ob/profile
Set the profile for this node

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
BODY raw
{
    "handle": "drwasho",
    "name": "Washington Sanchez",
    "location": "Brisbane",
    "about": "The Dude",
    "shortDescription": "Yo",
    "contactInfo": {
	    "website": "Mylaxx.org",
	    "email": "drwasho@Mylaxx.org",
	    "phoneNumber": "12345"
    },
    "nsfw": false,
    "vendor": true,
    "moderator": false,
    "colors": {
	    "primary": "#000000",
	    "secondary": "#FFD700",
	    "text": "#ffffff",
	    "highlight": "#123ABC",
	    "highlightText": "#DEAD00"
    }
}

________________________________________________________________________________

POST Batch fetch profiles 
http://localhost:4002/ob/fetchprofiles?async=
Fetch multiple profiles

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
PARAMS
async
BODY raw
[
  "QmSQdPxagtLYptbFEnWycdj6rWfkYSfXN68d6ghTX3bByj",
  "QmPUTcwFdCdd4CntWAVcS8Kvg5xvD4X7cfbK8CVYC57JuU",
  "QmSit3mDpKWJi9ChT8UtfvuJYEF6UebeXSUUKzmryCvMgJ",
  "QmdzzGGc9xZq8w4z42vSHe32DZM7VXfDUFEUyfPvYNYhXE",
  "QmRqNRKaqinxpTRJ5JUSzJNSzjdWWTRGy3XxVxdn5h2FnW",
  "QmbeiRH4SsoHqC1SYbCFQxHdiVi24a6FRY4s7yLAXA5ZRM",
  "QmS8xvjUfFdPhx6BpsMr8HNywMAokqTiWcCPHkyQinRUbi",
  "QmTBkgWoN2ERwutZHGN58vdh8faSmsXX2UJJuf23Wewwc2",
  "QmbtjH8WXyj5Fg39cezEHiVtVsh4v6Dnar9WuQdzs3kEdC"
]

________________________________________________________________________


PUT Edit the profile 
http://localhost:4002/ob/profile
Edit the profile for this node

HEADERS
Content-Typeapplication/json
AuthorizationBasic ZHJ3YXNoby10ZXN0LTAwNjpwYXNzd29yZA==
BODY raw
{
  "peerID": "QmVD3WwjZKRwEnczMcVEwEm9Xq9ZKoZpQbDkqPp5Uoi9RZ",
  "handle": "@manbazaar",
  "name": "Men&#39;s Clothing Bazaar",
  "location": "",
  "about": "",
  "shortDescription": "Clothing for men",
  "nsfw": false,
  "vendor": true,
  "moderator": false,
  "contactInfo": {
    "website": "",
    "email": "",
    "phoneNumber": ""
  },
  "colors": {
    "primary": "",
    "secondary": "",
    "text": "",
    "highlight": "",
    "highlightText": ""
  },
  "avatarHashes": {
    "tiny": "Qmckqkv8su5Tw6Styrmzzi3gv78k95iReZrQywb5Rny3gz",
    "small": "QmZ6irSjduA6w3p9EMi7fcZGNdPNYungpxekBAn3PWhDKK",
    "medium": "QmRfe8HTeUqEYj2NvrTo24SZBxUaBganTVSo76PRksFggE",
    "large": "QmYwgMDSYpokfz7seLkj3rtZhat6qJtcvddt1c1XR4s5jL",
    "original": "QmPK8X2FPi9cregxSRvfhPxMVowSLxHBJfdMnyRg1LkyUW"
  },
  "headerHashes": {
    "tiny": "Qmdzo6zCzTUyDdwWC5hg4axXkxPRS7wFiKX6nvcBkRFQUw",
    "small": "QmZpAfKAy4NG9PZfJaSM9ahXS9mQTtjQ8hDNPcHyQJLBps",
    "medium": "QmZirmkZpuLTBUmBU4G5iPZ2ZFLeb3hkmx46mPrWhqRkWR",
    "large": "QmXA6ZSLM9HXCNfyk3QS4z436VeZoZ2sXLWN3NCvwSH6sM",
    "original": "QmVYEs71oQM78ZFWpWD5RX1tYXpjYoSzitvfuyfi8ovQWJ"
  },
  "stats": {
    "followerCount": 0,
    "followingCount": 0,
    "listingCount": 3,
    "ratingCount": 0,
    "averageRating": 0
  },
  "bitcoinPubkey": "0299f8403f38e7af3487faceada4d60919d01ed6ed2b6d5f38e151ffc9c0d71be2",
  "lastModified": "2017-06-08T11:02:38.264019143Z"
}

_______________________________________________________________________________


PATCH Patch the profile 
http://localhost:4002/ob/profile
Patch the profile for this node

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
BODY raw
{
  "peerID": "Qmai9U7856XKgDSvMFExPbQufcsc4ksG779VyG4Md5dn4J",
  "handle": "drwasho",
  "name": "Washington Sanchez",
  "location": "Brisbane",
  "about": "The Dude",
  "shortDescription": "Yo",
  "nsfw": true,
  "vendor": true,
  "moderator": false,
  "contactInfo": {
    "website": "Mylaxx.org",
    "email": "drwasho@Mylaxx.org",
    "phoneNumber": "12345"
  },
  "colors": {
    "primary": "#000000",
    "secondary": "#FFD700",
    "text": "#ffffff",
    "highlight": "#123ABC",
    "highlightText": "#DEAD00"
  },
  "avatarHashes": {
    "tiny": "QmPjkgnvCY8RnYQ5LFBrr36EEzTiagjZjVtxcZ5kV7NFNw",
    "small": "QmPJhf9FQ74qZebKZBXXR81bFNbC1tWPXzTNjmNNm58rES",
    "medium": "QmWdwMSNWFSKYdL8n47dK5EN7DRmPuTzjG12LPqQHAWb6P",
    "large": "QmSReXYhgiWATTuiYWDXzPMDy2k9g3PnM5VmHhwNREVA3B",
    "original": "QmUQmMHqpFR2SN6s5iNktUBHgBnzxpH9862pCDpNKnA5Vd"
  },
  "stats": {
    "followerCount": 0,
    "followingCount": 0,
    "listingCount": 0,
    "ratingCount": 0,
    "averageRating": 0
  },
  "bitcoinPubkey": "03b54f70a26768dfc6335ff1896de9fb34c7c8de98b45141fc264913583b867387",
  "lastModified": "2017-03-24T13:10:53.419703009Z"
}

____________________________________________________________________________________________


rating

API calls that handling blocking a node.

GET Get a rating 
http://localhost:4002/ob/rating/:ratingHash
Get a rating

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
Basic base64encode(username + ":" + password)

PATH VARIABLES
ratingHashzb2rhjyzgRaeomTGLD8P97RC7SRMb9DXEi9wtti9wJ1cuJQ9E
(String) The IPFS hash of the rating object.


___________________________________________________________________________


GET Get ratings of a node 
http://localhost:4002/ob/ratings/:peerId
Get the rating history of a node

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
Basic base64encode(username + ":" + password)

PATH VARIABLES
peerIdQmYEChohLWGxbdgezhwSbaeCsTAG5iimzSkZyhLLUHv2TC
(String) The peerId of the target node.

_______________________________________________________________________--



settings

API calls for handling settings

GET Get the settings 
http://localhost:4002/ob/settings
Fetch the settings from the database

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
Basic base64encode(username + ":" + password)

______________________________________________________________________________



POST Set settings 
http://localhost:4002/ob/settings
Set the settings object in the database

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
Basic base64encode(username + ":" + password)

BODY raw
{
    "paymentDataInQR": true,
    "showNotifications": true,
    "showNsfw": true,
    "shippingAddresses": [
        {
            "name": "Seymour Butts",
            "company": "Globex Corporation",
            "addressLineOne": "31 Spooner Street",
            "addressLineTwo": "Apt. 124",
            "city": "Quahog",
            "state": "RI",
            "country": "UNITED_STATES",
            "addressNotes": "Leave package at back door"
        }
    ],
    "localCurrency": "USD",
    "country": "UNITED_STATES",
    "language": "English",
    "termsAndConditions": "By purchasing this item you agree to the following...",
    "refundPolicy": "All sales are final.",
    "blockedNodes": [],
    "storeModerators": [
        "QmcdkKM2fWCKzTZdpjEY6abzRbDhRRJvNmqwNKVyiDtGky"
    ],
    "smtpSettings": {
        "notifications": false,
        "serverAddress": "smtp.urbanart.com:465",
        "username": "urbanart",
        "password": "letmein",
        "senderEmail": "notifications@urbanart.com",
        "recipientEmail": "Dave@gmail.com"
    }
}

____________________________________________________________________________________


PUT Update settings 
http://localhost:4002/ob/settings
Update the settings object in the database. Overwrites all fields.

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
BODY raw
{
    "paymentDataInQR": true,
    "showNotifications": true,
    "showNsfw": true,
    "shippingAddresses": [
        {
            "name": "Seymour Butts",
            "company": "Globex Corporation",
            "addressLineOne": "31 Spooner Street",
            "addressLineTwo": "Apt. 124",
            "city": "Quahog",
            "state": "RI",
            "country": "UNITED_STATES",
            "addressNotes": "Leave package at back door"
        }
    ],
    "localCurrency": "USD",
    "country": "UNITED_STATES",
    "language": "English",
    "termsAndConditions": "By purchasing this item you agree to the following...",
    "refundPolicy": "All sales are final.",
    "blockedNodes": [],
    "storeModerators": [
        "QmcdkKM2fWCKzTZdpjEY6abzRbDhRRJvNmqwNKVyiDtGky"
    ],
    "smtpSettings": {
        "notifications": false,
        "serverAddress": "smtp.urbanart.com:465",
        "username": "urbanart",
        "password": "letmein",
        "senderEmail": "notifications@urbanart.com",
        "recipientEmail": "Dave@gmail.com"
    }
}

________________________________________________________________________________


PATCH Patch settings 
http://localhost:4002/ob/settings
Update individual settings without overwriting the rest.

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
BODY raw
{
	"localCurrency": "AUD"
}

__________________________________________________________________________________


wallet

Folder for wallet

GET Get a crypto address 
http://localhost:4002/wallet/address
Returns an unused crypto address. Note the same address is returned until the address is used.

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
Basic base64encode(username + ":" + password)

_______________________________________________________________________________



GET Get the mnemonic seed 
http://localhost:4002/wallet/mnemonic
Returns the mnemonic seed used to derive all the bitcoin keys. Note this seed should never be disclosed as it will allow someone to drain your wallet and steal your Mylaxx identity.

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
Basic base64encode(username + ":" + password)

__________________________________________________________________________________


GET Get the wallet's balance 
http://localhost:4002/wallet/balance
Returns the unconfirmed and confirmed balances of the wallet in satoshi (1/100000000 of a full bitcoin).

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
Basic base64encode(username + ":" + password)

_________________________________________________________________________________-



GET Get the wallet transaction history 
http://localhost:4002/wallet/transactions/:currencyCode
Returns the wallet transaction history with associated metadata.

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
Basic base64encode(username + ":" + password)

PATH VARIABLES
currencyCodeBTC
_______________________________________________________________________________


GET Get wallet status 
http://localhost:4002/wallet/status
Returns the last seen block height and hash.

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
Basic base64encode(username + ":" + password)

_______________________________________________________________________________________

GET Estimate fees 
http://localhost:4002/wallet/estimatefee/:coinType?amount=100000000&feeLevel=NORMAL
Estimate the network transaction fees

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
Basic base64encode(username + ":" + password)

PARAMS
amount100000000
feeLevelNORMAL
PATH VARIABLES
coinTypeETH
_________________________________________________________________________________

GET Fetch fee levels 
http://localhost:4002/wallet/fees
Estimate the network transaction fees

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
Basic base64encode(username + ":" + password)

________________________________________________________________________________


GET Get exchange rates 
http://localhost:4002/ob/exchangerate/:currencyCode
Returns either the exchange rate (per BTC) for the given fiat currency or all rates.

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
Basic base64encode(username + ":" + password)

PATH VARIABLES
currencyCodeBTC
(String) The currency code.

_________________________________________________________________________________



GET Get all defined currencies 
http://localhost:4002/wallet/currencies
Get a list of all known currency definitions


_____________________________________________________________________________


GET Estimate specific fee given an amount/priority
http://localhost:4002/wallet/estimatefee/:currencyCode?feeLevel=NORMAL&amount=1300
Estimate the fee given a specific amount of value is being sent. Knowing how much to spend can give a more accurate fee estimation because it changes the number of inputs which may be used.

PARAMS
feeLevelNORMAL
amount1300
PATH VARIABLES
currencyCodeLTC
_____________________________________________________________________

POST Send funds 
http://localhost:4002/wallet/spend
This API is used to send cryptocurrency from the Mylaxx wallet.

This API should not be used to fund Mylaxx orders; instead use POST /ob/orderspend (this is especially important for orders paid in ETH or ERC-20 tokens, as funds may be lost if POST /ob/orderspend is not used).

Fees

Note the feelLevel parameter controls how much of a fee to use.
By default the wallet will query a fee api to get the appropriate fee-per-byte for the given level.
There is a max fee value in the config file.
If the api returns more than the max, then a default fee will be used.
The default values, max values, and api can be configured in the config file.
If you do not wish to use a fee api, and just want to use the default value, set the fee api to "".
HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
Basic base64encode(username + ":" + password)

BODY raw
{
    "amount": "10000",
    "currency": {
        "code": "LTC",
        "divisibility": 8
    },
    "currencyCode": "TLTC",
    "address": "mpu9RjF1p7y3dBAPaDBaUKVphh8DYCbZoq",
    "feeLevel": "NORMAL",
    "memo": "Test spend testnet Litecoin",
    "spendAll": false
}

______________________________________________________________________


POST Bulk update accepted currencies 
http://localhost:4002/ob/bulkupdatecurrency
Set what cryptocurrencies your store will accept.

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
Basic base64encode(username + ":" + password)

BODY raw
{
    "currencies": [
        "BTC",
        "BCH",
        "LTC",
        "ZEC",
        "ETH"
    ]
}

________________________________________________________________________


POST Bump fee 
http://localhost:4002/wallet/bumpfee/:txid
This API call will rebroadcast a transaction with a higher fee, which should be used if a transaction is stuck due to an insufficient fee.

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
Basic base64encode(username + ":" + password)

PATH VARIABLES
txid
(String) The unconfirmed transaction's ID.

_________________________________________________________________________________

POST Re-sync blockchain 
http://localhost:4002/wallet/resyncblockchain
RPC call to re-download the blocks in the blockchain. It is theoretically possible that the wallet could fail to hear of a transaction intended for itself if its peers fail to relay a transaction. In such cases the recourse is to re-download the historical blocks from a new peer to discover the missing transaction(s).

HEADERS
Content-Typeapplication/json
AuthorizationBasic dXNlcm5hbWU6cGFzc3dvcmQ=
Basic base64encode(username + ":" + password)

BODY raw
{}


