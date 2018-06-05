# set up the environment
export base=https://api.tacc.utexas.edu
export tok=<your_token>

# add permission
ag.files.updatePermissions(systemId='csc.2018.storage', filePath=users[11], body={'username': 'apitest', 'permission': 'WRITE', 'recursive': True})

# list permissions
ag.files.listPermissions(systemId='csc.2018.storage', filePath=users[i])

# list contents
curl -H "Authorization: Bearer $tok" $base/files/v2/listings/system/csc.2018.storage/<username>

# create a directory
curl -X PUT -d "action=mkdir&path=test" -H "Authorization: Bearer $tok" $base/files/v2/media/system/csc.2018.storage/<username>

# upload a text file
curl -F "fileToUpload=@path/to/test.txt" -H "Authorization: Bearer $tok" $base/files/v2/media/system/csc.2018.storage/<username>/test

Example response:
```
{  
   "status":"success",
   "message":null,
   "version":"2.2.20-r7f2871d",
   "result":{  
      "name":"test.txt",
      "uuid":"2161305060668019176-242ac113-0001-002",
      "owner":"apitest",
      "internalUsername":null,
      "lastModified":"2018-06-04T20:23:56.926-05:00",
      "source":"http://129.114.97.130/pip-selfcheck.json",
      "path":"apitest/test1/test.txt",
      "status":"STAGING_QUEUED",
      "systemId":"csc.2018.storage",
      "nativeFormat":"raw",
      "_links":{  
         "self":{  
            "href":"https://api.tacc.utexas.edu/files/v2/media/system/csc.2018.storage//apitest/test1/pip-selfcheck.json"
         },
         "system":{  
            "href":"https://api.tacc.utexas.edu/systems/v2/csc.2018.storage"
         },
         "profile":{  
            "href":"https://api.tacc.utexas.edu/profiles/v2/apitest"
         },
         "history":{  
            "href":"https://api.tacc.utexas.edu/files/v2/history/system/csc.2018.storage//apitest/test1/test.txt"
         },
         "notification":[  

         ]
      }
   }
}
```

# download the text file, redirect stdout to a text file
curl -H "Authorization: Bearer $tok" $base/files/v2/media/system/csc.2018.storage/apitest/test1/test.txt > test.txt