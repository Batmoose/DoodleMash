# DoodleMash
Open source Tinder for doodles in the cloud

API usage
========    

The path notation `/some/path` assumes a domain (and port number for testing purposes). The full URL should resemble `<hostname>:<port>/some/path`. Anything in `<>` is to be replaced with the appropriate data upon implementation. Query strings are strings in the format `?key=value&anotherkey=anothervalue`. These inform the server of data you would like to send.

##`/:udid<string>/photo_frags`
###Request - Method: POST
Adds a fragment or set of fragments to be queued for drawing. The fragment should be the raw bytes of the picture.
####Headers
`content-type: application/octet-stream`  
`content-length: <number of bytes in fragment>`
####Query String
`?metadata=<your format of metadata for this fragment>` - append to the end of URL  

This metadata will be associated with the fragment permanently. When a doodle is made of this fragment, **it will be permanently paired with this metadata**.
####Response
#####Status Codes
`200` for successful write. Body contains a message explaining the action performed  
`500` for a failure to write to the queue of photo_frags  
`400` for a malformed request

##`/:udid<string>/doodle_frags`
###Request - Method: GET
####Headers
`accept: application/json`  

Gets references to all pending doodles for the user, with metadata, and access tokens. Response format detailed below.
####Response
#####Headers
`content-type: application/json`  
`content-length: <length of JSON object>`
#####Body

    [
      {
        clusterId = <some string>,
        frags : [
          {
            metadata: <saved metadata from photo upload>,
            accessToken: <token to add as query string when downloading doodle fragment>,
            path: /:udid<string>/doodle_frags/<some id string>
          },
          {
            ...
          }
          ...
        ]
      },
      {
        ...
      }
      ...
    ]

The returned JSON object contains references to a set of doodle fragments. When doodle fragment is downloaded, the server registers that it has been requested. Once all fragments are requested from a cluster, the cluster of fragments will "close," meaning that any future requests for doodle fragments will not longer include that cluster. If the client would like to "revive" a closed cluster, issue a POST request to `/:udid<something>/doodle_frags/revive_cluster/<previous cluster id>` This is detailed later.
#####Status Codes
`200` for having active clusters and returning them
`400` for a malformed request
`403` for nothing to return
`500` for a misc server error

##`/:udid<string>/doodle_frags/<some id string>`
###Request - Method: GET
Gets a doodle fragment.
####Headers
`accept: application/octet-stream`  
`[content-length: <length of query string>]`
####Query String
`?token=<token from accessToken>` - append to the end of URL or put inside request body.  
####Response
#####Headers
`content-type: application/octet-stream`  
`content-length: <length of doodle fragment in bytes>`  

#####Body
The whole doodle fragment is sent if success
#####Status Codes
`200` For doodle exists and you can access it 
`400` For wrong "accepts" field.  
`401` For invalid token, not authorized to access doodle fragment  
`404` For doodle doesn't exist  

##`/:udid<something>/doodle_frags/revive_cluster/<previous cluster id>`
###Request - Method: POST
"Revives" a cluster of "closed" doodle fragments such that it re-appears in server response to `/:udid<string>/doodle_frags`.
####Response
#####Status Codes
`200` For a successful operation  
`404` For a non-existent or no longer valid cluster id  
##`/photo_frags/`
###Request - Method: GET
Gets the reference to a random photo fragment to be drawn
####Headers
`accept: application/json`  
####Response
#####Headers
`content-type: application/json`  
`content-length: <length of json object in bytes>`  
#####Body
    {
        writePath: /doodle_frags/<some doodle frag id>,
        token: <some string>,
        path: /photo_frags/<some photo frag id>,
        metadata: <string of metadata>
    }
    
Issue a get request to the returned path to receive the raw photo. Save the token to be submitted with the doodle as well. Details below.
##`/photo_frags/<some photo frag id>`
###Request - Method: GET
####Headers
`accept: application/octet-stream`
####Query String
`?token=<token as returned by request to /photo_frags/>`
####Response
#####Headers
`content-type: application/octet-stream`  
`content-length: <length of image fragment in bytes>`  
#####Body
The raw contents of the photo fragment.
##`/doodle_frags/<some doodle frag id>`
###Request - Method: POST
####Headers
`content-type: application/octet-stream`  
`content-length: <size of doodle fragment uploaded in bytes>`
####Query String
`?token=<token as returned by a get request to /photo_frags/` - append to the end of URL
####Body
The raw contents of a doodle fragment to be uploaded
