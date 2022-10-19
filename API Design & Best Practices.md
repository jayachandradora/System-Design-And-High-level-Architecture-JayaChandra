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

## 7. Versioning

## 8. API Documentation

## 9. Using SSL/TLS
