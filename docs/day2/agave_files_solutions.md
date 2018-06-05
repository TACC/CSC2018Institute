# Solutions to the Agave files exercises

0. set up the environment
```
$ export base=https://api.tacc.utexas.edu
$ export tok=<your_oauth_token>
```

1. list contents of home directory
```
$ curl -H "Authorization: Bearer $tok" $base/files/v2/listings/system/csc.2018.storage/<username>
```

2. Create a directory within home directory
```
$ curl -X PUT -d "action=mkdir&path=test" -H "Authorization: Bearer $tok" $base/files/v2/media/system/csc.2018.storage/<username>
```

3. Upload a text file to the test directory
```
$ curl -F "fileToUpload=@path/to/test.txt" -H "Authorization: Bearer $tok" $base/files/v2/media/system/csc.2018.storage/<username>/test
```

Example API response:
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
      "source":"http://129.114.97.130/test.txt",
      "path":"apitest/test1/test.txt",
      "status":"STAGING_QUEUED",
      "systemId":"csc.2018.storage",
      "nativeFormat":"raw",
      "_links":{  
         "self":{  
            "href":"https://api.tacc.utexas.edu/files/v2/media/system/csc.2018.storage//apitest/test1/test.txt"
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

4. download the text file, redirect stdout to a text file
```
$ curl -H "Authorization: Bearer $tok" $base/files/v2/media/system/csc.2018.storage/apitest/test1/test.txt > test.txt
```
