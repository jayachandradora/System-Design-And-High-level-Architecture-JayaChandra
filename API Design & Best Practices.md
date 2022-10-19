# System-Design-And-High-level-Architecture-JayaChandra
Software Design Architecture - High Level Design

# API Design & Best Practices

## 1. Use JSON

There are lots of data formats we can use like JSON, XML, HTML, CSV, Plain Text etc. But mostly used and prety format is JSON for best practuce. Also JSON is mostly human readability formats. This is very light weight.<br> 

## 2. Use Nouns instead of Verbs
In REST API's we should not use vers or actions. Allways use nouns end points name of a resources.<br>

Good to put resource names in small letters.<br>

#### Formats
Action : Method Name - GET, POST, PUT, DELETE, PATCH etc.
End Points: www.xyz.com/api/1.0/users

#### Ex. 
API Details: Get All users details <br>
Action: GET <br>
End Points: www.xyz.com/api/1.0/users <br>

API Details: Create users <br>
Action: POST<br>
End Points: www.xyz.com/api/1.0/users <br>
*Body:* {<br>
        "id":111<br>
        "name": "JayaChandra"<br>
        "Address": "Bangalore"<br>
      }<br>

API Details: Delete users <br>
Action: DELETE<br>
End Points: www.xyz.com/api/1.0/users/123 <br>

#### Bad Practices API Design
End Points: Ex<br>
www.xyz.com/api/1.0/addUser <br>
www.xyz.com/api/1.0/deleteBooks<br>
www.xyz.com/api/1.0/deleteBooks/123<br>

## 3. Name the collections using Plural Nouns

API Details: Get a specific user <br>
Action: GET<br>
End Points: www.xyz.com/api/1.0/users/123 <br>  Just remember dont use "user". Its "users"

API Details: Delete specific User <br>
Action: DELETE<br>
End Points: www.xyz.com/api/1.0/users/123/ <br>

## 4. Use resource nesting

API Details: Get Users all Orders <br>
Action: GET<br>
End Points: www.xyz.com/api/1.0/users/123/orders <br>

API Details: Get Users Specific Orders <br>
Action: GET<br>
End Points: www.xyz.com/api/1.0/users/123/orders/o123 <br>

## 5. Error Handling

#### HTTP Status Codes

![image](https://user-images.githubusercontent.com/115500959/196740878-3e1e90a9-9739-4376-9341-8d6acb2e132f.png)


## 6. Filtering, sorting, paging, and field selection

All these are done on collections. <br>

Ex. <br>
API Details: Filter based on language<br>
URI: www.bookmyshow.com/api/1.0/filim <br>
Action: Method - GET <br>
Query Param: language : English <br>
             limit : 5 - limit to 5 records <br>
             sort : sort the records on release date or sort based on theater or sort based on name ascending or descending order etc. <br>
             paging : start offset and limit of records etc. <br>

## 7. Versioning

Version could 1.0, v1, release date or Sprint release number etc.<br>
Ex.<br>

www.bookmyshow.com/api/1.0/filim <br>
www.bookmyshow.com/api/v1/filim <br>
www.bookmyshow.com/api/2022_09_19/filim <br>

## 8. API Documentation

Please create good documentation with these resouces or end points so that consumers will better understand and will not miss use any time.
Incase you are using java spring boot as a framework then there is a good documentation called "Swagger" UI available from Netflix please use that.

## 9. Using SSL/TLS

Don't compromise on data with communication protocol and no exception with your data. So, allways use "HTTPS".
