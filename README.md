#test README.MD

preconditions

password hashing application downloaded from public s3 bucket (wget --no-check-certificate --no-proxy
https://s3.amazonaws.com/qa-broken-hashserve/broken-hashserve.tgz)
PORT set to 8088

test cases

Verify hashing application answers on PORT 8088 (echo $PORT) - pass
Verify POST to /hash endpoint returns job identifier immediately - fail
Verify POST to /hash endpoint time to compute password hash = 5 seconds (5000 ms) - pass
Verify POST to /hash endpoint with environment variable 8088 returns 201 - fail
Verify POST to /hash endpoint with environment variable 8089 returns 'connection refused' - pass 
Verify POST to /hash endpoint supports special characters - pass
Verify POST to /hash endpoint supports integers - pass
Verify POST to /hash endpoint supports int + special characters - pass
Verify POST to /hash endpoint supports spaces - pass
Verify Min and Max input in POST request - fail 
Verify behavior with empty request - pass



Verify GET to /hash with valid job identifier returns 200 - pass
Verify GET to /hash with valid job identifier from corresponding POST response returns base64 encoded password hash - pass
Verify GET to /hash with invalid job ID returns 404 not found status code - fail 
Verify base64 password hash is encrypted - pass

Verify GET /stats _TotalRequests_ value is acurate from when the server was started - pass
Verify GET /stats _AverageTime_ value is in milliseconds - fail
Verify GET /stats returns a JSON schema - pass
Verify GET /stats returns a 404 not found when data is passed - pass 


Verify API supports multiple connections simultaneously - fail
Verify application gracefully handles shutdown when in-flight requests are being processed - fail
Verify API requests are rejected when shutdown is pending - pass
Verfify endpoints rejects new responses when AUT is shutdown - pass
Verify POST /shutdown returns a 200 with empty response - pass


Feature defects

**Defect#1**

Description - POST/hash does not return job_id immediately, currently the job_id is returned when password hash is computed. Per AC _AUT should return job identifier immediately_

endpoint in test - POST/http://127.0.0.1:8088/hash
Envrionment - 8088
Version - broken_hashserve_darwin

Repro steps:

1. Launch hash serve application
2. Launch postman
3. Within POST request enter JSON body {"password":"test1"}
4. Execute POST/http://127.0.0.1:8088/hash
5. Verify job identifier is not returned immediately in POST response
6. Verify job identifier is returned upon hash compute is complete

![image](https://user-images.githubusercontent.com/87154858/142806289-1cd51c9f-b06c-4e15-b60d-7b3572c68e34.png)



**Defect#2**

Description - POST/hash does not return the correct status code when creating hash record, please update status code from 200 to 201.

endpoint in test - POST/http://127.0.0.1:8088/hash
Envrionment - 8088
Version - broken_hashserve_darwin

Repro steps:

1. Launch hash serve application
2. Launch postman
3. Within POST request body, enter {"password":"test2"}
4. Execute POST/http://127.0.0.1:8088/hash
5. Verify response code 200 is returned

![image](https://user-images.githubusercontent.com/87154858/142806350-73a3cfff-a97f-4aea-b369-d9b22de996a4.png)


**Defect#3**

Description - Please add min and max character validation to password input for endpoint POST/http://127.0.0.1:8088/hash; per discussion with PO, maximum password length will be 128 characters and minimum of 8 characters. 

endpoint in test - POST/http://127.0.0.1:8088/hash
Envrionment - 8088
Version - broken_hashserve_darwin

Repro steps:

1. Launch hash serve application
2. Launch postman
3. Execute POST/http://127.0.0.1:8088/hash request when password <8 characters
4. Verify 200 response with job_id
5. Execute request when password >128 characters
6. Verify 200 response with job_id 


Expected results (min) - 400 bad request; "password too short"
Expected results (max) - 400 bad request; "password too long"


**Defect#4**

Issue - GET/hash request with invalid job_id returns status code 400 with "hash not found" error message, please update to 404 not found status code.

endpoint in test - GET/http://127.0.0.1:8088/hash/id
Envrionment - 8088
Version - broken_hashserve_darwin

Repro steps

1. Launch hash serve application
2. Launch postman
3. Execute GET/hash with invalid job_id; example http://127.0.0.1:8088/hash/999)
4. Verify 400 status code response with "hash not found" error

Expected - 404 not found

![image](https://user-images.githubusercontent.com/87154858/142810424-d608b7da-3fe6-402d-9cfa-401986d93046.png)


**Defect#5**

Issue - GET/stats response "AverageTime" is not in milliseconds 

endpoint in test - GET/http://127.0.0.1:8088/stats
Envrionment - 8088
Version - broken_hashserve_darwin

Repro steps

1. Launch hash serve application
2. Launch postman
3. Execute POST/http://127.0.0.1:8088/hash with valid password
4. Verify 200 with job_id is returned
5. Execute GET/http://127.0.0.1:8088/stats
6. Verify "AverageTime" in milliseconds = false

![image](https://user-images.githubusercontent.com/87154858/142811042-2fe7311f-b243-4c60-8be6-704d961eca9d.png)


**Defect#6**

Issue - AUT does not support multiple connection simultaneously

Envrionment - 8088
Version - broken_hashserve_darwin

Repro steps

1. Lanuch terminal
2. Set PORT environment variable to 8088
3. Launch application ./broken-hashserve/broken-hashserve_darwin
4. Open new terminal window
5. Set PORT environment variable to 8088
6. Launch application ./broken-hashserve/broken-hashserve_darwin
7. Verify error "2021/11/21 23:31:15 listen tcp :8088: bind: address already in use"


![image](https://user-images.githubusercontent.com/87154858/142814647-a3463e29-da51-414f-a4ac-115a378a7633.png)

**Defect#7**

Issue - POST/shutdown does not gracefully handle shutdown when in-flight requests are being processed
Envrionment - 8088
Version - broken_hashserve_darwin

Repro steps

1. Launch hash serve application
2. Launch postman
3. Create a new collection with POST/hash endpoint only
4. Launch 'Runner' within postman (top left)
5. Select API collection with post/hash endpoint only
6. Enter interation value = 20
7. Click 'Run {endpoint}'
8. Navigate to postman collection
9. Exectue POST/shutdown
10. Verify shutdown does not gracefully handle in-flight requests and indicate requests are being processed before shutting down application

![image](https://user-images.githubusercontent.com/87154858/142817755-1ba629b6-2ea1-4f26-b844-03f9d80666d2.png)

![image](https://user-images.githubusercontent.com/87154858/142817894-1722a8e3-a165-455e-866e-0fce8374c4a9.png)

6. Enter interation value = 20
