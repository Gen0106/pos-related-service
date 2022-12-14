# pos-related-service
This is a system which provide simple POS related services(Restful APIs, GraphQL)

Let's suppose we manage a POS integrated e-commerce platform which offers a point system and has a vast collection of payment methods integrated. 
Some payment methods require a commission fee from the payment provider thus you do not want to provide too much discount on a product if the customer selects that payment method. 
At the same time, you would want to control the points given per purchase based on the payment methods to minimize loss. 
The following is the list of possible payment methods, rates, points, and additional actions that will be performed based on the payment method.

| Payment Method | Possible final price | Points | Additional items |
| --- | --- | --- | --- |
| CASH | price * 0.9 ~ price | price * 0.05 |  |
| CASH_ON_DELIVERY | price ~ price * 1.02 | price * 0.05 | Should accept the following courier services only <br>● YAMATO<br>● SAGAWA |
| VISA | price * 0.95 ~ price * 1 | price * 0.03 | Should accept and store last 4 digits of the card |
| MASTERCARD | price * 0.95 ~ price * 1 | price * 0.03 | Should accept and store last 4 digits of the card |
| AMEX | price * 0.98 ~ price * 1.01 | price * 0.02 | Should accept and store last 4 digits of the card |
| JCB | price * 0.95 ~ price * 1 | price * 0.05 | Should accept and store last 4 digits of the card |
| LINE PAY | price | price * 0.01 |  |
| PAYPAY | price | price * 0.01 |  |
| POINTS | price | 0 |  |
| GRAB PAY | price | price * 0.01 |  |
| BANK TRANSFER | price | 0 | Should accept and store bank and account number information |
| CHEQUE | price * 0.9 ~ price | 0 | Should accept and store bank and cheque number information |

Shops and online stores will integrate with this system. To make a payment, the following requests and response should be provided

Request
```json
{
  "customer_id": "12345",
  "price": "100.00",
  "price_modifier": 0.95,
  "payment_method": "MASTERCARD",
  "datetime": "2022-09-01T00:00:00Z",
  "additional_item": {
      "last_4": "1234"
  }
}
```

Response
```json
{
  "final_price": "95.00",
  "points": 5
}
```

The calculation formula is likes the following
final_price = 100 * 0.95 = 95
points = 100 * 0.05 = 5

If the input is invalid, this system responds an error or error list.

This system allow the users of your e-commerce system to see how much sales were made within a date range broken down into hours. 
This system shows a list of sales and the points given out to the customer.

Request
```json
{
  "startDateTime":
  "2022-09-01T00:00:00Z",
  "endDateTime":
  "2022-09-01T23:59:59Z"
}
```

Response: 
```json
{
  "sales": [
  {
    "datetime": "2022-09-01T00:00:00Z",
    "sales": "1000.00",
    "points": 10
  },
  {
    "datetime": "2022-09-01T01:00:00Z",
    "sales": "2000.00",
    "points": 20
  },
  {
    "datetime": "2022-09-02T00:00:00Z",
    "sales": "5000.00",
    "points": 75
  },
  {
    "datetime": "2022-09-01T23:00:00Z",
    "sales" :"7000.00",
    "points": 30
  }
  ]
}
```

## Install Method
### Prerequisite
1. Java 11
2. Apache Maven 3.8.6

### Installing on local repository
In the project root folder
```
$ mvn clean
$ mvn compile
$ mvn install
```

### Excuting on the local
In the project root folder
```
$ mvn clean
$ mvn compile
$ mvn package
$ cd target
$ java -jar pos-related-service-0.0.1-SNAPSHOT.jar
```
#### Web APIs
**localhost:9000/save-sale**
 
Request format
```json
{
  "customer_id": "12234",
  "price": "100",
  "price_modifier": "0.95",
  "payment_method": "MASTERCARD",
  "datetime": "2022-11-20T22:10:39Z",
  "additional_item": {
    "last_4": "1111"
  }
}
```
Response format
```json
{
  "success": true,
  "result": {
    "final_price": "95",
    "points": "5"
  },
  "errList": []
}
```
<br>

**localhost:9000/all-sales**

Response format
```json
{
  "success": true,
  "result":[{
      "customer_id": "12234",
      "payment_method": "MASTERCARD",
      "sales": "95",
      "points": "5",
      "datetime": "2022-11-20T22:10:39Z",
      "additional_item": {
        "last_4": "1111"
      }
    },{
        "customer_id": "12234",
        "payment_method": "CASH",
        "sales": "98",
        "points": "0",
        "datetime": "2022-11-23T22:10:39Z",
        "additional_item": {
        }       
    }],
    "errList": []
  }
```


localhost:9000/sales-by-time-range

Request Url Param
```
start_time: 2022-10-20T00:00:00Z
end_time: 2022-11-22T23:59:59Z
```

Response format
```json
{
  "success": true,
  "result":[{
      "customer_id": "12234",
      "payment_method": "MASTERCARD",
      "sales": "95",
      "points": "5",
      "datetime": "2022-11-20T22:10:39Z",
      "additional_item": {
        "last_4": "1111"
      }
    },{
        "customer_id": "12234",
        "payment_method": "CASH",
        "sales": "98",
        "points": "0",
        "datetime": "2022-11-23T22:10:39Z",
        "additional_item": {
        }       
    }],
    "errList": []
  }
```
<br>

#### GraphQL
**url: localhost:9000/graphql**

Save Query And Variables Format
```
mutation($input: SaleInfoGraphQLRequest) {
    saveSaleInfo(input: $input) {
        final_price,
        points, 
        errors
    }
}
```

```json
{
    "input": {
        "customer_id": "12345",
        "price": 100.00,
        "payment_method": "PAYPAY",
        "datetime": "2022-09-07T00:00:00Z"
    }
}
```

All Sale Query
```
query {
    allSales {
        customer_id,
        payment_method,
        sales,
        points,
        datetime,
        additional_item
    }
}
```

Sale Query And Variable Format By Time Range
```
query($start_time: String, $end_time: String) {
    salesByTimeRange(start_time: $start_time, end_time: $end_time) {
        customer_id,
        payment_method,
        sales,
        points
    }
}
```
```json
{
    "start_time": "2022-06-10T11:11:11Z",
    "end_time": "2022-12-10T11:11:11Z"
}
```
