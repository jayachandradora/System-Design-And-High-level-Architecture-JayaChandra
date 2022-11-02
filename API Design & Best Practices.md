# System-Design-And-High-level-Architecture-JayaChandra
Software Design Architecture - High Level Design

# API Design & Best Practices

## 1. Use JSON

There are lots of data formats we can use like JSON, XML, HTML, CSV, Plain Text etc. But mostly used and prety format is JSON for best practuce. Also JSON is mostly human readability formats. This is very light weight.<br>

## 2. Use Nouns instead of Verbs
In REST API's we should not use vers or actions. Allways use nouns end points name of a resources. <br>

Good to put resource names in small letters.

#### Formats
```ruby
Action : Method Name - GET, POST, PUT, DELETE, PATCH etc.
End Points: www.xyz.com/api/1.0/users
```
#### Ex. 
```ruby
API Details: Get All users details 
Action: GET <br>
End Points: www.xyz.com/api/1.0/users 

API Details: Create users 
Action: POST<br>
End Points: www.xyz.com/api/1.0/users 
"Body:" {
        "id":111
        "name": "JayaChandra"
        "Address": "Bangalore"
      }

API Details: Delete users
Action: DELETE
End Points: www.xyz.com/api/1.0/users/123
```

#### Bad Practices API Design
```ruby
End Points: Ex
www.xyz.com/api/1.0/addUser 
www.xyz.com/api/1.0/deleteBooks
www.xyz.com/api/1.0/deleteBooks/123
```

## 3. Name the collections using Plural Nouns

```ruby
API Details: Get a specific user
Action: GET
End Points: www.xyz.com/api/1.0/users/123   Just remember dont use "user". Its "users"

API Details: Delete specific User
Action: DELETE
End Points: www.xyz.com/api/1.0/users/123/
```

## 4. Use resource nesting
```ruby
API Details: Get Users all Orders
Action: GET
End Points: www.xyz.com/api/1.0/users/123/orders

API Details: Get Users Specific Orders 
Action: GET
End Points: www.xyz.com/api/1.0/users/123/orders/o123 
```

## 5. Error Handling

#### HTTP Status Codes

![image](https://user-images.githubusercontent.com/115500959/196740878-3e1e90a9-9739-4376-9341-8d6acb2e132f.png)


## 6. Filtering, sorting, paging, and field selection

All these are done on collections. <br>

```ruby
Ex. 
API Details: Filter based on language
URI: www.bookmyshow.com/api/1.0/filim
Action: Method - GET
Query Param: language : English
             limit : 5 - limit to 5 records 
             sort : sort the records on release date or sort based on theater or sort based on name ascending or descending order etc.
             paging : start offset and limit of records etc.
```
## 7. Versioning

Version could 1.0, v1, release date or Sprint release number etc.
Ex.
```ruby
www.bookmyshow.com/api/1.0/filim
www.bookmyshow.com/api/v1/filim 
www.bookmyshow.com/api/2022_09_19/filim 
```

## 8. API Documentation

Please create good documentation with these resouces or end points so that consumers will better understand and will not miss use any time. <br>
Incase you are using java spring boot as a framework then there is a good documentation called "Swagger" UI available from Netflix please use that.<br>

## 9. Using SSL/TLS

Don't compromise on data with communication protocol and no exception with your data. So, allways use "HTTPS".
