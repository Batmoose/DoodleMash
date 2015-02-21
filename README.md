# DoodleMash
Open source Tinder for doodles in the cloud

API usage
========    

The path notation `/some/path` assumes a domain (and port number for testing purposes). The full URL should resemble `<hostname>:<port>/some/path`. Anything in `<>` is to be replaced with the appropriate data upon implementation. Query strings are strings in the format `?key=value&anotherkey=anothervalue`. These inform the server of data you would like to send.
##`/:udid<string>/photo_frags`
###Request
####Method: POST
Adds a fragment or set of fragments to be queued for drawing. The fragment should be the raw bytes of the picture.
#####Headers
`content-type: application/octet-stream`  
`content-length: <number of bytes in fragment>`
#####Query String
`?metadata=<your format of metadata for this fragment>` - append to the end of URL  

This metadata will be associated with the fragment permanently. When a doodle is made of this fragment, **it will be permanently paired with this metadata**.
###Response
#####Status Codes
`200` for successful write. Body contains a message explaining the action performed  
`500` for a failure to write to the queue of photo_frags  
`400` for a malformed request
##`/:udid<string>/doodle_frags`
##Request
####Method: GET
#####Headers
`accept: application/json`  

Gets references to all pending doodles for the user, with metadata, and access tokens. Response format detailed below.
###Response
#####Headers
`content-type: application/json`  
`content-length: <length of JSON object>`
#####Body

    [
      {
        id = <some string>,
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
      }
      {
        ...
      }
      ...
    ]

The returned JSON object contains references to a set of doodle fragments. When doodle fragment is downloaded, the server registers that it has been requested. Once all fragments are requested from a cluster, the cluster of fragments will "close," meaning that any future requests for doodle fragments will not longer include that cluster. If the client would like to "revive" a closed cluster, issue a POST request to /:udid<something>/doodle_frags/revive_cluster/:id<previous cluster id>. This is detailed later.

