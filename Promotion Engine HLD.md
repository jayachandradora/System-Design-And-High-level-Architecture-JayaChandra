# Software-Design-Architecture-JayaChandra
Software Design Architecture - High Level Design

## Promotion Engine Use Case
We need you to implement a simple promotion engine for a checkout process. Our Cart contains a list of single character SKU ids (A, B, C.	) over which the promotion engine will need to run.
The promotion engine will need to calculate the total order value after applying the 2 promotion types
buy 'n' items of a SKU for a fixed price (3 A's for 130)
buy SKU 1 & SKU 2 for a fixed price ( C + D = 30 )
The promotion engine should be modular to allow for more promotion types to be added at a later date (e.g. a future promotion could be x% of a SKU unit price). For this coding exercise you can assume that the promotions will be mutually exclusive; in other words if one is applied the other promotions will not apply

## Promotion Engine High Level Design

![image](https://user-images.githubusercontent.com/115500959/194984426-0eb35ced-1be7-4d54-9b46-de9441c69856.png)

Will ensure API Response would have in Milliseconds(SLA) not even secodonds.

## Promotion Request & Response Payload
### Apply Promotion for Bundle Product
##### Method : POST
##### URI : www.pricingengine.com/api/apply/promotion/bundleproduct
##### Request Payload
{<br>
 	items : ["C", "D"],<br>
 	promotedPrice: 30.0<br>
}
##### Response Payload
{<br>
 	items : ["C", "D"],<br>
 	promotedPrice: 30.0,
 	message: "Promotion applied successfully for bundle product"<br>
}

### Apply Promotion for Single Product(A) with Multiple Quantity
##### Method : POST
##### URI : www.pricingengine.com/api/apply/promotion/singleproductgroup
##### Request Payload
{<br>
 	"appliedItem" : "A",<br>
 	"quota" : 3,<br>
 	"promotedPrice": 30.0<br>
}
##### Response Payload
{<br>
 	"appliedItem" : "A",<br>
 	"quota" : 3,<br>
 	"promotedPrice": 30.0,<br>
 	message: "Promotion applied successfully for single product - A  with multi QTY"<br>
}

### Apply Promotion for Single Product(B) with Multiple Quantity
##### Method : POST
##### URI : www.pricingengine.com/api/apply/promotion/singleproductgroup
##### Request Payload
{<br>
 	"appliedItem" : "B",<br>
 	"quota" : 2,<br>
 	"promotedPrice": 45.0<br>
}
##### Response Payload
{<br>
 	"appliedItem" : "B",<br>
 	"quota" : 2,<br>
 	"promotedPrice": 45.0,<br>
 	message: "Promotion applied successfully for single product - B with multi QTY"<br>
}

## Test Setup
### Unit price for SKU IDs 
A	50<br>
B	30<br>
C	20<br>
D	15<br>
<br>
### Active Promotions
3 of A's for 130
2 of B's for 45 
C & D for 30
<br>
#### Scenario A
1 * A 50<br>
1 * B 30<br>
1 * C 20<br>
Total 100<br>
<br>
#### Scenario B
5 * A <b>130</b> + 2*50<br>
5 * B <b>45</b> + <b>45</b> + 30<br>
1 * C 20<br>
Total 370<br>
<br>
#### Scenario C
3 * A 130<br>
5 * B <b>45</b> + <b>45</b> + 30<br>
1 * C -<br>
1 * D <b>30</b><br>
Total	280<br>

#### Extending to this project 
1. Create API for all these functionalities.<br>
2. Keep track of applied promotion in user history level.<br>
3. Orchastrate the API while adding item to the cart and promotions should apply based on knowledge driven approach instead of Event driven approach.<br>
4. All these services should have single data base as a service.
5. etc....
