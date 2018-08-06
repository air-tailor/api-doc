# api-doc

# Revolve / Air Tailor Integration

This documentation will serve the purpose of explaining how to use the Air Tailor API to place orders on behalf of Revolve customers. 

Once an order is placed successfully, a shipping label and order confirmation email will be sent to the customer so they can send their order to the tailor. 

##### Prerequisites
In order to make a successful request you must first register your company, store, and user account in our system. Morgan Halberg will help get you set up with this.

##### Testing 
We strongly reccomend utilizing a mock to simulate the response from our API as you build out the client facing part of the integration. If needed, you can hit our sandbox API to confirm and record responses.

Sandbox Url: 'https://sandbox-airtailor-api.herokuapp.com/'

##### First Request
A request to the Air Tailor API should include:
* API Key in header 
* Order information
  * Each item that needs altered
    * Each alteration for a given item
* The customer's information

```
// route 
POST `/api/v1/orders`

// headers 
{ 
	"X-Api-Key": <YOUR API KEY>,
	"Content-Type": "application/json"
}

// body 
{
  "order": {
    "requester_notes": "these are notes the customer or revolve can leave",
    "items": [
      {
        "item_type_id": 7,
        "alterations": [
          { "alteration_id": 208 },
          { "alteration_id": 219 }
        ]
      },
      {
        "item_type_id": 6,
        "alterations": [
          { "alteration_id": 36 }
        ]
      }
    ],
    "customer": {
      "first_name": "Bob",
      "last_name": "Builder",
      "phone": "1234567890",
      "email": "bob@builder.com",
      "street": "123 B St",
      "street_two": "Apt 2R",
      "city": "New York",
      "state_province": "NY",
      "zip_code": "10031"
    }
  }
}
```

Optional Fields:
* requester_notes
* customer.street_two

##### API Responses 

1. If order placed successfully:
	* 200 ok, responds with json object representing created order including order id 
2. If API key is not authenticated
	* 401 forbiden, "Access Denied"
3. If client submits an item_type_id that does not exist in our system 
	* 404 not found, "Couldn't find ItemType with 'id'=7000"
4. If client submits an alteration_id that does not exist in our system 
	* 404 not found, "Couldn't find Alteration with 'id'=7000"
5. If required customer data is not included in request 
	* 422 unprocessable entity, "First name cannot be blank"
6. If order object is not included or empty in request 
	* 422 unprocessable entity, "param is missing or the value is empty: order"
6. If customer object is not included or empty in request 
	* 422 unprocessable entity, "param is missing or the value is empty: customer"
7. If items array is not included or empty in request
	* 422 unprocessable entity, "param is missing or the value is empty: items"
7. If alterations array is not included or empty in request
	* 422 unprocessable entity, "param is missing or the value is empty: alterations"
8. If any required address fields are not included 
	* 422 unprocessable entity, "Invalid Address"

Note: Errors will be contained in an array. Example error response:
```
{"errors":["Invalid Address"]}
```

##### Items and Alteration Options

These have the actual id numbers for the items and alterations:

Note - Included with the alterations is an image and instructions (if needed) to assist the Revolve customer pin their clothing.

```
// item_type list
[
  {
    id: 6,
    name: 'Pants',
  },
  {
    id: 7,
    name: 'Shirt',
  },
  {
    id: 10,
    name: 'Dress',
  },
  {
    id: 13,
    name: 'Skirt',
  },
  {
    id: 12,
    name: 'Suit Jacket',
  },
  {
    id: 14,
    name: 'Necktie',
  },
]
```



```
// alterations list

// note - users should not be able to select multiple alterations
// with the same `type` on a single garment. For example - for
// one pair of pants, users should not be able to select both
// Regular and Original Hem

// the how to pin images and instructions can be displayed
// to make pinning the images easy for the customer

[
  {
    id: 16,
    item_type_id: 6,
    title: 'Shorten Pant Length - Regular Hem',
    price: 15.5,
    how_to_pin_image:
      'https://s3.us-east-2.amazonaws.com/airtailor-images/new_how_to_pin/1.jpg',
    instructions:
      'Place safety pin at desired pant length. Pin both legs for different lengths.',
    type: 'pantsHem',
  },
  {
    id: 36,
    item_type_id: 6,
    title: 'Shorten Pant Length - Original Hem',
    price: 24.0,
    how_to_pin_image:
      'https://s3.us-east-2.amazonaws.com/airtailor-images/new_how_to_pin/1.jpg',
    instructions:
      'Place safety pin at desired pant length. Pin both legs for different lengths.',
    type: 'pantsHem',
  },
  {
    id: 208,
    item_type_id: 7,
    title: 'Shorten Shirt Sleeves — Without Cuff',
    price: 15.5,
    how_to_pin_image:
      'https://s3.us-east-2.amazonaws.com/airtailor-images/new_how_to_pin/8.jpg',
    instructions:
      'Place safety pin at desired sleeve length. Pin both sleeves for different lengths.',
    type: 'sleevesTakenUp',
  },
  {
    id: 210,
    item_type_id: 7,
    title: 'Shorten Shirt Straps',
    price: 15.5,
    how_to_pin_image:
      'https://s3.us-east-2.amazonaws.com/airtailor-images/new_how_to_pin/43.jpg',
    instructions:
      'Pinch excess fabric on shoulder straps to desired fit and safety pin the fabric together. Make sure bust still fits comfortably.',
    type: 'sleevesTakenUp',
  },
  {
    id: 219,
    item_type_id: 7,
    title: 'Shorten Dress Shirt (Hem)',
    price: 24.0,
    how_to_pin_image:
      'https://s3.us-east-2.amazonaws.com/airtailor-images/new_how_to_pin/11.jpg',
    instructions:
      'Place safety pin at desired shirt length. Use multiple pins to outline a new shape, or one pin to maintain the current shape.',
    type: 'skirtLength',
  },
  {
    id: 217,
    item_type_id: 13,
    title: 'Shorten Skirt (Hem) — Single Layer',
    price: 30.0,
    how_to_pin_image:
      'https://s3.us-east-2.amazonaws.com/airtailor-images/new_how_to_pin/14.jpg',
    instructions:
      'Place safety pin at desired skirt length. Use multiple pins to outline a new shape, or one pin to maintain the current shape. If multiple layers, just mark outer layer.',
    type: 'skirtLength',
  },
  {
    id: 218,
    item_type_id: 7,
    title: 'Shorten T-Shirt (Hem)',
    price: 15.5,
    how_to_pin_image:
      'https://s3.us-east-2.amazonaws.com/airtailor-images/new_how_to_pin/12.jpg',
    instructions:
      'Place safety pin at desired shirt length. Use multiple pins to outline a new shape, or one pin to maintain the current shape.',
    type: 'shirtLength',
  },
  {
    id: 211,
    item_type_id: 10,
    title: 'Shorten Dress (Hem) — Single Layer',
    price: 30.0,
    how_to_pin_image:
      'https://s3.us-east-2.amazonaws.com/airtailor-images/new_how_to_pin/19.jpg',
    instructions:
      'Place safety pin at desired dress length. Use multiple pins to outline a new shape, or one pin to maintain the current shape. If multiple layers, just mark outer layer.',
    type: 'dressLength',
  },
  {
    id: 215,
    item_type_id: 10,
    title: 'Shorten Dress Sleeves',
    price: 15.0,
    how_to_pin_image:
      'https://s3.us-east-2.amazonaws.com/airtailor-images/new_how_to_pin/22.jpg',
    instructions:
      'Place safety pin at desired sleeve length. Pin both sleeves for different lengths.',
    type: 'dressShoulders',
  },
  {
    id: 216,
    item_type_id: 10,
    title: 'Shorten Dress Straps',
    price: 15.0,
    how_to_pin_image:
      'https://s3.us-east-2.amazonaws.com/airtailor-images/new_how_to_pin/21.jpg',
    instructions:
      'Pinch excess fabric on shoulder straps to desired fit and safety pin the fabric together. Make sure bust and waist still fit comfortably.',
    type: 'dressShoulders',
  },
]
```
